---
title: "Creating modules with Easybuild {{< fa trowel-bricks >}}"
description: "Getting rid of conda/mamba/venv"
date: "2024-03-25"
image: omar-flores.jpg
image-height: "100px"
categories: 
  - linux
  - HPC
format:
  html: 
    code-line-numbers: false
---



![Photo by <a href="https://unsplash.com/@designedbyflores">Omar Flores</a> on <a href="https://unsplash.com/photos/blue-red-and-white-artwork-lQT_bOWtysE">Unsplash</a>](omar-flores-lQT_bOWtysE-unsplash.jpg){height=200 fig-align="left"}
  
## Rationale

Getting rid of this hell, nicely summarized by [xkcd](https://xkcd.com/):

![[Python Environment by xkcd](https://xkcd.com/1987)](https://imgs.xkcd.com/comics/python_environment.png)

And this is not complete, because it misses [`mamba`](https://mamba.readthedocs.io/en/latest/). And since it is not enough, the same team 
released [`micromamba`](https://mamba.readthedocs.io/en/latest/installation/micromamba-installation.html).
I attended a Python class in December 2023 where the instructor recommended `micromamba` even on HPC clusters. I asked if any of `conda`, `miniconda` ... `mamba` is deprecated, the answer was "no, since they all serve different purposes".

To add one more to this madness, I recently learnt about [`pipx`](https://github.com/pypa/pipx) which again propose Python apps in isolated environments.

Now I understand why Python2 and 3 co-existed for so many years.

## Easybuild

>[Easybuild](https://easybuild.io) is a software build and installation framework that allows you to manage (scientific) software on High Performance Computing (HPC) systems in an efficient way.

I wanted to have [`SnakeMake`](https://snakemake.github.io/) running on HPC using this solution that already exists for more general software on our University [HPC](https://hpc.uni.lu). Currently, I was doing: 

``` bash
module load tools/Singularity
conda activate snakemake
```

Which is mixing up things, I need module to get `singularity`^[Now `apptainer`] but for `snakemake` I need some virtual envs.
And in my home, some different Python modules, all enclosed, I was several times reaching the hard limit of 1M inodes quota.

::: aside
![Easybuild logo](https://easybuild.io/images/easybuild_logo_horizontal_dark.png)
:::


[Xavier](https://github.com/besserox) explained me how to properly create my own `module` with Easybuild. Thanks a LOT!

## Easybuild recipe

His first advice was to look into the available [easyconfigs](https://github.com/easybuilders/easybuild-easyconfigs/tree/develop/easybuild/easyconfigs).

And indeed, `SnakeMake` was present, actually not the latest version but close to.
I went into the [PyPi homepage](https://pypi.org/project/snakemake/#files) and checked the `sha256sum` and updated to the version `8.9.0` along with the dependencies that were updated.

The resulting **easyconfig** is:

```{.bash filename="file: snakemake-8.9.0-foss-2023a.eb"}
easyblock = 'PythonBundle'

name = 'snakemake'
version = '8.9.0'

homepage = 'https://snakemake.readthedocs.io'
description = "The Snakemake workflow management system is a tool to create reproducible and scalable data analyses."

toolchain = {'name': 'foss', 'version': '2023a'}

builddependencies = [
    ('poetry', '1.5.1'),
]
dependencies = [
    ('Python', '3.11.3'),
    ('Python-bundle-PyPI', '2023.06'),
    ('SciPy-bundle', '2023.07'),
    ('GitPython', '3.1.40'),
    ('IPython', '8.14.0'),
    ('PyYAML', '6.0'),
    ('wrapt', '1.15.0'),
    ('PuLP', '2.8.0'),
]

use_pip = True
sanity_pip_check = True

exts_list = [
    ('datrie', '0.8.2', {
        'checksums': ['525b08f638d5cf6115df6ccd818e5a01298cd230b2dac91c8ff2e6499d18765d'],
    }),
    ('plac', '1.4.2', {
        'checksums': ['b0d04d9bc4875625df45982bc900e9d9826861c221850dbfda096eab82fe3330'],
    }),
    ('dpath', '2.1.6', {
        'checksums': ['f1e07c72e8605c6a9e80b64bc8f42714de08a789c7de417e49c3f87a19692e47'],
    }),
    ('yte', '1.5.4', {
        'checksums': ['d2d77e53eafca74f58234fcd3fea28cc0a719e4f3784911511e35e86594bc880'],
    }),
    ('toposort', '1.10', {
        'checksums': ['bfbb479c53d0a696ea7402601f4e693c97b0367837c8898bc6471adfca37a6bd'],
    }),
    ('throttler', '1.2.2', {
        'checksums': ['d54db406d98e1b54d18a9ba2b31ab9f093ac64a0a59d730c1cf7bb1cdfc94a58'],
    }),
    ('stopit', '1.1.2', {
        'checksums': ['f7f39c583fd92027bd9d06127b259aee7a5b7945c1f1fa56263811e1e766996d'],
    }),
    ('ConfigArgParse', '1.7', {
        'checksums': ['e7067471884de5478c58a511e529f0f9bd1c66bfef1dea90935438d6c23306d1'],
    }),
    ('argparse-dataclass', '2.0.0', {
        'modulename': 'argparse_dataclass',
        'source_tmpl': 'argparse_dataclass-%(version)s.tar.gz',
        'checksums': ['09ab641c914a2f12882337b9c3e5086196dbf2ee6bf0ef67895c74002cc9297f'],
    }),
    ('snakemake-interface-common', '1.17.1', {
        'modulename': 'snakemake_interface_common',
        'source_tmpl': 'snakemake_interface_common-%(version)s.tar.gz',
        'checksums': ['555c8218d9b68ddc1046f94a517e7d0f22e15bdc839d6ce149608d8ec137b9ae'],
    }),
    ('reretry', '0.11.8', {
        'checksums': ['f2791fcebe512ea2f1d153a2874778523a8064860b591cd90afc21a8bed432e3'],
    }),
    ('snakemake-interface-storage-plugins', '3.1.1', {
        'modulename': 'snakemake_interface_storage_plugins',
        'source_tmpl': 'snakemake_interface_storage_plugins-%(version)s.tar.gz',
        'checksums': ['d4d2b72ac964f12c5ba343639499c797316d6368dda471fba63610aec8e77cbb'],
    }),
    ('snakemake-interface-executor-plugins', '9.0.0', {
        'modulename': 'snakemake_interface_executor_plugins',
        'source_tmpl': 'snakemake_interface_executor_plugins-%(version)s.tar.gz',
        'checksums': ['22b7337d9ea4f9e32679b96fa873337608d73f2d41443cc6bde18de4549acdb7'],
    }),
    ('smart-open', '6.4.0', {
        'sources': ['smart_open-%(version)s.tar.gz'],
        'checksums': ['be3c92c246fbe80ebce8fbacb180494a481a77fcdcb7c1aadb2ea5b9c2bee8b9'],

    ('jupyter-core', '5.7.1', {
        'modulename': 'jupyter_core',
        'source_tmpl': 'jupyter_core-%(version)s.tar.gz',
        'checksums': ['de61a9d7fc71240f688b2fb5ab659fbb56979458dc66a71decd098e03c79e218'],
    }),
    ('fastjsonschema', '2.19.1', {
        'checksums': ['e3126a94bdc4623d3de4485f8d468a12f02a67921315ddc87836d6e456dc789d'],
    }),
    ('nbformat', '5.9.2', {
        'source_tmpl': '%(name)s-%(version)s-py3-none-any.whl',
        'checksums': ['1c5172d786a41b82bcfd0c23f9e6b6f072e8fb49c39250219e4acfff1efe89e9'],
    }),
    ('immutables', '0.20', {
        'checksums': ['1d2f83e6a6a8455466cd97b9a90e2b4f7864648616dfa6b19d18f49badac3876'],
    }),
    ('humanfriendly', '10.0', {
        'checksums': ['6b0b831ce8f15f7300721aa49829fc4e83921a9a301cc7f606be6686a2288ddc'],
    }),
    ('connection-pool', '0.0.3', {
        'sources': ['connection_pool-%(version)s.tar.gz'],
        'checksums': ['bf429e7aef65921c69b4ed48f3d48d3eac1383b05d2df91884705842d974d0dc'],
    }),
    ('conda-inject', '1.3.1', {
        'sources': ['conda_inject-%(version)s.tar.gz'],
        'checksums': ['9e8d902230261beba74083aae12c2c5a395e29b408469fefadc8aaf51ee441e5'],
    }),
    (name, version, {
        'checksums': ['1c36d231da92a1e37ab9f96d35346f5268949fbd1cebf4c9d429816a05538066'],
    }),
    # Also install some of the snakemake executors
    ('snakemake-executor-plugin-slurm-jobstep', '0.1.11', {
        'modulename': 'snakemake_executor_plugin_slurm_jobstep',
        'source_tmpl': 'snakemake_executor_plugin_slurm_jobstep-%(version)s.tar.gz',
        'checksums': ['cafdac937796ab0dfc0354c42380167a44a1db00c4edc98ab736a6ace2201a94'],
    }),
    ('snakemake-executor-plugin-flux', '0.1.1', {
        'modulename': 'snakemake_executor_plugin_flux',
        'source_tmpl': 'snakemake_executor_plugin_flux-%(version)s.tar.gz',
        'checksums': ['26655bd1cf5d7db5dfcfdfbd006c1db35968c0ad1772e0b010e64e6f71b00163'],
    }),
    ('snakemake-executor-plugin-slurm', '0.4.2', {
        'modulename': 'snakemake_executor_plugin_slurm',
        'source_tmpl': 'snakemake_executor_plugin_slurm-%(version)s.tar.gz',
        'checksums': ['265ffff24cdaa7929769bdbe822c39d8ac059b0642e92fc6fa9e55c9cdc7d018'],
    }),
    ('snakemake-executor-plugin-cluster-sync', '0.1.4', {
        'modulename': 'snakemake_executor_plugin_cluster_sync',
        'source_tmpl': 'snakemake_executor_plugin_cluster_sync-%(version)s.tar.gz',
        'checksums': ['6a6dcb2110d4c2ee74f9a48ea68e0fd7ddd2800672ebef00a01faa4affa835ad'],
    }),
    ('snakemake-executor-plugin-cluster-generic', '1.0.9', {
        'modulename': 'snakemake_executor_plugin_cluster_generic',
        'source_tmpl': 'snakemake_executor_plugin_cluster_generic-%(version)s.tar.gz',
        'checksums': ['ad0dc2d8bde7d4f336364bebe11a3b2209653c481ce8fbb0ae8bec81016a9a14'],
    }),
    ('snakemake-interface-report-plugins', '1.0.0', {
        'modulename': 'snakemake_interface_report_plugins',
        'source_tmpl': 'snakemake_interface_report_plugins-%(version)s.tar.gz',
        'checksums': ['02311cdc4bebab2a1c28469b5e6d5c6ac6e9c66998ad4e4b3229f1472127490f'],
    }),
]

sanity_check_paths = {
    'files': ['bin/snakemake'],
    'dirs': ['lib/python%(pyshortver)s/site-packages/snakemake'],
}

sanity_check_commands = ['snakemake --help']

moduleclass = 'tools'
```

Now we need to build it with actually all the FOSS `2023a` toolchain.

## slurm launcher

Xavier also advice to install the modules inside a _project_ folder to ease collaboration and sharing.

It takes ~ 6 hours, so the passive launcher was:

``` bash
#!/bin/bash -l
#SBATCH -N 1
#SBATCH -J eb-snakemake
#SBATCH --ntasks-per-node=1
#SBATCH -c 2
#SBATCH --mem=30GB
#SBATCH --time=0-07:00:00
#SBATCH -p batch

set -euo pipefail

export SRUN_CPUS_PER_TASK=$SLURM_CPUS_PER_TASK

# Install SnakeMake 8.9.0 with toolchain foss/2023a
# Author: Xavier Besseron / A. Ginolhac
# Date: 2024-03-20

module use "$HOME/.local/easybuild/modules/all"
module load tools/EasyBuild/4.9.0

# Installation path,
# if you want something different than ~/.local/easybuild
# for example to share with other users
mkdir -p /work/projects/xxx/easybuild/
export EASYBUILD_PREFIX=/work/projects/xxx/easybuild/

# Build directory, to setup build and avoid quota issues in home directory                                                                    
export EASYBUILD_BUILDPATH="/dev/shm"

# Build and install SnakeMake and all the required dependencies
# This can take quite some time to compile and install everything.
# GCCcore took 1h20min, Rust 1h10min. Total 5h40

# /!\ If you install in a project directory, you should this to avoid quota issues                                                            
sg xxx -c 'eb snakemake-8.9.0-foss-2023a.eb -r .'


# Use the path given in EASYBUILD_INSTALL suffixed by 'modules/all/'
# (This can be put in your ~/.bashrc if you like)
module use /work/projects/xxx/easybuild/modules/all/

# Load the SnakeMake module
module load tools/snakemake/8.9.0-foss-2023a

# Check the SnakeMake version
snakemake --version
```

See the specific recommendations as comments.

## Setting

Now that the modules are installed, we need to tell `Easybuild` where to find them.
One can use the `module use ...` command for a one-time session, or use this for more persistent usage.

``` bash
command -v module >/dev/null 2>&1 && module use /work/projects/xxx/easybuild/modules/all
```

The `command -v ...` is to avoid running it on the **access** node but only **computing** nodes.

## Testing


``` bash
$ module spider snakemake

-------------------------------------------------------------------------------------------------------------------------------------------
  tools/snakemake:
-------------------------------------------------------------------------------------------------------------------------------------------
    Description:
      The Snakemake workflow management system is a tool to create reproducible and scalable data analyses.

     Versions:
        tools/snakemake/8.4.2-foss-2023a
        tools/snakemake/8.9.0-foss-2023a

-------------------------------------------------------------------------------------------------------------------------------------------
  For detailed information about a specific "tools/snakemake" package (including how to load the modules) use the module's full name.
  Note that names that have a trailing (E) are extensions provided by other modules.
  For example:

     $ module spider tools/snakemake/8.9.0-foss-2023a
-------------------------------------------------------------------------------------------------------------------------------------------

```

``` bash
module load tools/snakemake
snakemake --version
```

``` default
8.9.0
```
