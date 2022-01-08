---
title: "NPM"
date: "2022-01-07"
description: ""
# tags: []
categories: [
    "frontend",
]
series: ["frontend"]
katex: true

---



Learn how to use npm.



<!--more-->



## npm

### version

- `major`
- `minor`
- `path`
- `prerelease`
  - `alpha`: WIP, unstable
  - `beta`: features completed but with unknown bugs
  - `rc`: ready to release



Suppose the current version is `1.2.3`, there are many ways to update the version.

```bash
# 2.0.0
npm version major

# 1.3.0
npm version minor

# 1.2.4
npm version patch

npm version <version>

# 1.2.4-0
npm version prerelease -m "update %s" --no-git-tag-version


# if the current version is 1.2.3-alpha.0, then it becomes 1.2.3-alpha.1
npm version prerelease --preid=<prerelease-id>
```



### tag



#### add tags to the existed pkg

```
npm dist-tag add <pkg>@<version> [<tag>]
```



#### publish with tags

By default, `npm publish` will tag your package with the `latest` tag. 

```
npm publish --tag beta
```



#### Install pkg with tags

```
npm install somepkg@beta
```



### link



### npmrc



## npx





## nrm



### Usage

install 

```
npm install nrm -g
nrm ls
```



add your own registry

```
nrm add localnpm http://localhost:4873/
nrm use localnpm
```



![](/blog/post/images/nrm-ls.png)



remove 

```
nrm del localnpm
```



### Why



## Reference

- http://npm.github.io/publishing-pkgs-docs/publishing/index.html