<div align="center">
  <br>
  <a href="https://github.com/jeffreytse/jekyll-deploy-action">
    <img alt="jekyll-theme-yat →~ jekyll" src="https://user-images.githubusercontent.com/9413601/107134556-211ea280-692e-11eb-9d13-afb253db5c67.png" width="600">
  </a>

  <p>🪂 A GitHub Action to deploy the Jekyll site conveniently for GitHub Pages.</p>
  <br>

  <h1> JEKYLL DEPLOY ACTION </h1>
</div>

<h4 align="center">
  <a href="https://jekyllrb.com/" target="_blank"><code>Jekyll</code></a> action for deployment.
</h4>

<p align="center">
  <a href="https://jeffreytse.github.io/jekyll-deploy-action">
    <img src="https://github.com/jeffreytse/jekyll-deploy-action/workflows/Tests/badge.svg" alt="Tests" />
  </a>
  <a href="https://github.com/jeffreytse/jekyll-deploy-action/releases">
    <img src="https://img.shields.io/github/v/release/jeffreytse/jekyll-deploy-action?color=brightgreen" alt="Release Version" />
  </a>
  <a href="https://opensource.org/licenses/MIT">
    <img src="https://img.shields.io/badge/License-MIT-brightgreen.svg" alt="License: MIT" /></a>
  <a href="https://liberapay.com/jeffreytse">
    <img src="http://img.shields.io/liberapay/goal/jeffreytse.svg?logo=liberapay" alt="Donate (Liberapay)" />
  </a>
  <a href="https://patreon.com/jeffreytse">
    <img src="https://img.shields.io/badge/support-patreon-F96854.svg?style=flat-square" alt="Donate (Patreon)" />
  </a>
  <a href="https://ko-fi.com/jeffreytse">
    <img height="20" src="https://www.ko-fi.com/img/githubbutton_sm.svg" alt="Donate (Ko-fi)" />
  </a>
</p>

<div align="center">
  <sub>Built with ❤︎ by
  <a href="https://jeffreytse.net">jeffreytse</a> and
  <a href="https://github.com/jeffreytse/jekyll-deploy-action/graphs/contributors">contributors </a>
</div>

## ✨ Story

GitHub Pages runs in `safe` mode and only allows a whitelisted [set of plugins](https://pages.github.com/versions/). 
Sometimes we might want to use other plugins, like those listed [here](https://github.com/planetjekyll/awesome-jekyll-plugins).  E.g.
  
- GitHub-Pages Unscramble
- Jekyll Spaceship

To use these additional gem plugins in GitHub Pages, we need to build our Jekyll site locally or use CI (e.g. [travis](https://travis-ci.org/), [github workflow](https://help.github.com/en/actions/configuring-and-managing-workflows/configuring-a-workflow)) and then deploy to our `gh-pages` branch.

TL;DR:
  
**If you want to make a Jekyll site run as if it were local, such as let custom plugins work properly, then this repo provides a GitHub Action that automates the Jekyll build or you.  It builds the site and deploys to GitHub Pages.**

## 📚 Usage

### Create the GitHub Action

Add a github workflow file - e.g. `.github/workflows/build-jekyll.yml` - in your repository's `master` branch. You can create it manually, 
or Actions --> New workflow --> set up a workflow yourself --> `build-jekyll.yml`.  
  
The file contents needs to look like this:

```yml
name: Build and Deploy to Github Pages

on:
  push:
    branches:
      - master  # Here source code branch is `master`, it could be other branch

jobs:
  build_and_deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      # Use GitHub Actions' cache to cache dependencies on servers
      - uses: actions/cache@v3
        with:
          path: vendor/bundle
          key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile.lock') }}
          restore-keys: |
            ${{ runner.os }}-gems-

      # Use GitHub Deploy Action to build and deploy to Github
      - uses: jeffreytse/jekyll-deploy-action@v0.4.0
        with:
          provider: 'github'
          token: ${{ secrets.GITHUB_TOKEN }} # It's your Personal Access Token(PAT)
          repository: ''             # Default is current repository
          branch: 'gh-pages'         # Default is gh-pages for github provider
          jekyll_src: './'           # This is the root of your Jekyll site, relative to your repo. E.g. './docs/'. Default is root directory
          jekyll_cfg: '_config.yml'  # Default is _config.yml
          jekyll_baseurl: ''         # Default is according to _config.yml
          bundler_ver: '>=0'         # Default is latest bundler version
          cname: ''                  # Default is to not use a cname
          actor: ''                  # Default is the GITHUB_ACTOR
          pre_build_commands: ''     # Installing additional dependencies (Arch Linux)
```

**💡 Don't forget to update your `jekyll_src` location to point to your docs folder, if it's not in the repo root.**

Now commit your action.

### Ensure You Have a `gh-pages` Branch
  
If you don't have the `gh-pages` branch, you should create one.  This code creates an empty `gh-pages` branch with no source, as we require:

```bash
git checkout --orphan gh-pages
git rm -rf .
git commit --allow-empty -m "initial commit"
git push origin gh-pages
```

Back on your client, you'll probably now want to switch back to your master/main branch, e.g.

```bash
git checkout master
```

### Configure Your GitHub Pages Settings
  
Finally:
  
Go to your repository’s Settings --> Pages, and set the `Source` to the `gh-pages` branch. Leave the folder as `/(root)`.

**💡 Note:** 
- The `gh-pages` branch is only for the built site static files.  The built content will be created in the root.
  Consequently, when configuring your Repo --> Settings --> Pages, set the folder to `/(root)`.
  (Even if the source for your site lives in a different folder in the master branch.)
- The source of your site (i.e. the pre-rendered site) should still live in the `master` branch.

### Configuring Additional Permissions
  
At the start of each workflow run, GitHub automatically creates a unique `GITHUB_TOKEN` secret to use in your workflow. 
You can use the `GITHUB_TOKEN` to authenticate in a workflow run. 
You can use the `GITHUB_TOKEN` by using the standard syntax for referencing secrets: `${{ secrets.GITHUB_TOKEN }}`. 
For more information, see [here](https://docs.github.com/en/actions/security-guides/automatic-token-authentication).

If you need a token that requires permissions that aren't available in the `GITHUB_TOKEN`, 
you can create a Personal Access Token (PAT), and set it as a secret in your repository for this action to push to the `gh-pages` branch:

- Create a [Personal Access Token](https://github.com/settings/tokens) with custom permissions and copy the value.
- Go to your repository’s Settings and then switch to the Secrets tab.
- Create a token named `GH_TOKEN` (important) using the value copied.

## Running and Scheduling
  
This workflow will be triggered whenever you commit to your master branch.

You can also schedule the workflow to run routinely, but there may not be any need to do this.
To schedule a workflow, you can use the POSIX cron syntax in your workflow file.
The shortest interval you can run scheduled workflows is once every 5 minutes. For example, this workflow is triggered every hour.

```yml
on:
  schedule:
    - cron:  '0 * * * *'
```

## 🌱 Credits

- [Jekyll](https://github.com/jekyll/jekyll) - A blog-aware static site generator in Ruby.
- [actions/checkout](https://github.com/actions/checkout) - Action for checking out a repo.
- [actions/cache](https://github.com/actions/cache) - Cache dependencies and build outputs in GitHub Actions.

## ✍️  Contributing

Issues and Pull Requests are greatly appreciated. If you've never contributed to an open source project before I'm more than happy to walk you through how to create a pull request.

You can start by [opening an issue](https://github.com/jeffreytse/jekyll-deploy-action/issues/new) describing the problem that you're looking to resolve and we'll go from there.

## 🌈 License

This software is licensed under the [MIT license](https://opensource.org/licenses/mit-license.php) © JeffreyTse.
