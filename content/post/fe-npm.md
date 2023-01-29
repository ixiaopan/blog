---
title: "Package Manager"
date: "2022-01-07"
description: ""
# tags: []
categories: [
  "Frontend",
]
series: ["Frontend"]
katex: true
---



Learn how to use the modern package managers.



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

By default, `npm publish` will tag your package with the `latest` tag. 

```bash
# add tags to the existed pkg
npm <yourtag> add <pkg>@<version>

npm publish --tag beta

npm install somepkg@beta
```



### link

`npm link` can be used to debug your local package that is still developing. Suppose your project relies on the developing package `pkgA`  as shown below.

```bash
- project
  - node_modules
  	- pkgA
```

To debug pkgA, we follow th following steps

```bash
~ cd pkgA
~ npm link

~ cd project
~ npm link pkgA
```

When you are done, you can remove local dependency using `npm unlink`

```bash
cd project
npm unlink pkgA
```

You can also opt to remove the global link

```bash
cd pkgA
npm uninstall pkgA -g
```


### npmrc
This is the npm config file, which can be used to update the global or user-level config files. There are four relevant files

- project-related file (`path-to-project/.npmrc`)

- user-level config file (`~/.npmrc`)

- the global config file (`$PREFIX/etc/.npmrc`)

- the built-in config file


To access the the user-level config file, you can run 

```bash
npm config get userconfig
```

Likewise, to access the the global config file, run this command,

```bash
npm config get prefix
```


![](/blog/post/images/npmrc.png)



How to use it? One of the useful settings is `registry`. For example, we can set the default registry for our project to speed up the download progress.

```bash
cd project
touch .npmrc
```

Then specify your registry in `.npmrc`

```bash
registry=https://registry.npm.taobao.org/
```

If you have multiple project, you can also set the user-level or global config file


```bash
npm config set registry https://registry.npm.taobao.org/ [-g]
```

To remove the configuration, you just need to specify the key and delete it

```bash
npm config delete <key>
```


## yarn

### lock

yarn is another widely used package manager. When running `yarn install`, the file `yarn.lock` will be created. 

So, what's the use of this file? As its name suggests, `yarn.lock` will lock down the versions of `dependencies` specified in the `package.json`, which ensures that everyone in a team has the same dependencies.


### upgrade

Wait, how to upgrade versions of dependencies? Simply run `yarn upgrade`.

But this command can only upgrade versions between specified version range. If you need the latest version ignoring the specified version range, run `yarn upgrade --latest`.


## pnpm



## npx




## nrm

install 

```bash
npm install nrm -g
nrm ls
```



add your own registry

```bash
nrm add localnpm http://localhost:4873/
nrm use localnpm
```



![](/blog/post/images/nrm-ls.png)



remove 

```bash
nrm del localnpm
```


## References

- [publish-pkgs-docs](http://npm.github.io/publishing-pkgs-docs/publishing/index.html)
- [package.json](https://docs.npmjs.com/cli/v6/configuring-npm/package-json)
