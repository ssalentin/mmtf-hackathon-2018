## What is MMTF?

The Macromolecular Transmission Format (MMTF) is a new compact binary format to transmit and store biomolecular structures for fast 3D visualization and analysis.

## MMTF Addition

This Docker image contains tools needed to run [MMTF](https://github.com/sbl-sdsc/mmtf-pyspark)
(Methods for the parallel and distributed analysis and mining of the Protein Data Bank using MMTF and Apache Spark.)

Part of the 2018 UCSD Structural Bioinformatics Hackathon - https://github.com/sbl-sdsc/mmtf-workshop-2018

From the host, [connect to an AWS instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)

```
### Replace ubuntu@ec2-00-00-000-00.compute-1.amazonaws.com with your AWS Public DNS
ssh -i "/path/to/sshkey.pem" -L 8888:0.0.0.0:8888 ubuntu@ec2-00-00-000-00.compute-1.amazonaws.com
```

From AWS (ubuntu) instance, install docker and run

```
### install Docker if needed
sudo apt-get update
sudo apt-get install -y docker.io


### run Docker image from a directory containing your Jupyter Notebook or clone it
git clone https://github.com/sbl-sdsc/mmtf-workshop-2018.git
docker run -it --rm -v `pwd`:`pwd` -w `pwd` -p 8888:8888 stevetsa/mmtf-docker

## Open a read-only Jupyter notebook stored in the container
## docker run -it --rm -w /home/jovyan/work -p 8888:8888 stevetsa/mmtf-docker
```

Point browser to http://localhost:8888/?token=abc........ (Last line of output from the previous command)

## End of Addition

![docker pulls](https://img.shields.io/docker/pulls/jupyter/base-notebook.svg) ![docker stars](https://img.shields.io/docker/stars/jupyter/base-notebook.svg) [![](https:/$

# Base Jupyter Notebook Stack

Small base image for defining your own stack

## What it Gives You

* Minimally-functional Jupyter Notebook 5.2.x (e.g., no pandoc for document conversion)
* Miniconda Python 3.x
* No preinstalled scientific computing packages
* Unprivileged user `jovyan` (uid=1000, configurable, see options) in group `users` (gid=100) with ownership over `/home/jovyan` and `/opt/conda`
* [tini](https://github.com/krallin/tini) as the container entrypoint and [start-notebook.sh](./start-notebook.sh) as the default command
* A [start-singleuser.sh](./start-singleuser.sh) script useful for running a single-user instance of the Notebook server, as required by JupyterHub
* A [start.sh](./start.sh) script useful for running alternative commands in the container (e.g. `ipython`, `jupyter kernelgateway`, `jupyter lab`)
* Options for a self-signed HTTPS certificate and passwordless `sudo`


## Basic Use

The following command starts a container with the Notebook server listening for HTTP connections on port 8888 with a randomly generated authentication token configured.

```
docker run -it --rm -p 8888:8888 jupyter/base-notebook
```

Take note of the authentication token included in the notebook startup log messages. Include it in the URL you visit to access the Notebook server or enter it in the Notebook login form.

## Notebook Options

The Docker container executes a [`start-notebook.sh` script](./start-notebook.sh) script by default. The `start-notebook.sh` script handles the `NB_UID`, `NB_GID` and `GRANT_SUDO` features documented in the next section, and then executes the `jupyter notebook`.

You can launch [JupyterLab](https://github.com/jupyterlab/jupyterlab) by setting `JUPYTER_ENABLE_LAB`:

```
docker run -it --rm -e JUPYTER_ENABLE_LAB=1 --rm -p 8888:8888 jupyter/base-notebook
```

You can pass [Jupyter command line options](https://jupyter.readthedocs.io/en/latest/projects/jupyter-command.html) through the `start-notebook.sh` script when launching the container. For example, to secure the Notebook server with a custom password hashed using `IPython.lib.passwd()` instead of the default token, run the following:

```
docker run -d -p 8888:8888 jupyter/base-notebook start-notebook.sh --NotebookApp.password='sha1:74ba40f8a388:c913541b7ee99d15d5ed31d4226bf7838f83a50e'
```

For example, to set the base URL of the notebook server, run the following:

```
docker run -d -p 8888:8888 jupyter/base-notebook start-notebook.sh --NotebookApp.base_url=/some/path
```

For example, to disable all authentication mechanisms (not a recommended practice):

```
docker run -d -p 8888:8888 jupyter/base-notebook start-notebook.sh --NotebookApp.token=''
```

You can sidestep the `start-notebook.sh` script and run your own commands in the container. See the *Alternative Commands* section later in this document for more information.

## Docker Options

You may customize the execution of the Docker container and the command it is running with the following optional arguments.

* `-e GEN_CERT=yes` - Generates a self-signed SSL certificate and configures Jupyter Notebook to use it to accept encrypted HTTPS connections.
* `-e NB_UID=1000` - Specify the uid of the `jovyan` user. Useful to mount host volumes with specific file ownership. For this option to take effect, you must run the container with `--user root`. (The `start-notebook.sh` script will `su jovyan` after adjusting the user id.)
* `-e NB_GID=100` - Specify the gid of the `jovyan` user. Useful to mount host volumes with specific file ownership. For this option to take effect, you must run the container with `--user root`. (The `start-notebook.sh` script will `su jovyan` after adjusting the group id.)
* `-e GRANT_SUDO=yes` - Gives the `jovyan` user passwordless `sudo` capability. Useful for installing OS packages. For this option to take effect, you must run the container with `--user root`. (The `start-notebook.sh` script will `su jovyan` after adding `jovyan` to sudoers.) **You should only enable `sudo` if you trust the user or if the container is running on an isolated host.**
* `-v /some/host/folder/for/work:/home/jovyan/work` - Mounts a host machine directory as folder in the container. Useful when you want to preserve notebooks and other work even after the container is destroyed. **You must grant the within-container notebook user or group (`NB_UID` or `NB_GID`) write access to the host directory (e.g., `sudo chown 1000 /some/host/folder/for/work`).**
* `--group-add users` - use this argument if you are also specifying
  a specific user id to launch the container (`-u 5000`), rather than launching the container as root and relying on NB_UID and NB_GID to set the user and group.


## SSL Certificates

You may mount SSL key and certificate files into a container and configure Jupyter Notebook to use them to accept HTTPS connections. For example, to mount a host folder containing a `notebook.key` and `notebook.crt`:

```
docker run -d -p 8888:8888 \
    -v /some/host/folder:/etc/ssl/notebook \
    jupyter/base-notebook start-notebook.sh \
    --NotebookApp.keyfile=/etc/ssl/notebook/notebook.key
    --NotebookApp.certfile=/etc/ssl/notebook/notebook.crt
```

Alternatively, you may mount a single PEM file containing both the key and certificate. For example:

```
docker run -d -p 8888:8888 \
    -v /some/host/folder/notebook.pem:/etc/ssl/notebook.pem \
    jupyter/base-notebook start-notebook.sh \
    --NotebookApp.certfile=/etc/ssl/notebook.pem
```

In either case, Jupyter Notebook expects the key and certificate to be a base64 encoded text file. The certificate file or PEM may contain one or more certificates (e.g., server, intermediate, and root).

For additional information about using SSL, see the following:

* The [docker-stacks/examples](https://github.com/jupyter/docker-stacks/tree/master/examples) for information about how to use [Let's Encrypt](https://letsencrypt.org/) certificates when you run these stacks on a publicly visible domain.
* The [jupyter_notebook_config.py](jupyter_notebook_config.py) file for how this Docker image generates a self-signed certificate.
* The [Jupyter Notebook documentation](https://jupyter-notebook.readthedocs.io/en/latest/public_server.html#using-ssl-for-encrypted-communication) for best practices about running a public notebook server in general, most of which are encoded in this image.


## Conda Environments

The default Python 3.x [Conda environment](http://conda.pydata.org/docs/using/envs.html) resides in `/opt/conda`. 

The commands `jupyter`, `ipython`, `python`, `pip`, and `conda` (among others) are available in both environments. For convenience, you can install packages into either environment regardless of what environment is currently active using commands like the following:

```
# install a package into the default (python 3.x) environment
pip install some-package
conda install some-package
```


## Alternative Commands

### start.sh

The `start.sh` script supports the same features as the default `start-notebook.sh` script (e.g., `GRANT_SUDO`), but allows you to specify an arbitrary command to execute. For example, to run the text-based `ipython` console in a container, do the following:

```
docker run -it --rm jupyter/base-notebook start.sh ipython
```

Or, to run JupyterLab instead of the classic notebook, run the following:

```
docker run -it --rm -p 8888:8888 jupyter/base-notebook start.sh jupyter lab
```

This script is particularly useful when you derive a new Dockerfile from this image and install additional Jupyter applications with subcommands like `jupyter console`, `jupyter kernelgateway`, etc.

### Others

You can bypass the provided scripts and specify your an arbitrary start command. If you do, keep in mind that certain features documented above will not function (e.g., `GRANT_SUDO`).

