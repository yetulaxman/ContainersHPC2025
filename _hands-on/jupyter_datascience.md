---
topic: containers
title: Tutorial1 - Launch custom Jupyter notebook 
---

# Launching a custom Jupyter notebook from a pre-built container image

> This tutorial is done on **Puhti and its web interface**, which requires that:

- You have a [user account at CSC](https://docs.csc.fi/accounts/how-to-create-new-user-account/).
- Your account belongs to a project [that has access to the Puhti service](https://docs.csc.fi/accounts/how-to-add-service-access-for-project/).


💬 In this tutorial, you will learn how to:
  - Install Jupyter notebook based on a pre-existing docker image from a docker registry.
  - Launch the Jupyter notebook from [Puhti Web Interface](https://www.puhti.csc.fi/public/).
    
## Search a data science notebook image from a Docker registry

We can use a data science Jupyter notebook managed by the [Jupyter project](https://github.com/jupyter) as an example here. One can find a ready-made Docker container image in Docker registries such as [Docker Hub](https://hub.docker.com/) and [Red Hat Quay](https://quay.io/). On these registries, you can search with the keyword "data sciene" and then select "jupyter/datascience-notebook" to see the images for Jupyter notebook.

## Install a Jupyter notebook docker image to */projappl* directory using *tykky* container wrapper
While the container images can be pulled and used directly as singularity containers on HPC systems, CSC’s [Tykky](https://docs.csc.fi/computing/containers/tykky/) container wrapper serves as an easy installation method for containers. For the purpose of this tutorial, we will use a data science docker image from [Docker Hub](https://hub.docker.com/r/jupyter/datascience-notebook). You can install Jupyter notebook environment to */projappl* directory as below:  

```bash
# Navigate to /projappl area of your course project 
cd /projappl/project_20xxxx/      # Make sure to replace 20xxxx with correct course project number
module purge                      # Clean your environment
module load tykky                 # load the Tykky container wrapper
mkdir -p /projappl/project_20xxxx/$USER  &&  mkdir -p /projappl/project_20xxxx/$USER/Notebook
# You can use the wrap-container command from tykky module to install image binaries to /projappl
wrap-container -w /opt/conda/bin docker://docker.io/jupyter/datascience-notebook:x86_64-ubuntu-22.04 --prefix /projappl/project_200xxxx/$USER/Notebook
# This installation can take for a while
# The -w option specifies the installation directory inside the container. For this data science container image, path is /opt/conda/bin
# The --prefix option specifies the directory where we want to install the software on the host system.
```
Upon succesful installation, the executables of the Jupyter notebook will be available in the directory /projappl/project_200xxxx/$USER/Notebook/bin. 

## Download a simple Python notebook for testing the installed environment

You can download an example Python notebook to perform basic data analysis tasks inside the installed Jupyter notebook as below: 
```bash
cd /scratch/project_200xxxx/$USER
wget https://a3s.fi/biocontainers2024/course_notebook.tar.gz
tar -xavf course_notebook.tar.gz
```

## Launch the installed Jupyter notebook from the Puhti web interface

1. Login to [Puhti web interface](https://www.puhti.csc.fi)
2. Login with CSC/HAKA/VIRTU credentials 
3. Once login is successful, select "Jupyter" icon from the pinned apps on the landing page.  Then open the Jupyter notebook and use the following settings:
    Reservation: use course reservation if available.  
    Project: project_2003682  
    Partition: small  
    Number of CPU cores: 2  
    Memory (Gb): 2  
    Local disk: 0  
    Time: 0:45:00  
    Python: custom path  
    Custom Python interpreter: /projappl/project_2003682/$USER/Notebook/bin/python  (please replace $USER with your CSC username)
    Working directory: /scratch/project_2003682/  
    and finally "Launch" notebook  
   
 4. This may take a while before seeing "Connect to Jupyter" button upon successful launching of notebook. Once Jupyter notebook is opened, navigate to your own $USER area (under /scratch/project_200xxxx/) where you have downloaded Python notebook earlier. Finally, click "explore_datascience.ipynb" to open a Python notebook on your browser.

‼️  Please note that the course reservation (name: container_course) field on Puhti web interface will only on the course day(s) for the members of course project

###  Useful CSC documentation

- [Tykky containerisation](https://docs.csc.fi/computing/containers/tykky/)
- [Jupyter for course](https://docs.csc.fi/computing/webinterface/jupyter-for-courses/)

