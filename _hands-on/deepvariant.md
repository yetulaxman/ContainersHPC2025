---
topic: docker
title: Tutorial5(bonus) - WGS analysis with DeepVariant container 
---

## Analysis of whole genome sequencing (WGS) data using DeepVariant Apptainer (aka singularity) container
Run [DeepVariant method](https://github.com/google/deepvariant) to perform variant calling on WGS and WES data sets in Puhti supercomputing environment using Apptainer container. One needs to prepare DeepVariant Apptainer image, models and test data to run the analysis. Additionally, other input files for running DeepVariant method include 1) A reference genome in [FASTA](https://en.wikipedia.org/wiki/FASTA_format) format and its corresponding index file (.fai). 2) An aligned reads file in [BAM](http://genome.sph.umich.edu/wiki/BAM) format and its corresponding index file (.bai). For the sake of this tutorial, test data is provided as a downloadable link in the later sections. 

> This tutorial is done on interactive node of **Puhti**, which requires that:
- You have a [user account at CSC](https://docs.csc.fi/accounts/how-to-create-new-user-account/).
- Your account belongs to a project [that has access to the Puhti service](https://docs.csc.fi/accounts/how-to-add-service-access-for-project/).
  

### Expected learning from tutorial:
Upon completion of this tutorial you will learn to: 
- Prepare a Apptainer image (in this case, DeepVariant) interactively from [DockerHub](https://hub.docker.com/)
- Deploy an Apptainer container for WGS analysis as a batch job on Puhti
- Learn to use Apptainer_wrapper tool at CSC

### Run WGS analysis with DeepVariant Apptainer container on Puhti

1. First login to Puhti supercomputer using *SSH* (or by opening a login node shell in the [Puhti web interface](https://www.puhti.csc.fi/public/):
   ```bash
   ssh yourcscusername@puhti.csc.fi
   ```
2. Navigate to your *scratch* directory and prepare a folder for analysis:
   ```bash
    cd /scratch/project_xxxx/$USER   # replace xxxx with your course (or own) project number
    mkdir deepvariant && cd deepvariant
   ```
4. Start interactive session on Puhti:
   ```
    sinteractive -c 2 -m 4G -d 100
   ```
    You have to choose project number of the course  on the command prompt to start an interactive session.

5. Prepare Apptainer image from docker image for DeepVariant analysis. Here we use DeepVariant Docker image from [DockerHub](https://hub.docker.com/) with a
   specific tag (i.e., 1.2.0). You can explore more about the image on the DokcerHub. It is advisable to use LOCAL_SCRATCH for Apptainer TMPDIR and CACHEDIR. 
   Unsetting XDG_RUNTIME_DIR will silence some unnecessary warnings. We will learn more about these settings later in the course.

   ```bash
    export APPTAINER_TMPDIR=$LOCAL_SCRATCH
    export APPTAINER_CACHEDIR=$LOCAL_SCRATCH
    unset XDG_RUNTIME_DIR
    apptainer build deepvariant_cpu_1.2.0.sif docker://google/deepvariant:1.2.0
   ```
   **Please note** that this image building process for DeepVariant can take sometime as image is relatively big with several image layers.
6. Download and unpack the test data for DeepVariant analysis
   ```bash
    wget https://a3s.fi/containers-workflows/deepvariant_testdata.tar.gz
    tar -xavf deepvariant_testdata.tar.gz
   ```
7. Prepare a batch script (e.g., deepvariant_puhti.sh) to run WGS analysis on Puhti. A batch script template with all necessary information is provided below. 
  Please note that this batch script also uses a special CSC-specific [singualrity_wrapper](https://docs.csc.fi/computing/containers/run-existing/) command to set
  appropriate options automatically for running Apptainer. You are free to use plain Apptainer command by taking care of bind mounts appropriately. You
  are required to use a valid project number in the script before submitting it to Puhti cluster.
   
   ```bash
   #!/bin/bash
   #SBATCH --time=00:10:00
   #SBATCH --partition=test     # You can also choose partition : "small" for this toy example
   #SBATCH --account=project_xxxx
   #SBATCH --ntasks=1
   #SBATCH --cpus-per-task=1
   #SBATCH --mem=4000
   

   export SING_IMAGE="$PWD/deepvariant_cpu_1.2.0.sif" 
   apptainer_wrapper exec  \
   /opt/deepvariant/bin/run_deepvariant \
   --model_type=WGS   --ref=$PWD/testdata/ucsc.hg19.chr20.unittest.fasta \
   --reads=$PWD/testdata/NA12878_S1.chr20.10_10p1mb.bam \
   --regions "chr20:10,000,000-10,010,000" \
   --output_vcf=$PWD/output.vcf.gz \
   --output_gvcf=$PWD/output.g.vcf.gz
   ```
8. Submit your job to Puhti cluster

   ```bash
   sbatch -J deepvariant deepvariant_puhti.sh
   ```

9. Monitor the status of submitted Slurm job
    
   ```bash
   squeue -j <slurmjobid>
    # or
   squeue --me
    # or
   squeue -u $USER
   ```

