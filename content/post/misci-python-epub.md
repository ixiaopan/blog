---
title: "html2epub"
date: "2022-02-19"
description: ""
categories: [
  "Misc",
]
series: ["Computer Science"]
katex: true
---


A simple book scraper written in Python, converting html to epub.


<!--more-->


## Install

```bash
pip install bs4 inquirer html2epub tqdm
```

## Usage

```bash
wget https://raw.githubusercontent.com/ixiaopan/DataScience/master/Utilities/Scraper/download.py

python download.py
```

After running the above commands, you need to select one source, as shown below.

![](/blog/post/images/html2epub.png)


Note that this script can only convert html to epub. In other words, only books in HTML format can be downloaded. 

### Books with chapter list

Books may or may not have a chapter list. For books with a chapter list, just type the URL. For example, the below one.

```
https://booksvooks.com/selfish-shallow-and-self-absorbed-sixteen-writers-on-the-decision-not-to-have-kids-pdf.html
```

### Books without chapter list

For books without a chapter list, you need to add the total page number to the page URL. For example, the book `this is going to hurt`  does not have a chapter list, so the url is

```
https://booksvooks.com/fullbook/this-is-going-to-hurt-pdf-adam-kay.html?page=32
```


The book will be downloaded in the folder `booksVooks/` or `jinjiang` based on your source.

```
- booksVooks
  - thisisgoingtohurtpdfadamkaypage32.epub
```
