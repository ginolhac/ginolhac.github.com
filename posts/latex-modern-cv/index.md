---
title: LaTex modern CV
author: 'Aurélien Ginolhac'
description: "How to setup latex using tinytex"
date: '2018-03-26'
categories:
  - academia
---


## tlmgr and tinytex




## install missing packages

``` bash
tlmgr install moderncv
tlmgr install xcolor
tlmgr install colortbl
tlmgr install fancyhdr
tlmgr install microtype
tlmgr install pgf # to install tikz
tlmgr install textgreek
tlmgr install fontawesome
tlmgr install lastpage
tlmgr install marvosym 
tlmgr install greek-fontenc
tlmgr install babel-greek
```

``` bash
tlmgr update --self --all
tlmgr path add
```

## issue with `fontawesome`

- reload / install fonts `fmtutil-sys --all`

side effect, the `Roboto condensed` for [Robert Rudis, ggplot2 theme](https://github.com/hrbrmstr/hrbrthemes) is now working nicely!

## solve issue of NFSS corrupted 

following [this answer](https://tex.stackexchange.com/a/304354/159133):

the error was completed with _For encoding scheme LGR the defaults cmr/m/n do not form a valid font shape_

``` bash
tlmgr install cbfonts
```

allows to get the Greek letters working. Why the `greek-fontenc` and `babel-greek` was not sufficient? I don't know


## Workflow

- bibliography from pubmed to `.bib` using http://www.bioinformatics.org/texmed/ as `biblio.bib`
- fix gamma character `{I}{F}{N}γ responses` to `{I}{F}{N}{$\gamma$} responses`
- first compilation, generate `biblio.aux`
- in `plainyr_rev.bst`, specify `"Ginolhac, A"` so my name gets underlined and bolded
- run `bibtex biblio` to generate `biblio.bbl`
- second compilation to generate the pdf
