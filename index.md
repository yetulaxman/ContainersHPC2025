---
title: Using Containers in Supercomputing Environment
author: CSC Training
---

{% assign items = site.hands-on |  sort: "title" %}

## 1. Introduction to HPC Containers
### Slides: [Brief Introduction to CSC HPC Environments](https://a3s.fi/biocontainers2025/CSC_HPC_Systems.html)
### Slides: [Introduction to Apptainer containers](https://a3s.fi/biocontainers2025/Introduction_to_containers.html)
### Slides: [Tykky Container Wrapper](https://a3s.fi/biocontainers2025/Container_wrapper.html)

###  Tutorials and exercises
{% for hands-on in items %}
{% if hands-on.topic == 'introduction' %}
1. [{{ hands-on.title }}]({{ hands-on.url | relative_url }})
{% endif %}
{% endfor %}

## 2. Building and Using HPC Containers
### Slides:  [Basic Usage of Apptainer Containers](https://a3s.fi/biocontainers2025/Basic_usage_of_Apptainer.html)
### Slides:  [Building Apptainer/Singularity container images](https://a3s.fi/biocontainers2024/Buiding_container_images.html)

###  Tutorials and exercises
{% for hands-on in items %}
{% if hands-on.topic == 'apptainer' %}
1. [{{ hands-on.title }}]({{ hands-on.url | relative_url }})
{% endif %}
{% endfor %}


## 3. Working with Docker Containers 
### Slides: [Basic introduction to Docker containers](https://a3s.fi/biocontainers2025/intro_docker_2025.html)
### Slides: [Working with containerised applications](https://a3s.fi/biocontainers2025/containerised_applications_2025.html)

###  Tutorials and exercises
{% for hands-on in items %}
{% if hands-on.topic == 'docker' %}
1. [{{ hands-on.title }}]({{ hands-on.url | relative_url }})
{% endif %}
{% endfor %}

## 4. Containers in web applications and workflows
### Slides: [Rstudio and Jupyter Notebooks](https://a3s.fi/biocontainers2025/notebooks_2025.html)
### Slides: [Brief Introduction to Bio-workflows](https://a3s.fi/biocontainers2025/intro_workflows_2025.html)
### Slides: [Running Nextflow and Snakemake workflows at CSC](https://a3s.fi/biocontainers2025/running_workflows_2025.html)
###  Tutorials and exercises
{% for hands-on in items %}
{% if hands-on.topic == 'containers' %}
1. [{{ hands-on.title }}]({{ hands-on.url | relative_url }})
{% endif %}
{% endfor %}
