title: Common Hexo Workflow
tags:
  - hexo
date: 2015-09-04 12:30:35
---


## Create draft

```sh
hexo new draft "your article name"
emacs "your article name"
```
After this, the draft file will be created under `source/_drafts` folder without `Date` tag added.


## Publish draft

```sh
hexo publish "your article name"
```
After this, the draft will will be move to `source/_posts` and the internal `Date` flag will be updated to the publish timestamp.

## Test your post
```sh
hexo generate
hexo server
```
Generate all the static blog file and start the local server.

You can simply do this via alias:
```sh
hexo g && hexo s
```

## Publish to github page
```sh
hexo generate
hexo deploy
```
Of course you need to config the git in site `_config.yml` file. And the hexo will publish the result to github page.

## Some memo
- Manually set article digest
  after this line in your markdown file will be hidden from the homepage.
  `<!--more-->`

