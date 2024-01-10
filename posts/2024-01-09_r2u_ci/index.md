---
title: "r2u for faster continuous integration"
author: "Aurélien Ginolhac"
date: "2024-01-09"
code-tools: false
categories: [R, CI/CD]
---


![Rocker on [DockerHub](https://hub.docker.com/r/rocker/r2u)](rocker.png){fig-align="left" width="150px"}

## Rationale

`{renv}` is a great tool to manage {{< fa brands r-project >}} dependencies. Especially it **records** 
versions of packages used in a project. This is a great feature for reproducibility, of course only if the `renv.lock` is **`git`** tracked.

However, there are situations where we don't really care about reproducibility, but we want to **speed up** the CI process. For example, when we built a teaching website or a Quarto blog.

In CI, starting from raw Ubuntu images, a `renv::restore()` with `tidyverse` and `bioconductor` packages can take **more than 30 minutes**. This is a lot of time wasted for a simple blog post or a teaching website.

## A failing solution

One solution is to use **pre-built** {{< fa brands r-project >}} packages. The [Public Posit Package Manager](https://packagemanager.posit.co/client/#/) should offer this feature (see [previous post](../renv-linux-binary)), but the different flavor of {{< fa brands linux >}} makes it over-complicated and some past versions of packages are not available. 
Even more problematic is missing system libraries are not solved. Only when you do `library(xml2)`
you realize that `libxml2-dev` is missing.

## A working solution

This is what the `{r2u}` project is about: **CRAN as Ubuntu Binaries**.

This project was initiated and is maintained by [Dirk Eddelbuettel](https://dirk.eddelbuettel.com/) and hosted [on Github](https://github.com/eddelbuettel/r2u).

Recently, it even got a [Docker image](https://hub.docker.com/r/rocker/r2u) on the [Rocker project](https://github.com/rocker-org/rocker).

### How to use it

I saw this toot from **Dirk** on [Mastodon](https://mastodon.social/@eddelbuettel/111693310455494722):

<iframe src="https://mastodon.social/@eddelbuettel/111693310455494722/embed" width="500" height="400"  allowfullscreen="allowfullscreen" sandbox="allow-scripts allow-same-origin allow-popups allow-popups-to-escape-sandbox allow-forms"></iframe>

And decided to go for a fake `DESCRIPTION` file.

I will write down the setup for a blog on Gitlab as an example.

All packages could be in the `DESCRIPTION` file but we could also cache directly in the {{< fa brands docker >}} image some packages.

#### Dockerfile

Here I have all kind of sources:

- `CRAN` packages
- `Bioconductor` packages (needs `BiocManager` first)
- `Github` packages

And Quarto, fetch the pre-release from {{< fa brands github >}}.


``` dockerfile
FROM rocker/r2u:22.04
RUN  apt-get update && \
  apt-get install -y \
  curl netbase zip git \
  libxml2-dev libcurl4-openssl-dev libmagick++-dev

RUN install.r tidyverse rmarkdown BiocManager Rcpp V8 htmlwidgets magick \
  && installBioc.r ComplexHeatmap && installGithub.r koncina/ek.plot

ARG QUARTO_VERSION="1.4.537"
RUN curl -LO https://github.com/quarto-dev/quarto-cli/releases/download/v${QUARTO_VERSION}/quarto-${QUARTO_VERSION}-linux-amd64.deb && \
    apt-get update -qq && apt-get -y install \
    ./quarto-${QUARTO_VERSION}-linux-amd64.deb && rm quarto-${QUARTO_VERSION}-linux-amd64.deb \
    && apt autoremove -y && apt clean -y && rm -rf /var/lib/apt/lists/*
```


#### DESCRIPTION file

I guess it could also be shortened to the `Imports` field only.

``` r
Package: bloguilur
Title: bloguilur Companion
Version: 1.0.0
Authors@R: 
    person("Aurélien", "Ginolhac", , "aurelien.ginolhac@uni.lu", role = c("aut", "cre"))
Description: Companion website University of Luxembourg
License: MIT + file LICENSE
URL: https://r-training.pages.uni.lu/bloguilur/
Depends:
  R (>= 4.1.0)
Imports:
  countdown,
  cropcircles,
  fontawesome,
  ggimage,
  ggplot2,
  glue,
  gt,
  gtExtras,
  huxtable,
  knitr,
  ragg
Encoding: UTF-8
Language: en-US
RoxygenNote: 7.2.3
```

#### Gitlab CI configuration

`.gitlab-ci.yml` file, showing only the `build_site` stage:

``` yaml
build_site:
  stage: build_site
  image: $CI_REGISTRY_IMAGE:latest
  cache:
    key: ${CI_JOB_NAME}
    paths:
      - _site/
      - _freeze/
  artifacts:
    name: "$CI_JOB_NAME"
    expire_in: 2 days
    paths:
      - _site/
  before_script:
    - |
      Rscript -e "read.dcf('DESCRIPTION', 'Imports') |> 
        tools:::.split_dependencies() |> 
        names() |> 
        setdiff(tools:::.get_standard_package_names()$base) |> 
        install.packages()"
  script:
    - quarto render
  tags:
    - shared-cache
  when: always
```

### Results

Building the Docker image took **5 minutes 30 seconds**.

![](docker_build1.png)
[...]
![](docker_build2.png)

Building the website took **1 minutes 7 seconds**.

![](build_site1.png)
[...]
![](build_site2.png)

Once the {{< fa brands docker >}} image is there, the site building + publishing on the {{< fa brands gitlab >}} pages took **1 minutes 47 seconds**.

## GitHub {{< fa brands github >}} Actions

For this blog, the config file is:

``` yaml
on:
  push:
    branches:
      - main

name: Build and Publish blog

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    container:
      image: rocker/r2u:latest
    env:
      GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}
    steps:
      - name: Check out repository
        uses: actions/checkout@v3
      - name: System Dependencies
        run: |
          apt update -qq && apt install --yes --no-install-recommends cmake git \
          curl jq netbase \
          libxml2-dev libcurl4-openssl-dev libmagick++-dev
      - name: Mixed packages
        run: |
          install.r tidyverse rmarkdown BiocManager Rcpp V8 htmlwidgets magick \
          && installBioc.r ComplexHeatmap && installGithub.r koncina/ek.plot
      - name: Package Dependencies
        run: |
          Rscript -e "read.dcf('DESCRIPTION', 'Imports') |> 
            tools:::.split_dependencies() |> 
            names() |> 
            setdiff(tools:::.get_standard_package_names()$base) |> 
            install.packages()"
      - name: Set up Quarto
        env:
          QUARTO_VERSION: "1.4.538"
        run: |
          curl -s -LO https://github.com/quarto-dev/quarto-cli/releases/download/v${QUARTO_VERSION}/quarto-${QUARTO_VERSION}-linux-amd64.deb && \
          apt-get update -qq && apt-get -y install \
          ./quarto-${QUARTO_VERSION}-linux-amd64.deb && rm quarto-${QUARTO_VERSION}-linux-amd64.deb
      - name: Publish Quarto blog
        uses: quarto-dev/quarto-actions/publish@v2
        with:
          target: gh-pages
```
