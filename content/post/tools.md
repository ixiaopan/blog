---
title: "Tools for daily work"
date: "2022-01-03"
description: ""
# tags: []
categories: [
    "frontend",
]
series: ["frontend"]
katex: true
---



Sharp tools make good work.



<!--more-->



## Daily Tools



### Terminal

- [Alfred](https://www.alfredapp.com/)

- [iTerm2 Download](https://iterm2.com/index.html)

- [oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh)

- [z - jump around](https://github.com/rupa/z)

```Bash
## download z.sh, and then put it at your favorite place
wget https://raw.githubusercontent.com/rupa/z/master/z.sh

## add z.sh to the .zashrc
# method 1
# vi .zashc
# method 2
echo 'path/to/z.sh' >> ~/.zshrc

## reload 
source ~/.zshrc
```



### Coding

- [Xcode - App Store](https://apps.apple.com/us/app/xcode/id497799835)

- [VSCode Download](https://code.visualstudio.com/download)
- VSCode Plugin List
  - Visual Studio IntelliCode
  - (ES6) code snippets
  - ES7 React/Redux/GraphQL/React-Native snippets
  - Vetur
  - Volar
  - 
  - Prettier — Code formatter
  - ESLint
  - EditorConfig for VS Code
  - dotenv
  - 
  - Path Intellisense
  - Auto Rename Tag
  - Code Runner
  - filesize
  - GitLens
  - Better Comments
  - Code Spell Checker



### Package Manager



- [Homebrew](https://brew.sh/)



```Bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```



You might encounter the error message `brew not found` when you type `brew` on your terminal due to the wrong path. First check the place where your brew is installed,



```
~ which brew
// /opt/homebrew/bin/brew
```



then add it to the global `PATH` env as follows.



```JavaScript
export PATH=$HOME/bin:/usr/local/bin:/opt/homebrew/bin:$PATH
```



- node



We use `nvm` to manage different versions of node.

```Nginx
brew install nvm

nvm install v16.13.1

nvm ls
nvm ls-remote
```



### Git

```bash
brew install git

// identity
git config --global user.name 'yourname'
git config --global user.email 'xx@mogic.ai'


// view your config
vi ~/.gitconfig
// or
git config --list


git config --global init.defaultBranch main
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.glog  'log --all --decorate --oneline --graph'

// remove
git config --global --unset alias.co
```





## FrontEnd Basis

### Fundamentals

- [CSS](https://developer.mozilla.org/en-US/docs/Web/CSS)
- [CSS Secrets](https://books.google.com.hk/books?id=nokNCgAAQBAJ&printsec=frontcover#v=onepage&q&f=false)
- [JavaScript.info](https://javascript.info/)
- [ES6 - You Don't Know JS ](https://github.com/getify/You-Dont-Know-JS)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)



### Framework

- React
- Vue



### Advanced

- [Webgl 2](https://webgl2fundamentals.org/webgl/lessons/)



### Building Tools

- Webpack
- Vite
- rollup
- gulp



### CI/CD

- [Pro Git - Book](https://git-scm.com/book/en/v2)
- [Semantic Version](https://semver.org/)



## Reference

- [A Guide to the 20 Best VSCode Extensions for Frontend Developers](https://javascript.plainenglish.io/a-guide-to-the-20-best-vscode-extensions-for-frontend-developers-f75a5d716091)
- [Pro Git]([Git - Book](https://git-scm.com/book/en/v2))