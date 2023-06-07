---
title: "Tools for daily work"
date: "2022-01-03"
description: ""
# tags: []
categories: [
   "Frontend",
]
series: ["Frontend"]
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



#### [Homebrew](https://brew.sh/)



```Bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```


You might encounter the error message `brew not found` when you type `brew` on your terminal due to the wrong path. First check the place where your brew is installed,



```bash
~ which brew
# /opt/homebrew/bin/brew
```



then add it to the global `PATH` env as follows.



```JavaScript
export PATH=$HOME/bin:/usr/local/bin:/opt/homebrew/bin:$PATH
```

#### pip

https://pip.pypa.io/en/stable/installation/


```bash
python -m ensurepip --upgrade

wget  https://bootstrap.pypa.io/get-pip.py

python get-pip.py
```


#### nvm



We use `nvm` to manage different versions of node.

```bash
brew install nvm

nvm install v16.13.1

nvm ls
nvm ls-remote
```



### Git

```bash
brew install git

# identity
git config --global user.name 'yourname'
git config --global user.email 'xx@163.com'


# view your config
vi ~/.gitconfig
# or
git config --list


git config --global init.defaultBranch main
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.glog  'log --all --decorate --oneline --graph'

# remove
git config --global --unset alias.co

# clean branches
## look up
git br | grep 'pattern'
## delete it
git br | grep 'pattern' | xargs git branch -D
```


### Download

- [Library Genesis](http://libgen.rs/)

- [jiumodiary](https://www.jiumodiary.com/)

- [gutenberg.org](https://www.gutenberg.org/)

- [allitbooks.net](https://allitbooks.net/) 


### UX
- [SimpleIcons](https://simpleicons.org/)

- [IconFinder](https://www.iconfinder.com/)

- [SVG Sprite Generator](https://svgsprit.es/)



## English

### Vocabulary

- [vocabulary.com](https://www.vocabulary.com/dictionary/)

- [Grammar Byte](https://chompchomp.com/terms.htm)


### Paraphrase

- [powerthesaurus](https://www.powerthesaurus.org/)

- [quillbot (Strongly Recommend)](https://quillbot.com/)

- [Ginger Grammar Checker](https://www.gingersoftware.com/grammarcheck#.XRHyeZMzbOQ)

- [lingohelpme](https://lingohelp.me/)

- [BibGuru](https://app.bibguru.com/p/ef52ca5d-cd2d-41a7-bd90-5f23b0f75651)


## FrontEnd

### Fundamentals

- [CSS - MDN](https://developer.mozilla.org/en-US/docs/Web/CSS)
- [CSS Secrets](https://books.google.com.hk/books?id=nokNCgAAQBAJ&printsec=frontcover#v=onepage&q&f=false)
- [JavaScript.info](https://javascript.info/)
- [ES6 - You Don't Know JS ](https://github.com/getify/You-Dont-Know-JS)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)
- [Functional-Light JS](https://github.com/getify/Functional-Light-JS)


### Advanced

- [WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly)
- [Webgl 2](https://webgl2fundamentals.org/webgl/lessons/)
- [Web Audio](https://webaudioapi.com/book/Web_Audio_API_Boris_Smus_html/toc.html)

### CI/CD

- [Pro Git - Book](https://git-scm.com/book/en/v2)
- [Semantic Version](https://semver.org/)


## Database

- [SQL Zoo](https://sqlzoo.net/wiki/SELECT_basics)
- [SQL Tutorial](https://www.sqltutorial.org/)
- [Build Your Own Database From Scratch](https://build-your-own.org/database/)

## Statistics

- [Math24](https://math24.net/probability-density-function.html)
- [XMathMain (Math Collection)](https://xaktly.com/XMathMain.html)
- [Introduction to Statistics](https://courses.lumenlearning.com/introstats1/)
- [Intro to Probability](https://dlsun.github.io/probability/)
- [Applied Multivariate Statistical Analysis](https://online.stat.psu.edu/stat505/book/)



## Machine Learning


- [from-python-to-numpy](https://www.labri.fr/perso/nrougier/from-python-to-numpy/)
- [shanelynn blog](https://www.shanelynn.ie/)
- [Neural Networks and Deep Learning - By Michael Nielsen / Dec 2019 ](http://neuralnetworksanddeeplearning.com/chap1.html)
- [Deep Learning Do it Yourself](https://dataflowr.github.io/website/)

## Visualisation

- [Data Visualization Book Reviews](https://www.visualcinnamon.com/resources/learning-data-visualization/books/)
- [NAPA Cards](https://napa-cards.net/)
- [datavizproject](https://datavizproject.com/)
- [data-to-viz](https://www.data-to-viz.com/)
- [datavizcatalogue](https://datavizcatalogue.com/)
- [makeovermonday](https://www.makeovermonday.co.uk/data/)

## Complexity

- [introduction-to-complexity](https://www.complexityexplorer.org/courses/97-introduction-to-complexity-2019/segments/8352?summary)
