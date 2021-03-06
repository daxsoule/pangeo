FROM jupyter/base-notebook:5811dcb711ba

USER root

RUN apt-get update \
  && apt-get install -yq gpg

RUN echo "deb http://packages.cloud.google.com/apt cloud-sdk-xenial main" > /etc/apt/sources.list.d/google-cloud-sdk.list

RUN wget https://packages.cloud.google.com/apt/doc/apt-key.gpg -q -O - | sudo apt-key add -

RUN apt-get update \
  && apt-get install -yq --no-install-recommends \
        vim git jq less openssh-client google-cloud-sdk zip unzip man-db \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

USER $NB_USER

RUN conda update -n base conda
RUN conda config --add channels conda-forge --force
RUN conda install --yes  \
    av>=0.4.0 \
    bokeh \
    cython \
    cytoolz \
    dask \
    dask-kubernetes \
    datashader \
    distributed \
    fastparquet \
    google-cloud-storage \
    pyviz/label/dev::geoviews \
    pyviz/label/dev::holoviews \
    pyviz/label/dev::hvplot \
    imagehash \
    intake::intake \
    intake::intake-xarray \
    ipywidgets \
    jupyter \
    jupyterlab=0.35 \
    jupyterlab_launcher \
    jupyter_client \
    julia \
    lz4 \
    matplotlib \
    nbserverproxy \
    netcdf4 \
    nomkl \
    numba \
    numcodecs \
    numpy \
    pandas \
    pyarrow \
    pyviz/label/dev::panel \
    pyproj \
    pytables \
    python-blosc \
    python-graphviz \
    scipy \
    scikit-image \
    xarray \
    zict \
    obspy \
    && conda clean -tipsy

RUN pip install --upgrade pip
RUN pip install numpngw \
                azure-storage-blob
RUN pip install git+https://github.com/tjcrone/pycamhd@d9cc0741f736a770dd1a19d39108026adc0deca2
RUN pip install git+https://github.com/tjcrone/driveanon

RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager
RUN jupyter labextension install @pyviz/jupyterlab_pyviz
RUN jupyter labextension install jupyterlab_bokeh

USER root
COPY prepare.sh /usr/bin/prepare.sh
RUN chmod +x /usr/bin/prepare.sh

RUN mkdir /pre-home && chown -R $NB_USER /pre-home
COPY config.yaml /pre-home
COPY worker-template.yaml /pre-home

# fix .ssh permissions
RUN printf "\n#fix .ssh permissions\nif [ -f /home/$NB_USER/.ssh/id_rsa ]; then\n   chmod 400 \"/home/$NB_USER/.ssh/id_rsa\"\nfi\nif [ -f /home/$NB_USER/.ssh/id_rsa.pub ]; then\n   chmod 444 \"/home/$NB_USER/.ssh/id_rsa.pub\"\nfi\nif [ -f /home/$NB_USER/.ssh/known_hosts ]; then\n   chmod 600 \"/home/$NB_USER/.ssh/known_hosts\"\nfi\n" >> /etc/bash.bashrc

# Add NB_USER to sudo
RUN echo "$NB_USER ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/notebook
RUN sed -ri "s#Defaults\s+secure_path=\"([^\"]+)\"#Defaults secure_path=\"\1:$CONDA_DIR/bin\"#" /etc/sudoers
USER $NB_USER

ENTRYPOINT ["tini", "--", "/usr/bin/prepare.sh"]
CMD ["start.sh jupyter lab"]
