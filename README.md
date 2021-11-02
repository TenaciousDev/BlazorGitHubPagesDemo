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

```http
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
mkdir BlazorGitHubPagesDemo
cd BlazorGitHubPagesDemo
dotnet new blazorwasm
```

Once the application has finished scaffolding out, go ahead and give it a test run. The following command will build and run the project using kestrel server:

```cmd
dotnet run
```

Or, if you'd like to leave the app running while you make changes, and then have the app automatically rebuild and run when you save those changes, you can add the `watch` command:

```cmd
dotnet watch run
```

Running either of these should cause your default browser to open in either a new window or a new tab. The default view of the Blazor App should look like this:

![Blazor App Default View](wwwroot\assets\BlazorDefaultImage.png)

Once you've verified that your Blazor WebAssembly is configured correctly, you can use `Ctrl + C` in your CLI to stop the application from running and proceed to the next step.

> Note: In the event that you aren't able to use `Ctrl +C` to stop the run - if, for example, you've closed the CLI instance that initiated the run, and the run did not self-terminate - there are two other ways to kill the kestrel server instance.
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

Now that all source code has been pushed to GitHub, it's time to dig in to the bulk of what makes this work: GitHub Actions.

> Section to be continued...
