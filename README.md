# Singularity Overview
This is a quick and fast overview for some friends of mine.
I created this to show the power of singularity, therefore  this doc only cover some topics which i like most. 

Who should check out singularity:

- everyone who does scientific programming with the need for portable code.
- people looking for an easier and more copy and go solution than docker.

## The singularity project overview

What is it? A container framework adapted for scientific workloads which often run on HPC systems (where users cant become root). 

### Why should you have a look?
Why not docker? If you ever tried to make a scheduler like SLURM speak to docker, its horrible and has security issues and has not much support for HPC environments. 
You can basically treat your singularity container as if it is an executable or standalone program.
Your image file is simply a file. You can cp it or gzip it and put it with you on your usb stick. 

If your image works on your local computer. Chances are high, that it will work on a cluster like a self build program. 

### Pages one could read to start:
* Singularity project page: http://singularity.lbl.gov/
* Installation and quick start: http://singularity.lbl.gov/quickstart
* User guide: http://singularity.lbl.gov/user-guide
* Introduction https://github.com/singularityware/singularity
* Google Group: https://groups.google.com/a/lbl.gov/forum/#!forum/singularity
* Examples: search on github
    * [Singularity project examples](https://github.com/singularityware/singularity/tree/master/examples)
    * [Tensorflow recipe](https://github.com/drorlab/tf-singularity)
    * [Python miniconda example](https://github.com/georghildebrand/singularity_ipyparallel)
    * [Singularity and Docker](http://singularity.lbl.gov/docs-docker)

## Install (Linux)
You need to do this on a machine where you are root!

    wget https://github.com/singularityware/singularity/releases/download/$VERSION/singularity-$VERSION.tar.gz
    tar xvf singularity-$VERSION.tar.gz
    cd singularity-$VERSION
    ./configure --prefix=/usr/local
    make
    sudo make install

## Basics (use a computer where you are root!)

Always use the --help command to get more information

    singularity --help
    CONTAINER USAGE COMMANDS:
        exec          Execute a command within container
        run           Launch a runscript within container
        shell         Run a Bourne shell within container
        test          Execute any test code defined within container

    CONTAINER MANAGEMENT COMMANDS (requires root):
        bootstrap     Bootstrap a new Singularity image from scratch
        copy          Copy files from your host into the container
        create        Create a new container image
        expand        Grow the container image
        export        Export the contents of a container via a tar pipe
        import        Import/add container contents via a tar pipe
        mount         Mount a Singularity container image

Example download of base alpine linux image:

Bootstrap (create/import) an image manually:

    sudo singularity create /tmp/alpine.img
    sudo singularity import /tmp/alpine.img docker://alpine:3.3

Test it:

    echo "hello world"| singularity exec /tmp/alpine.img grep "hello"
    # Should print "hello world"

Next, lets create a the file "/singularity" inside the container.
This file is executed when the container is called directly "/tmp/alpine" or via `singurlarity run` 

Lets create the file outside the container with the following content:

    $cat singularity
    echo "Arguments received: $*"
    exec grep "$@"

Now lets copy the singularity runscript file into the alpine container:

    sudo singularity copy /tmp/alpine.img singularity /singularity
    #check content
    singularity exec /tmp/alpine.img cat /singularity                                                                                                                                                 255 ↵ ──(Fr,Mai05)─┘
    echo "Arguments received: $*"
    exec grep "$@"
    # make it executable
    sudo singularity exec -w /tmp/alpine.img chmod 755 /singularity

Test it:
    
    /tmp/alpine.img arg1 arg2 
    #or
    singularity run /tmp/alpine.img arg1 arg2    

You can build powerful pipelines with only one container!!1!

## Automating builds via image.def files:

Ok, now its time to read the docs about definition files which you can find [here](http://singularity.lbl.gov/bootstrap-image).. 

Some stuff good to know about image definitions:
* Similar to dockerfile usage 
* In **%setup**:
    * Use `${SINGULARITY_ROOTFS}` variable to copy files directly to the container
* In **%post**:
    * Singularity by default takes over your `ENV` vars! 
    * Export your env variables modifying file /environment inside the container

## Other points good to know:

-   If you are using NVIDIA GPUs:
    - The container needs the same driver version as the host. Check via `nvidia smi` on the host.
-   Checkout default values in /etc/singularity/singularity.conf on your host:
    - Eg. deactivate default mount behaviour `mount home = no` 
    

# Tasks for training:
-   Try out one of the above examples
-   Use the `singularity mount` option to copy files to the container
-   Run the container via SLURM in parallel (requires that singularity is installed on your cluster! Use saltstack:-))
    - eg: `sbatch -n10 --wrap="singularity image.img hostname"`

# Todo:
- Idea: use ipyparallel and run a tensorflow job across gpu and cpu nodes comparing performance differences:
- Idea: port SPARK and run it on SLURM
