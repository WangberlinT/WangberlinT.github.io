---
title: Deploy Github Page with Github Action
date: 2023-07-16 20:33:56
tags: CI/CD
---

I’m using [Hexo](https://hexo.io/) + [Github action](https://github.com/features/actions) + Github page to create and deploy this blog.

It’s a very handy tool to do automation.

## What is Github action?

> GitHub Actions is a continuous integration and continuous delivery (CI/CD) platform that allows you to automate your build, test, and deployment pipeline. You can create workflows that build and test every pull request to your repository, or deploy merged pull requests to production.
> 

In short, Github action allows you to do some automation base on the events.

## **How it works?**

![Untitled](Deploy%20Github%20Page%20with%20Github%20Action/github_action.png)

Github action contains the following parts:

1. Event: a group of github / git events
2. Workflow: A workflow observes events, check condition, and run jobs parallel or sequentially.
3. Job: A job will execute several steps sequencially in an environment.
4. Step: A step is a group of sequencial actions and scripts. We can find a lot of handy action in [awesome-actions](https://github.com/sdras/awesome-actions#github-pages)

This is the example of my blog’s Github action configuration.

```java
# ./github/workflows/pages.yaml
name: Pages # the name of the workflow.

on: # events and conditions
  push: # only push event on main branch will trigger it
    branches:
      - main  # default branch

jobs: # jobs it runs
  pages: # a job called pages
    runs-on: ubuntu-latest # run on ubunto-latest enviorment
    permissions:
      contents: write
    steps: # steps executed in the job
      - uses: actions/checkout@v3 # step1 checkout branch https://github.com/actions/checkout
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Use Node.js 18.x # step2 setup nodejs
        uses: actions/setup-node@v2
        with:
          node-version: '18'
      - name: Cache NPM dependencies # step3 setup dependencies cache
        uses: actions/cache@v2 # https://github.com/actions/cache
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies # step4 install dependencies
        run: npm install # script
      - name: Build # step 5 build blog pages.
        run: npm run build
      - name: Deploy # step 6 deploy page to github page
        uses: peaceiris/actions-gh-pages@v3 # https://github.com/peaceiris/actions-gh-pages
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

The workflow is configurated in [YAML](https://en.wikipedia.org/wiki/YAML) file. Github action will check and run workflow YAML files under `./github/workflows/` accroding to the given event.

So in this `pages` workflow, everytimes I push something to `main` branch, it will execute the following 6 steps:

1. checkout and update current branch
2. setup [Node.js](https://nodejs.org/en)
3. cache npm dependencies (next time `npm install` will skip all cached dependencies)
4. install all dependencies.
5. build hexo blog
6. deploy the web page output to github page.

## Read more

[AWS: What is Continuous Integration?](https://aws.amazon.com/devops/continuous-integration/#:~:text=Continuous%20integration%20refers%20to%20the,for%20a%20release%20to%20production).

[Hexo: deploy to github page](https://hexo.io/docs/github-pages)