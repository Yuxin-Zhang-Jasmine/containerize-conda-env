# Containerize an existing conda environment


This repository is based on [the contribution of Gregor Sturm](https://github.com/Yuxin-Zhang-Jasmine/containerize-conda).

In my case, I need to execute my deep learning codes in our High Performance Computing (HPC) cluster and it is required to
- use singularity to containerize an existing conda environment;
- make the image based on Ubuntu (because the existing conda environment is in my local machine with Ubuntu OS);
- activate automatically the containerized conda environment once the container is started.
 
I tried "conda-pack" but it doesn't work, therefore I applied conda_to_singularity.py given by Gregor Sturm without modifying anything and edited his singularity.template to fit in my case. The file singularity.template in the current repository is the edited one.

## Usage

(exactly the same way as in [Gregor Sturm's repository](https://github.com/Yuxin-Zhang-Jasmine/containerize-conda).)

```
usage: conda_to_singularity.py [-h] [--template TEMPLATE] CONDA_ENV OUTPUT_CONTAINER

Convert a conda env to a singularity container.

positional arguments:
  CONDA_ENV            Absolute path to the conda enviornment. Must be exactely the path as it shows up in `conda env list`, not a symbolic link to it, nor a realpath.
  OUTPUT_CONTAINER     Output path where the singularity container will be safed.

optional arguments:
  -h, --help           show this help message and exit
  --template TEMPLATE  Path to a Singularity template file. Must contain a `{conda_env}` placeholder. If not specified, uses the default template shipped with this script.
```

For example

```
conda_to_singularity.py /home/sturm/.conda/envs/whatever whatever.sif
```

By default, the image will be based on Ubuntu. If you want a different base image,
you can modify `Singularity.template`, and specify it with the `--template` argument.


## Clarification

### for Ubuntu:
1. modify the Header of the singularity definition file to
```
Bootstrap: debootstrap
MirrorURL: http://us.archive.ubuntu.com/ubuntu/
OSVersion: jammy
```
where 
- the mandatory Bootstrap keyword describes the bootstrap module to use. The debootstrap build agent module allows us to build a Debian/Ubuntu style container from a mirror URI. And it is mandatory to specify the OS version and a URI for the mirror when we use the debootstrap module to specify a base for a Debian-like container. 

- jammy corresponds to Ubuntu 22.04 according to [the website of the Ubuntu releases](https://wiki.ubuntu.com/Releases). Similarily you can use other code words like trusty (14.04), xenial (16.04), and yakkety (17.04) for Ubuntu. (More info. can be found [here](https://docs.sylabs.io/guides/3.5/user-guide/appendix.html).)

2. modify the post section as
```
%post
  apt-get update
  apt-get install -y tar curl
```

### for the automatic activation 

The originally repository by Gregor Sturm has the line below in the singularity definition file to append a line to the Singularity environment file in order to activates the conda environment once the container is started
```
echo "source /opt/conda/bin/activate {conda_env}" >>$SINGULARITY_ENVIRONMENT
```
However, this doesn't work for me because of two reasons:
 1. my UNIX shell does not support source. instead, it support .
 2. activate cannot be executed in my case.
And in the updated version, I use the two lines below to replace the line above,

```
echo ". /opt/conda/etc/profile.d/conda.sh" >>$SINGULARITY_ENVIRONMENT
echo "conda activate {conda_env}" >>$SINGULARITY_ENVIRONMENT
```
where the renewed version appends the conda command to the singularity environment and then use conda command to do activation. Because conda activate is also written in the singularity environment, this command would be executed once the container is started.


### others in the def file
- In the environment section, I added
```
export LC_ALL=C
```
- In the post section, I modified miniconda to be the latest version
```
curl https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh > /install_conda.sh
```
- In the post section, I added
```
chmod -R o+rx /opt/conda
```


