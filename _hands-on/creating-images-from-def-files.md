---
topic: apptainer
title: Tutorial2 - Creating Apptainer containers from definition files
---

# Creating Apptainer containers from definition files

In this tutorial we create a Apptainer container from a definition file.

There are many ways to build Apptainer containers. While writing and using definition files 
may not be the most intuitive one, it has many benefits: Definition files are easily 
upgradeabla and reusable. They are the prefereable way to to distribute containers (as opposed
to "black box" image files) and are necessary when uploading your images to most repositories.

On some systems Apptainer provides ways to build container images locally without root privileges 
with the [fakeroot function](https://apptainer.org/docs/user/latest/fakeroot.html). There will 
be various limitations depending on how the feature has been implemented. On CSC supercomputers
`fakeroot` is available. Root privileges and namespaces have bee disabled due to safety concerns.

Even if fakeroot is available, in most cases it is preferable to build containers in a system 
where you have root access. This could be your own machine or a suitable virtual machine. This 
kind of setup gives you the most options, and makes troubleshooting the easiest.

We will be building a container for [COBRA](https://github.com/linxingchen/cobra) software.

## 1. Creating a definition file

Create an empty definition file. You can use a text editor of your choice, e.g. `nano`:

```bash
nano cobra.def
```

💬 In nano you can use `ctrl + o` to save and `ctrl + x` to exit.

A definition file will have a header defining the base image, and number of sections defining
different aspects of the container. All sections are optional. We will discuss some of the most 
used options here. Please see the [Apptainer documentation](https://apptainer.org/docs/user/latest/definition_files.html) 
for a full list and detailed descriptions.

### 1.1 Selecting the base image

You typically start by selecting a suitable base image. This will have the basic operating system,
file system etc for your image. These can be selected from e.g Singularity Cloud Library, Docker 
Hub or Singularity Hub. The possible sources and options are discussed in more detail in the lecture.

Selection of base image depends on the software. If, for example, the software developer provide 
installation instructions for a certain Linux distribution, it's probably easiest to start with that.

If you plan to build an image with MPI support, it is easiesto start with something close to the host 
system. In case of Puhti this would be e,g Rocky Linux 8 or some other RHEL 8 based distribution.

In this case we are installing a Python software, so choice of operating systems is not that critical. 
In this case we'll start with Ubuntu 20.04 definition from Docker Hub.

Add the following lines to the definition file:

```text
Bootstrap: docker
From: ubuntu:24.04
```

### 1.2 Including files

When building an image on a local computer, you can include files form host file system. These are 
defined in the `%files` section. The format is `/path/in/host /path/in/container`, so to include file 
`myfile.txt` from the current directory to directory `/some/path` inside the container, you would add:

```text
%files
  myfile.txt /some/path/myfile.txt
```

To make the definition file more portable, it's best to make any necessary files available from net 
and download the from inside the container using  `git`, `wget`, `curl` or similar in the `%post` section. 

💬These tools are often not included in the base image, so you will need to install them first.

COBRA does not need any extra files, so in this case we can just omit the whole section.


### 1.3 Installing software

All commands that should be run inside the container after the base image is running, are included in section 
`%post`. These typically include all installation commands.

As these commands are executed inside the container, we usually have access to package manager applications 
for each distribution, e.g. `apt` for Ubunbtu (depends on the base image used). Commands should not be 
prefixed with `sudo`.

💬 The user appears to be root:root inside the container, but no actual root privileges are given.

Many base images are very barebones to keep the images small. For example you often have to start by installing 
the compiler environment, file transfer tools and other tools you may need. A packet manager is usually included.

The base image we chose does not include Python, so we will start by installing it. It does include packet 
manager `apt`, so we can use it to install Python. 

Add the following lines to the definition file:

```text
%post
  # Install python
  apt update
  apt install -y wget
  apt install -y python3
  apt install -y python3-pip  
```

💬Indentation is optional, but improves readability. 
💬Comments will make the definition file easier to read for others (and also for yourself after some time has passed).

We will experiment with different installation methods to test some of the options in definition files.

#### 1.3.1 Installing from PyPI

COBRA is available in PyPI, so the easiest way to install it is by using `pip`.

Add the following lines to the definition file:

```text
  # Install COBRA
  PIP_BREAK_SYSTEM_PACKAGES=1 pip install cobra-meta
```
Note that Ubuntu 24.04 does not like using pip as root, so we need to set `PIP_BREAK_SYSTEM_PACKAGES=1`

By default Apptainer saves temporary files to you home directory. As the home directory is quite small and easily fills up you may want to
use some other location. You can do this by setting some environment variables:

```bash
export APPTAINER_TMPDIR=$TMPDIR
export APPTAINER_CACHEDIR=$TMPDIR
```

You can now try to build it:

```bash 
apptainer build --fakeroot cobra.sif cobra.def
```

💬 If no root access or namespaces are detected `fakeroot`is assumed by default and
option `--fakeroot` can be omitted. We include it here for the sake of calrity.

If the build finishes, try it:

```bash
apptainer exec cobra.sif cobra-meta -h
```

#### 1.3.2 Extra task: Installing from source code

As an optional extra task you can also try to install the software
from git.

Since COBRA is still under development, we might prefer to install from 
source code instead to include any changes not yet available in PyPI. 
If we clone the latest version from git, we can update the software 
simply by re-running the `apptainer build` command.
For this we also need to install git, python3-devel and python3-setuptools packages.

Comment out or delete the pip command, and add the following to the definition file:

```text
  apt install -y python3-dev
  apt install -y python3-setuptools
  apt-mark hold openssh-client && apt install git -y
  git clone https://github.com/linxingchen/cobra.git
  cd cobra
  python3 setup.py install
```

💬 Sometimes building with `fakeroot` requires workarounds. Installing git
will by default install openssh-client. This will fail when trying  to install 
without `sudo`. 

You can now try to build it:

```bash
apptainer build --fakeroot cobra.sif cobra.def
```

If the build finishes, try it:

```bash
apptainer exec cobra.sif cobra-meta -h
```

In this case there will be an error message about pandas module not being found. 

This is a good reminder that even if build finishes there could be problems with 
the container, so you should always test.

Try adding

```bash
PIP_BREAK_SYSTEM_PACKAGES=1 pip install pandas
```

Rebuild and re-test.

#### 1.3.3 Setting up the environment

Any commands that set up the environment go to the `%environment` section.

The default `$PATH` for a Apptainer environment is

```text
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

The default installation location for software is usually one of the folders in the default 
`$PATH`, and thus we don't need to adjust the environment. 

Sometimes however, especially when using Conda or some software specific installations scripts, the 
installation may end up in some other location.

To test this, let's edit the definition file to add blastn to the image.

```text
  cd /
  wget https://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.17.0/ncbi-blast-2.17.0+-x64-linux.tar.gz
  tar xf ncbi-blast-2.17.0+-x64-linux.tar.gz
  rm ncbi-blast-2.17.0+-x64-linux.tar.gz
```

We will now need either to copy the binaries to somewhere in `$PATH` or to adjust `$PATH`

```text
%environment
  export PATH=/ncbi-blast-2.17.0+/bin:$PATH
```

💬`PATH` can also be set at runtime in the host system, by e.g. setting `$APPTAINERENV_PREPEND_PATH`
but it's more user friendly to set it in container.

For future compatibility it is best to make any changes to environment in the 
%environment section or using variable `$APPTAINER_ENVIRONMENT` in %post, instead 
of relying on any fixed file path (/environment or a file in /.singularity.d/env).

## 2. Adding additional metadata

If you are building the container image just for yourself, you may 
be happy with an image with just the basic functionality.

If you plan of sharing the image with others, it is helpfull to add 
some additional metada to the definition file.

### 2.1 Adding a runscript

Runscript is a list of commands that will be executed when the container 
is run with `apptainer run` or when the container is run as an executable. 
It is defined in section `%runscript`.

If the container has a main application that is typically run, e.g. some 
frontend script, it's usually a good idea to make runscript run that so 
the end user can simply to run the container as an executable.

One option is to make the container run any command line options as a 
command. This will make `apptainer run` behave similarily to `apptainer 
exec`. You can also choose to leave it empty or e.g. make it print out 
usage help.

In this case we could make it run cobra-meta:

```text
%runscript
  exec cobra-meta "$@"
```

You could now run the container as an executable:

```bash
chmod u+x cobra.sif
./cobra.sif -h
```


💬In a HPC system, especially with batch jobs, it's best to always use `aptainer exec` to run.

The runscript of an existing container can be checked with:

```bash
apptainer inspect --runscript cobra.sif
```

### 2.2 Adding information about the container

Apptainer automatically adds some metadata to the container. This includes Apptainer version number, build date, base image etc. If you wish to add some additional metadata, e.g your contact information, you do it in section `%labels`. This is advisable, especially if you wish to distribute the container.

```text
%labels
  Maintainer my.address@example.net
  Version v1.0
```

This information can be checked with command:

```bash
apptainer inspect --labels cobra.sif
```

### 2.3 Adding help text

It is also possible to add usage information. This goes to section `%help` and can be viewed with
command:

```bash
apptainer run-help cobra.sif
```

```text
%help
  This is a container for COBRA software
  https://github.com/linxingchen/cobra
  COBRA is distributed under MIT License
  Usage:
  apptainer exec cobra.sif cobra-meta -h
```

Try adding some additional metadata to the definition file and rebuild.

### 3 Putting it all together

You can try this definition file:

```text
Bootstrap: docker
From: ubuntu:22.04

%post
  # Install system requirements
  apt update
  apt install -y wget
  apt install -y python3
  apt install -y python3-pip
    
  # Install COBRA
  PIP_BREAK_SYSTEM_PACKAGES=1 pip install cobra-meta

  # Install BLAST
  cd  /
  wget https://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.17.0/ncbi-blast-2.17.0+-x64-linux.tar.gz
  tar xf ncbi-blast-2.17.0+-x64-linux.tar.gz
  rm ncbi-blast-2.17.0+-x64-linux.tar.gz

%environment
  export PATH=/ncbi-blast-2.17.0+/bin$PATH 

%runscript
  exec cobra-meta "$@"

%help
  This is a container for COBRA software
  https://github.com/linxingchen/cobra
  COBRA is distributed under MIT License
  Usage:
  apptainer exec cobra.sif cobra-meta -h

%labels
  Maintainer my.address@example.net
  Version v1.0

```

```bash
apptainer build --fakeroot cobra.sif cobra.def
```

If the build finishes, try it:

```bash
apptainer exec cobra.sif cobra-meta -h
apptainer exec cobra.sif blastn -h
```

Try the run script:

```bash
chmod u+x cobra.sif
./cobra.sif -h
```

You can check the information you added:

```bash
apptainer inspect --runscript cobra.sif
apptainer inspect --labels cobra.sif
apptainer run-help cobra.sif
```

