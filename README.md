# Deploy a .NET Blazor WebAssembly to GitHub Pages

> While getting this up and running, I relied heavily on the instructions laid out in [this excellent post](https://swimburger.net/blog/dotnet/how-to-deploy-aspnet-blazor-webassembly-to-github-pages) by [Niels Swimberghe](https://github.com/Swimburger)

## Introduction

Blazor WebAssembly is a UI technology from Microsoft that allows developers to create single-page applications using C# and .NET. It utilizes a component-based architecture (think React.js) and is a client-side, in-browser implementation of Blazor. Because WebAssembly is client-side rather than server-side, this means that (with some tweaks) you can host a Blazor WebAssembly in GitHub Pages!

## Technologies

This project is built with Microsoft's Blazor, using C# and .NET SDK 5.0.x. It also relies heavily on GitHub Actions for implementation, and GitHub Pages for hosting. Because of this, you will need a GitHub account to reproduce this project.

## Example

A working example of a Blazor WebAssembly hosted on GitHub Pages may be found at [https://tenaciousdev.github.io/BlazorGitHubPagesDemo/](https://tenaciousdev.github.io/BlazorGitHubPagesDemo/)

## Setup

To install this project locally, clone the repo located at:

```
https://github.com/TenaciousDev/BlazorGitHubPagesDemo.git
```

The GitHub actions necessary to deploy this project to GitHub Pages are written into a YAML file at `.github/workflows/main.yml`

## Steps To Reproduce (Walkthrough)

As noted above, when I was developing this implementation, I relied heavily on [a blog post by Niels Swimberghe](https://swimburger.net/blog/dotnet/how-to-deploy-aspnet-blazor-webassembly-to-github-pages). Because of some differences in .NET versions, and updates on the part of GitHub since the time of Mr. Swimberghe's writing, I've decided to compose a step-by-step walkthrough of my own implementation. I highly recommend reading Mr. Swimberghe's blog post anyway, it was an invaluable resource while figuring out how to get this up and running.

Now, let's explore the process.

---

### Create the Blazor WebAssembly project

Open your preferred CLI and navigate to whichever directory you would like to make the parent directory of your project. Run the following commands:

```cmd
dotnet new blazorwasm -o BlazorGitHubPagesDemo
```

Once the application has finished scaffolding out, navigate into the output folder you created with the `-o` flag:

```cmd
cd BlazorGitHubPagesDemo
```

Go ahead and give it a test run. The following command will build and run the project using kestrel server:

```cmd
dotnet run
```

Or, if you'd like to leave the app running while you make changes, and then have the app automatically rebuild and run when you save those changes, you can add the `watch` command:

```cmd
dotnet watch run
```

Running either of these should cause your default browser to open in either a new window or a new tab. The default view of the Blazor App should look like this:

![Blazor App Default View](wwwroot/assets/BlazorDefaultImage.png)

Once you've verified that your Blazor WebAssembly is configured correctly, you can use `Ctrl + C` in your CLI to stop the application from running and proceed to the next step.

> Note: In the event that you aren't able to use `Ctrl + C` to stop the run - if, for example, you've closed the CLI instance that initiated the run, and the run did not self-terminate - there are two other ways to kill the kestrel server instance.
>
> 1. If you know the process Id of the run, you can open a Powershell instance and run the command `kill <id>`, replacing `<id>` with the process Id.
>
> 2. If you do not know the process Id, you can open a Powershell instance and run the command pipeline `Get-Process -Name *dotnet* | Stop-Process` to kill all active instances of kestrel server.

---

### Commit the project using Git, and push to a GitHub repository

Now that we've scaffolded a Blazor WebAssembly, it's time to initialize a git repository. We'll make sure we have an appropriate `.gitignore` for our project, and we'll commit our newly scaffolded project. After the commit, we'll ensure our default branch is named `main` in accordance with [GitHub's guidelines](https://github.com/github/renaming). Run the following commands in your CLI:

```git
dotnet new gitignore
git init
git add .
git commit -m "Initial commit"
git branch -M main
```

Create a new GitHub repository, set its address as the remote origin of your local repository, and configure upstream tracking:

```git
git remote add origin https://github.com/<GH_USERNAME>/<GH_REPO_NAME>.git
git push -u origin main
```

Confirm your local repository has been pushed to the remote GitHub repository, then move on to the next step.

---

### Create and run a GitHub Actions Workflow

Now that all source code has been pushed to GitHub, it's time to dig in to the bulk of what makes this work: GitHub Actions! From the [documentation](https://docs.github.com/en/actions):

> GitHub Actions help you automate tasks within your software development life cycle. GitHub Actions are event-driven, meaning that you can run a series of commands after a specified event has occurred. For example, every time someone creates a pull request for a repository, you can automatically run a command that executes a software testing script.

A GitHub Actions Workflow can be run on a GitHub-hosted virtual machine called a [runner](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners). These runners support various types of software, depending on the OS of the runner VM. We'll use a VM using the latest version of [Ubuntu](https://github.com/actions/virtual-environments/blob/main/images/linux/Ubuntu2004-README.md) in order to access the other pieces of software we'll need for our Workflow.

Now, let's think about what our Workflow will need to look like.

For our project, we'll need to create a Workflow that builds the project each time we push to our `main` branch, and then commits and pushes the output of that build to a separate GitHub branch. This branch, which we'll call `gh-pages`, will be used by GitHub Pages to host our static site files.

In your browser, navigae to the **Actions** tab in your GitHub repository. Once there, click the link that says "set up a workflow yourself". This will direct you to the path `BlazorgitHubPagesDemo / .github / workflows /`, and will create a new [YAML](https://yaml.org/) file named `main.yml`.

You should see the contents of this new `main.yml` file in a GitHub text editor. Delete _all the contents_ of this file. We're going to rebuild it from scratch!

The actions we'll write into this file will be contained in a Workflow, and a Workflow is comprised of one or more jobs. The jobs are then _further_ broken down into steps! For our purposes here, we'll run a single job with multiple steps. These steps will be, in order:

1. use GitHub's checkout action to checkout code from the main branch

2. set up our .NET SDK

3. publish our Blazor projects to an output folder called `release`

4. change an HTML element - the `<base />` tag - in our static `index.html` file so that it matches the GitHub Pages subdirectory structure

   > Note: we'll make this change **_only in our remote `gh_pages` branch_**

5. redirect away from GitHub's built-in **404** page in the event that a resource isn't found (we'll direct back to `index.html` for now)

6. add a special file called `.nojekyll` to allow GitHub pages to read files and folders that start with an underscore

7. finally, after all the other steps have been completed, publish our `wwwroot` directory to GitHub Pages

We'll do all this inside our `main.yml` file in the steps that follow. But first, we need to give our workflow a descriptive name, and give it a trigger. Write the following code into your `main.yml` file:

```yml
name: Deploy to GitHub Pages

# Run workflow on every push to the main branch
on:
  push:
    branches: [main]
```

Now we'll configure our jobs list, and add a job to it. This job will specify that it needs to run on the latest version of Ubuntu, and we'll also get ready to configure our list of steps. Add this new section of code to `main.yml`, underneath the code we added previously:

```yml
jobs:
  deploy-to-github-pages:
    # use ubuntu-latest image to run steps on
    runs-on: ubuntu-latest
    steps:
```

Now it's time to configure our steps!

Let's go ahead and add some comments to block off space for our steps, and make sure we know what the structure of `main.yml` will look like when we're done. These comments will also act as explanation for what each part of our YAML code does:

```yml
name: Deploy to GitHub Pages

# Run workflow on every push to the main branch
on:
  push:
    branches: [main]

jobs:
  deploy-to-github-pages:
    # use ubuntu-latest image to run steps on
    runs-on: ubuntu-latest
    steps:
      # uses GitHub's checkout action to checkout code from the main branch

      # sets up .NET SDK 5.0.x

      # publishes Blazor project to the release folder

      # changes the base-tag in index.html from '/' to 'BlazorGitHubPagesDemo' to match GitHub Pages repository subdirectory

      # copy index.html to 404.html to serve the same file when a file is not found

      # add .nojekyll file to tell GitHub pages to not treat this as a Jekyll project. (Allow files and folders starting with an underscore)

      # publishes wwwroot directory to GitHub Pages
```

Okay, now that we have the comments structured out, let's start at the top and work our way down the steps list. First, we need to checkout code from `main`. We'll do this by utlizing GitHub's [built-in checkout action](https://github.com/actions/checkout)! Add this code:

```yml
# uses GitHub's checkout action to checkout code from the main branch
- uses: actions/checkout@v2
```

Next, we need to set up our .NET SDK. We'll use another predefined action for this, and we'll specify that we want to use the most recent compatible version of .NET 5.0 with a `dotnet-version` keyword. Add this code:

```yml
# sets up .NET SDK 5.0.x
- name: Setup .NET SDK 5.0.x
  uses: actions/setup-dotnet@v1
  with:
    dotnet-version: "5.0.x"
```

We want to publish our Blazor project to the `release` output folder using the `Release` configuration. We'll use an `-o` flag to specify an output folder during publication, and a `-c` flag to specify the `Release` configuration:

```yml
# publishes Blazor project to the release folder
- name: Publish .NET Project
  run: dotnet publish BlazorGitHubPagesDemo.csproj -c Release -o release --nologo
```

> Note: the `--no-logo` flag may not be absolutely necessary here, but it prevents some unneccesary lines from being output to the console

Now, we need to prevent a major issue! Because of the way GitHub Pages structures it's subdirectories, if we leave our `wwwroot/index.html` file as it is right now, the application will try to grab its resources from `<username>.github.io`, because that's the root of the website. But unless we're deploying this application to that address - which we're not, or at least we shouldn't be - that isn't going to work.

We _could_ just go in to the `index.html` file and edit it directly, but if we do that we won't be able to run the application locally. There may be more than one solution to this issue, but for this walkthrough, we'll solve it by having our next step make a change to the `<base />` tag of `index.html`. Because this takes place as part of our GitHub actions _only when we push to `main`_, and because we'll be publishing our static files inside the `wwwroot` directory to a separate branch, this won't effect our local version of the file! Add the following code:

```yml
# changes the base-tag in index.html from '/' to 'BlazorGitHubPagesDemo' to match GitHub Pages repository subdirectory
- name: Change base-tag in index.html from / to BlazorGitHubPagesDemo
  run: sed -i 's/<base href="\/" \/>/<base href="\/BlazorGitHubPagesDemo\/" \/>/g' release/wwwroot/index.html
```

The next step is more of a housekeeping issue. GitHub automatically directs any request that returns a `404` response to the GitHub-specific 404 page. If one of our users requests a resource we don't have, this could yank them away from our application, which isn't the end of the world, but it's not a great user experience. For now, we'll run a command that will simply redirect any users that encounter a `404` rsponse, back to our main page. Add this code:

```yml
# copy index.html to 404.html to serve the same file when a file is not found
- name: copy index.html to 404.html
  run: cp release/wwwroot/index.html release/wwwroot/404.html
```

And another housekeeping step: we need to talk about underscores.

GitHub Pages uses [Jekyll](https://jekyllrb.com/) by default, and Jekyll is a wonderful tool for hosting static sites. The problem is that Jekyll ignores folders or files that start with an underscore. And when we publish our app with Blazor, the publisher creates a directory starting with, you guessed it, an underscore! This directory is `wwwroot/_framework`, and it's crucial to our application. We're gonna need a workaround.

Thankfully, GitHub solved this particular issue for us [back in 2009](https://github.blog/2009-12-29-bypassing-jekyll-on-github-pages/). All we need to do is add a `.nojekyll` file to our `wwwroot` directory before we publish it. Add this code:

```yml
# add .nojekyll file to tell GitHub pages to not treat this as a Jekyll project. (Allow files and folders starting with an underscore)
- name: Add .nojekyll file
  run: touch release/wwwroot/.nojekyll
```

We're almost there! Now all we need is to publish the `wwwroot` directory, which contains our static files, to GitHub Pages via our `gh-pages` branch. We'll use a handy predefined action as part of this, [courtesy of James Ives](https://github.com/JamesIves/github-pages-deploy-action).

We'll also use something called `GITHUB_TOKEN`, which is a secret variable automatically created at the beginning of a workflow run. used to authenticate the run. You can learn more about how this works in the [GitHub Docs](https://docs.github.com/en/actions/security-guides/automatic-token-authentication), but for now let's put this secret token to work! Add the following code:

```yml
# publishes wwwroot directory to GitHub Pages
- name: Commit wwwroot to GitHub Pages
  uses: JamesIves/github-pages-deploy-action@4.1.5
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    branch: gh-pages # The branch the action should deploy to.
    folder: release/wwwroot # The folder the action should deploy.
```

Whew! That was a lot, but you'll be happy to know we're just about done with our `main.yml` file, and after that it'll be time to deploy. First, let's read through our `main.yml` file one more time, just in case we have any typos hanging around. Here's what the complete file should look like:

```yml
name: Deploy to GitHub Pages

# Run workflow on every push to the main branch
on:
  push:
    branches: [main]

jobs:
  deploy-to-github-pages:
    # use ubuntu-latest image to run steps on
    runs-on: ubuntu-latest
    steps:
      # uses GitHub's checkout action to checkout code from the main branch
      - uses: actions/checkout@v2

      # sets up .NET SDK 5.0.x
      - name: Setup .NET SDK 5.0.x
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: "5.0.x"

      # publishes Blazor project to the release folder
      - name: Publish .NET Project
        run: dotnet publish BlazorGitHubPagesDemo.csproj -c Release -o release --nologo

      # changes the base-tag in index.html from '/' to 'BlazorGitHubPagesDemo' to match GitHub Pages repository subdirectory
      - name: Change base-tag in index.html from / to BlazorGitHubPagesDemo
        run: sed -i 's/<base href="\/" \/>/<base href="\/BlazorGitHubPagesDemo\/" \/>/g' release/wwwroot/index.html

      # copy index.html to 404.html to serve the same file when a file is not found
      - name: copy index.html to 404.html
        run: cp release/wwwroot/index.html release/wwwroot/404.html

      # add .nojekyll file to tell GitHub pages to not treat this as a Jekyll project. (Allow files and folders starting with an underscore)
      - name: Add .nojekyll file
        run: touch release/wwwroot/.nojekyll

      # publishes wwwroot directory to GitHub Pages
      - name: Commit wwwroot to GitHub Pages
        uses: JamesIves/github-pages-deploy-action@4.1.5
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          branch: gh-pages # The branch the action should deploy to.
          folder: release/wwwroot # The folder the action should deploy.
```

Once you're satisfied that everything is correct, it's time to commit our changes! If you've been writing this in the GitHub text editor, you can use the same interface to create a fresh commit. If you switched to a local IDE like VS Code, you can add your changes, make a new commit, and push to the remote repository.

After you've pushed your changes to GitHub, navigate back to the **Actions** tab. You should see your workflow running, or possibly already complete. Now all that's left is to configure GitHub Pages!

Navigte to **Settings > Pages**, and under _Source_ select the `gh-pages` branch, and click the _Save_ button. And boom! Your site will be published in a few moments!

That's it, folks!
