FROM jupyter/scipy-notebook:latest

LABEL org.opencontainers.image.source="https://github.com/MaastrichtU-IDS/jupyterlab"

ENV JUPYTER_ENABLE_LAB=yes

RUN npm install --global yarn

# Install jupyterlab autocomplete extensions
RUN conda install --quiet --yes \
    ipywidgets \
    jupyterlab-lsp \
    jupyter-lsp-python && \
    # rise && \
    ## Issue when building with GitHub Actions related to jedi package
    ## No reason why it fails, works on local docker
    ## I think someone need to learn what OCI runtime standard means
  fix-permissions $CONDA_DIR && \
  fix-permissions /home/$NB_USER

RUN pip install --upgrade pip && \
    pip install --upgrade \
      sparqlkernel 
      # jupyterlab jupyterlab-git 
      ## https://github.com/jupyterlab/jupyterlab-git
      ## They have basic issues with versions not matching between the JS and python


# Create a Python 2.x environment using conda including at least the ipython kernel
# and the kernda utility. Add any additional packages you want available for use
# in a Python 2 notebook to the first line here (e.g., pandas, matplotlib, etc.)
RUN conda create --quiet --yes -p $CONDA_DIR/envs/python2 python=2.7 ipython ipykernel kernda && \
    conda clean --all -f -y

USER root

# Create a global kernelspec in the image and modify it so that it properly activates
# the python2 conda environment.
RUN $CONDA_DIR/envs/python2/bin/python -m ipykernel install && \
$CONDA_DIR/envs/python2/bin/kernda -o -y /usr/local/share/jupyter/kernels/python2/kernel.json

USER $NB_USER

# Change to root user to install things
USER root

# # Install SPARQL kernel
RUN jupyter sparqlkernel install 

# Install Java
RUN apt-get update && \
    apt-get install default-jdk curl -y

# Nicer Bash terminal
RUN git clone --depth=1 https://github.com/Bash-it/bash-it.git ~/.bash_it && \
    bash ~/.bash_it/install.sh --silent

# Install Ijava kernel
RUN curl -L https://github.com/SpencerPark/IJava/releases/download/v1.3.0/ijava-1.3.0.zip > /opt/ijava-kernel.zip
RUN unzip /opt/ijava-kernel.zip -d /opt/ijava-kernel && \
  cd /opt/ijava-kernel && \
  python3 install.py --sys-prefix && \
  rm /opt/ijava-kernel.zip

RUN fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

USER $NB_USER

RUN jupyter labextension update --all
RUN jupyter lab build 
