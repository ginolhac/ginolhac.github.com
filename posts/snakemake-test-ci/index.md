---
title: "Testing Snakemake template"
description: "using Gitlab Continuous Integration"
date: "2023-09-07"
image: snakemake-logo.png
image-height: "100px"
draft: true
categories:
  - python
  - workflow managers
  - CI/CD
---


## Rationale

Testing software is necessary and I always do it too late.
Looking [William Landau](https://github.com/wlandau) coding (through watching his `{targets}` repo) 
it is clear that the best way to develop is to write _as the same time_:

- code
- unit tests
- documentation

When struggling on the first item, it appears difficult to concomitantly write the corresponding tests.
Not to mention that documentation could be not even considered in the first place.

This works somehow but suddenly, one repo could gain attention and documentation becomes an 
extended README. Then later, a fresh release is proven to break things, and that's bad when this 
is reported by users and not detected by the author.

Long story short, I wanted my [snakemake template](https://gitlab.lcsb.uni.lu/aurelien.ginolhac/snakemake-rna-seq) for bulk RNA-seq to be tested,
at least a short pipeline on fake data to catch obvious mistakes.

By the way, [Snakemake]() is a Python framework for managing bioinformatic workflow. That 
would deserve a post on itself.

## Inspiration from the experienced people

The template is derived from [this one](https://github.com/snakemake-workflows/rna-seq-star-deseq2) and they implemented testing. Apart from the great looking green badge, it is what I am after: run the pipeline on small data of the Yeast genome.

::: {.column-margin}
![Test passed!](test_passed.png)
:::




## CI/CD configuration

The content of the config `.gitlab-ci.yml` is:

``` yaml
services:
    - name: docker:dind

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: ""

stages:
  - test

testing:
  stage: test
  image: snakemake/snakemake:stable
  cache:
    key: ${CI_JOB_NAME}
    paths:
      - .test/.snakemake/
  script:
    - cd .test
    - snakemake --configfile config_basic/config.yaml 
      --snakefile Snakefile_test -j1 --use-singularity --show-failed-logs
    - snakemake --configfile config_basic/config.yaml 
      --snakefile Snakefile_test -j1 --report
    - NB_DEG=`wc -l results/diffexp/KO_vs_WT.diffexp.tsv | cut -f1 -d ' '`
    - test "$NB_DEG" -eq 94 || exit 1
    - diff results/diffexp/KO_vs_WT.diffexp.tsv expected/KO_vs_WT.diffexp.tsv
    - diff  <(grep -v seed trimmed_se/A1-1.settings)  <(grep -v seed expected/A1-1.settings)
    - diff  <(grep -v seed trimmed_pe/B1-1.settings)  <(grep -v seed expected/B1-1.settings)
    - diff  <(grep -v seed trimmed_pe/A2-1.settings)  <(grep -v seed expected/A2-1.settings)
    - diff  <(grep -v seed trimmed_pe/B2-1.settings)  <(grep -v seed expected/B2-1.settings)

  tags:
    - shared-cache
  when: always
```

I abstracted the specificity of our Gitlab, suppressing this line:

``` yaml
      command: ["--registry-mirror", "https://docker-registry.lcsb.uni.lu"]
```


## Gitlab repository

This repo is public and available [here](https://gitlab.lcsb.uni.lu/aurelien.ginolhac/snakemake-rna-seq)
