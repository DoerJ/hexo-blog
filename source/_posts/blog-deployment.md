---
title: Build A Blog Site with Hexo and Heroku
date: 2020-02-23 21:15:21
tags: hexo, heroku, deployment
index_img: /images/thumbnail/heroku.jpeg
---
Having a personal blog site is sweet, but sometimes building up your site from scratch could be a headache. That makes hexo and heroku one of the best solutions for building and hosting a blogging site.

Hexo is a blogging framework powered by NodeJS, which provides built-in command line tool that allows developers to create, update, and deploy the blog site within just a few simple steps. Having a compelling UI can be a strong backup for the presentation of your works, and give viewers a deeper impression. With the help from hundreds of open-source community contributors, developers are able to customize their blog sites within a wide variety of choices of hexo-compatible themes.

[Link to Hexo official documentation](https://hexo.io/docs/)

Heroku has been shined as a static content hosting platform for quite a long time. Among other hosting services such as Github Pages, heroku is one of the top choices for developers to host static web applications. Furthermore, heroku automates the processes for building and bundling your codebase, and deploying the app on the remote server. 

[Heroku dev documentation](https://devcenter.heroku.com/categories/reference)

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

And that's it. That's all it takes to set up your personal Hexo blog site. Now we are going to steer away from Hexo a bit and configure your heroku account.

## Heroku

### Create a new app
In heroku, an `app` refers to a repository where the deployed codebase will be saved on the master branch. The very first step will be creating a new app by providing a unique app name.

![Site Image](/images/heroku/heroku_01.png)

### Download `heroku-cli`
Now we want to install heroku command-line tool so we can link our local branch to the remote branch on heroku using built-in `heroku git` command line, after `git` is initialized within out project folder.

A handy way to download and install `heroku-cli` is to use `Homebrew` through terminal:
```bash
$ brew tap heroku/brew && brew install heroku
```

### Link local branch to remote repository on Heroku
We want to set our app, that is, the remote repository on heroku as the destination for the deployment. Thus the local project repository needs to associate with the remote repository. But before that, there is one more step for heroku user authentication.
```bash
$ heroku login
```
A new page will pop out in browser, prompting you to log in.

Now it's time to set up git. cd to the base folder of your blog project and do the following:
```bash
$ git init
$ heroku git:remote -a <your_app_name>
```
Up to this point, all the configurations on hexo and heroku are done!

## Deployment
To deploy the project to heroku, we will need some help of `hexo-deployer-heroku` which is a npm package that takes the deploy parameters from `_config.yml`, and does the job of `git add`, `git commit -m`, and `git push` with one single command. We will talk it through in detail later.

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
You can find your heroku git-url under `Settings` in your heroku user dashboard.

### Commit and deploy
After make changes to the code repository, make sure that you run `hexo generate`.
```bash
$ hexo generate
```
What this command does is to generate all untracked files which are the changes you have made to the repository. Now we need to add and commit these untracked files. To do so, we use only one-line command provided by `hexo-deployer-heroku`:
```bash
$ hexo deploy
```
This command squashes `git add`, `git commit -m`, and `git push` into one single command. Then you should be able see the project building progress through terminal, and that's when your project files gets re-built with Heroku Buildpacks and bundled into a more compact structure.

If the deployment process is successful, you should be able to see something similar to below:

![Site image](/images/heroku/heroku_02.png)

Now comes to the exciting part. Go to `<your_app_name>.herokuapp.com` and you should be able to see your personal blog site online! You can also replace this default domain name with your customized domain to make your blog site nice-branded and even more personalized. Furthermore, feel free to download your favorite hexo theme in [here](https://hexo.io/themes/), and coorporate it with your site.

Hope this article can give you a start on creating your very own blog site. Stay tuned for more of interesting topics on modern web development!
