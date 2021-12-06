---
title: Using pandoc to build pdf with toc in github action
updated: 2021-12-06
---

# Using pandoc to build pdf with toc in github action

[toc]

## Overview

This doc is written based on below listed version, any updates please refer to offical doc.
pandoc 2.16.2 (2021-11-21).
tlmgr revision 60693 (2021-10-04).
fnm 1.28.1 installed with node v17.2.0.

sample project: https://github.com/zilliztech/docs.zilliz.com
sample docker: https://hub.docker.com/repository/docker/xiaohongczh/xelatex

## Prepare source content

Pandoc supports various format doc transformation. (see https://pandoc.org/index.html). It extract doc title as toc entry. Like #, ##, ###... in markdown or h1, h2, h3...in html. Before we generate pdf, we have to reformat doc with proper doc title so that the pdf toc will meet our requirement.
Or
Another way is to use latex syntax to build a toc template and import this template while pandoc building process. We would not discussion latex syntax here.

## Build docker

To run pandoc in github action, a docker image is required. We may need different packages to build pdf. Offical image may need extra installation. So we build a customized image on ourselves. 

### Start a base ubuntu image
Start a minized Ubuntu image:
```docker run -it -v ~/Documents/docker_vol_01:/docker_vol_01 ubuntu bash```

### Install pandoc
Download deb package from [github](https://github.com/jgm/pandoc/releases/)

### Install texlive
Since we need to use xelatex as pdf build engine (default pdf engine is pdflatex which does not support some unicode like Chinese character or emoji well), texlive is a must to be installed.

Here is [some ways](https://tug.org/texlive/acquire.html) of installing texlive

The [Mirroring/downloading the TeX Live repository](https://tug.org/texlive/acquire-mirror.html) is the most stable and the only way that I succeeded.

Chose proper [site](https://ctan.org/mirrors)
and
rsync -a --delete rsync://somectan/somepath/systems/texlive/tlnet/ /your/local/dir/
or 
wget --mirror --no-parent ftp://somectan/somepath/systems/texlive/tlnet/ /your/local/dir

*This step might take dozens of hours, so be prepared in advanced.*

### Remove useless code
1. Get into `/usr/local/texlive/2021`
2. Run `du -sh ./*` `doc` folder is around 4GB.
3. Remove `doc` folder

### Install nodejs (optional)
We need script runtime to excute jobs. We chose nodejs to run jobs and fnm to install node.

You can pick any language you prefered and install them on this image.

### Config env variables
Check .bashrc ro .bash_profile if PATH is proplerly configured.

```shell
# texlive
export PATH=/usr/local/texlive/2021/bin/x86_64-linux:$PATH

# fnm
export PATH=/root/.fnm:$PATH
eval "$(fnm env)"
```

### Check every thing is ready

```shell
pandoc -v
tlmgr --version
fnm list
node -v
```

### Sample pandoc script to generate pdf

```shell
pandoc --toc --toc-depth=2 --verbose -s  --resource-path=src -f markdown+link_attributses -V geometry:"left=2cm, top=2cm, right=2cm, bottom=2cm" -V title="Test PDF" -V mainfont="Symbola" -V CJKmainfont="AR PL SungtiL GB"  --pdf-engine=xelatex -o  test_cn.pdf src/test.md
```

Important arguments:
* `--toc` means generate pdf with toc
* `--toc-depth=2` means extrac only top 2 level title as toc (h1 and h2 or # and ##)
* `--verbose` print all logs while generate
* `--resource-path` the the relative path for build pdf. If you have some relateive linked asset or resources like image in source file, add the argument.
* `-V` add some extra argument like:
  * `geometry` actuall size of pdf file
  * `title` pdf title
  * `mainfont` main font for pdf
  * `CJKmainfont` main font for CJK characters in pdf
* `--pdf-engine` pdf generate engine
* `-o` output file path and file name
