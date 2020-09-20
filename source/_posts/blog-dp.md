---
title: Build Your Blog Site with Hexo and Heroku
date: 2020-02-23 21:15:21
tags: hexo, heroku, deployment
---
Having a personal blog site is sweet, I mean a good blog site. But sometimes building up your site from flat ground could be a headache, and that makes Hexo + Heroku one of the best solutions for creating and hosting a static website.

Hexo is a blog framework powered by NodeJS, which provides built-in command line tool that allows developers to create, update, and deploy the blog site within a simple interface. Of course, having a compelling UI is also an important aspect for creating a personal blog site. With the help of hundreds of community contributors, developers can customize their blog site with wide choices of open source hexo-compatible themes.

[Link to Hexo official documentation](https://hexo.io/docs/)

Heroku has been shined as a static content hosting platform for quite a long time. And among other hosting services such as Github Pages, Heroku is one of the top choices for developers to host static web applications. Moreover, Heroku automates the bundling process to your code repository during the project build and deployment process. With this powerful add-on, the developers are able to clone and work on well-templated codebase without configuring webpack files themselves.

[Heroku dev documentation](https://devcenter.heroku.com/categories/reference)

This article will walk you through the steps from setting up your Hexo blog site from scratch, to deploying the site on Heroku.

## Hexo
> The installation of NPM is required before proceeding the following steps

### Initialize your project folder with `hexo-cli`
Create a new project folder `my-blog`, cd to the folder, and install hexo built-in command line tool globally
```bash
$ cd my-blog
$ npm install hexo-cli -g
```

Set up the blog
```bash
$ hexo init <blog_folder_name>
$ cd <blog_folder_name>
```

### Run Hexo server
Run Hexo server to see your blog site at `localhost`
```bash
$ hexo server
```
The blog site will run at `localhost:4000`. You should be able to see the Hexo default theme with a big "Hexo" at the center of the landing picture.

And that's it. That's all it takes to set up your personal Hexo blog site. Now we are going to steer away from Hexo a bit and configure your Heroku account.

## Heroku

### Create a new app
In Heroku an `app` refers to a repository where the deployed codebase will be saved on the master branch. The very first step will be creating a new app by providing a unique app name.

![Site Image](/images/heroku_01.png)

### Download `heroku-cli`
Now we want to install Heroku command line tool so we can link our local branch to the remote branch on Heroku using built-in `heroku git` command line, after `git` is initialized within out project folder.

A handy way to download and install `heroku-cli` is to use `Homebrew` through terminal:
```bash
$ brew tap heroku/brew && brew install heroku
```

### Link local branch to remote repository on Heroku
We want to set our app, that is, the remote repository on Heroku as the destination for the deployment. Thus the local project repository needs to associate with the remote repository. But before that, there is one more step for Heroku user authentication.
```bash
$ heroku login
```
A new page will pop out in browser, prompting you to log in.

Now it's time to set up git. cd to the base folder of your blog project and do the following:
```bash
$ git init
$ heroku git:remote -a <your_app_name>
```
Up to this point, all the configurations on Hexo and Heroku are done!

## Deployment
To deploy the project to Heroku, we will need some help of `hexo-deployer-heroku` which is a npm package that takes the deploy parameters from `_config.yml`, and does the job of `git add`, `git commit -m`, and `git push` with one single command. We will talk it through in detail later.

First we need to install `hexo-deployer-heroku` using npm:
```bash
$ npm install hexo-deployer-heroku
```
### Configure the plugin in `_config.yml`
Under the root path of your blog folder, there is a file called `_config.yml`. This file basically contains the meta data of your blog site including the theme, language, and settings for writing blog posts.

In the end of file, there should be a field called `deploy`. That's where you define the parameters for your deployment process. Configure the fields in such way:
```text
deploy:
  type: heroku
  repo: <heroku_repository_git_url>
  message: [message]
```
You can find your Heorku git url under `Settings` in your app dashboard.

### Commit and deploy
Whenever you make changes to the blog site, make sure run `hexo generate` beforehand.
```bash
$ hexo generate
```
What this command does is to generate all untracked files, that is, the changes you have made to the repository. Now we need to add and commit these untracked files. Instead, we use only one-line command provided by `hexo-deployer-heroku`:
```bash
$ hexo deploy
```
This command bundles `git add`, `git commit -m`, and `git push` into one single command. Then you should be able see the project building progress through terminal, and that's when your project files gets re-built with Heroku Buildpacks and bundled into a more compact structure.

If the project building process is successful Heorku will start deploying your blog site. In the end, you will see something like this:

![Site image](/images/heroku_02.png)

Now comes the most exciting part. Go to `<your_app_name>.herokuapp.com` and you will see your personal blog site now is online!

You can also replace the heroku domain name with your customized one to make your site even more personalized. Furthermore, feel free to select and download your favorite hexo theme in [here](https://hexo.io/themes/), and embed into your site.
