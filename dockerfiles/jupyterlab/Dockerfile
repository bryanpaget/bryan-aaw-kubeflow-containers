ARG BASE_VERSION=2023-08-07
FROM jupyter/datascience-notebook:$BASE_VERSION

# Must be set in deepest image
ENV DEFAULT_JUPYTER_URL=/lab 

# If using the docker bit in other Dockerfiles, this must get written over in a later layer
ENV GIT_EXAMPLE_NOTEBOOKS=https://github.com/StatCan/aaw-contrib-r-notebooks.git

ENV CS_DISABLE_FILE_DOWNLOADS=1
ENV XDG_DATA_HOME=/etc/share
ENV SERVICE_URL=https://extensions.coder.com/api

ENV PATH="/home/jovyan/.local/bin/:${PATH}"
ENV PATH=$PATH:/usr/lib/rstudio-server/bin

ARG KUBECTL_VERSION=v1.15.10
ARG KUBECTL_URL=https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl
ARG KUBECTL_SHA=38a0f73464f1c39ca383fd43196f84bdbe6e553fe3e677b6e7012ef7ad5eaf2b

ARG AZCLI_URL=https://aka.ms/InstallAzureCLIDeb

ARG TRINO_URL=https://repo1.maven.org/maven2/io/trino/trino-cli/410/trino-cli-410-executable.jar
ARG TRINO_SHA=f32c257b9cfc38e15e8c0b01292ae1f11bda2b23b5ce1b75332e108ca7bf2e9b

ARG ARGO_CLI_VERSION=v3.4.5
ARG ARGO_CLI_URL=https://github.com/argoproj/argo-workflows/releases/download/${ARGO_CLI_VERSION}/argo-linux-amd64.gz
ARG ARGO_CLI_SHA=0528ff0c0aa87a3f150376eee2f1b26e8b41eb96578c43d715c906304627d3a1 

ARG VSCODE_VERSION=4.16.1
ARG VSCODE_SHA=d468ce8e92a14effbf7e0297b95e70b644b505646ae9afb87cbce6f683e03891
ARG VSCODE_URL=https://github.com/coder/code-server/releases/download/v${VSCODE_VERSION}/code-server_${VSCODE_VERSION}_amd64.deb

ARG QUARTO_VERSION=1.4.176
ARG QUARTO_SHA=c06edd8930903a1018a27eb9f70fb9037b28a3cd8a7eb6299e8136876b4e11b3
ARG QUARTO_URL=https://github.com/quarto-dev/quarto-cli/releases/download/v${QUARTO_VERSION}/quarto-${QUARTO_VERSION}-linux-amd64.tar.gz

ARG RSTUDIO_VERSION=2023.06.2-561
ARG SHA256=f7ec39bc79b2f5ca8e653580ff15097b8275989fa1efe89f4f8ac17a926c94e3

# ---

USER root
RUN apt-get update && apt-get install -y --no-install-recommends \
    # alien is for installing RPM packages, not sure if we need this
    alien \
    # byobu is a terminal multiplexer, not sure if we need this
    byobu \
    curl \
    dirmngr \
    dos2unix \
    gdebi-core \
    htop \
    jq \
    language-pack-fr \
    less \
    libarchive-tools \
    libmagic1 \
    libpng-dev \
    libxmu6 \
    nginx \
    numactl \
    openjdk-8-jre \
    openssl \
    ranger \
    software-properties-common \
    tar \
    tig \
    tmux \
    tree \
    unixodbc \
    unixodbc-dev \
    vim \
    wget \
    zip && \
    rm -rf /var/lib/apt/lists/*

RUN pip3 --no-cache-dir install --quiet \
      fire \
      git+https://github.com/betatim/vscode-binder \
      git+https://github.com/zachomedia/s3fs@8aa929f78666ff9e323cde7d9be9262db5a17985 \
      joblib \
      # JupyterLab Stuff
      jupyter_contrib_nbextensions \
      jupyter-dash \
      jupyter-lsp \
      jupyter-rsession-proxy==2.2.0 \
      jupyter-server-proxy \
      jupyter-shiny-proxy \
      jupyterlab_execute_time \
      jupyterlab-language-pack-fr-FR \
      jupyterlab-lsp \
      # kubeflow-mpi \
      kubeflow-pytorchjob \
      kubeflow-tfjob \
      lxml \
      markupsafe \
      # maybe we can remove minio?
      minio \
      mpmath \
      oauthlib \
      pillow \
      pyjwt \
      pyodbc \
      pyyaml \
      setuptools \
      wheel && \
      fix-permissions /home/$NB_USER

# Install some packages from Anaconda using Mamba
RUN mamba install --quiet --yes \
      'r-arrow' \
      'r-aws.s3' \
      'r-catools' \
      'r-hdf5r' \
      'r-odbc' \
      'r-sf' \
      'r-markdown' \
      'r-languageserver' \
      'nb_conda_kernels' \
      'python-lsp-server' && \
    mamba clean --all -f -y && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

COPY aaw-suspend-server.sh /usr/local/bin
RUN chmod +x /usr/local/bin/aaw-suspend-server.sh

# --- CLI Stuff

# maybe we can remove this minio stuff?
COPY --from=minio/mc:RELEASE.2022-03-17T20-25-06Z /bin/mc /usr/local/bin/mc-original

# Add helpers for shell initialization
COPY shell_helpers.sh /tmp/shell_helpers.sh

RUN \ 
    # kubectl
    curl -LO "${KUBECTL_URL}" \
    && echo "${KUBECTL_SHA} kubectl" | sha256sum -c - \
    && chmod +x ./kubectl \
    && sudo mv ./kubectl /usr/local/bin/kubectl \
  && \
    # AzureCLI
    curl -sLO https://aka.ms/InstallAzureCLIDeb \
    && bash InstallAzureCLIDeb \
    && rm InstallAzureCLIDeb \
    && echo "azcli: ok" \
  && \
    # trino cli
    wget -q "${TRINO_URL}" -O /tmp/trino-original \
    && echo ${TRINO_SHA} /tmp/trino-original | sha256sum -c \
    && echo "trinocli: ok" \
    && chmod +x /tmp/trino-original \
    && sudo mv /tmp/trino-original /usr/local/bin/trino-original \
  && \
    # argo cli
    curl -sLO  ${ARGO_CLI_URL}\
    && echo "${ARGO_CLI_SHA}  argo-linux-amd64.gz"  | sha256sum -c - \
    && gunzip argo-linux-amd64.gz \
    && chmod +x argo-linux-amd64 \
    && sudo mv ./argo-linux-amd64 /usr/local/bin/argo \
    && argo version

# --- JupyterLab Stuff

USER root
RUN wget -q "${VSCODE_URL}" -O ./vscode.deb \
    && echo "${VSCODE_SHA}  ./vscode.deb" | sha256sum -c - \
    && dpkg -i ./vscode.deb \
    && rm ./vscode.deb \
    && rm -f /etc/apt/sources.list.d/vscode.list \
    && mkdir -p $HOME/.local/share \
    && mkdir -p $XDG_DATA_HOME/code-server/extensions

COPY vscode-overrides.json $XDG_DATA_HOME/code-server/Machine/settings.json

# Fix for VSCode extensions and CORS
# Languagepacks.json needs to exist for code-server to recognize the languagepack
COPY languagepacks.json $XDG_DATA_HOME/code-server/

RUN code-server --install-extension ms-python.python && \
    code-server --install-extension REditorSupport.r && \
    code-server --install-extension ms-ceintl.vscode-language-pack-fr && \
    code-server --install-extension quarto.quarto && \
    fix-permissions $XDG_DATA_HOME

RUN jupyter serverextension enable --py jupyter_server_proxy && \
    jupyter nbextension enable codefolding/main --sys-prefix && \
    jupyter labextension install --no-build \
      '@jupyterlab/translation-extension' \
      '@jupyterlab/server-proxy' \
      'jupyterlab-plotly' \
      'nbdime-jupyterlab' \
    && \
    jupyter lab build && \
    jupyter lab clean && \
  npm cache clean --force && \
  rm -rf /home/$NB_USER/.cache/yarn && \
  rm -rf /home/$NB_USER/.node-gyp && \
  fix-permissions $CONDA_DIR && \
  fix-permissions /home/$NB_USER

# Install Julia language server
RUN julia -e 'using Pkg; Pkg.add("LanguageServer")'

# Install other langauge servers
RUN npm i -g \
    'bash-language-server'  \
    'dockerfile-language-server-nodejs' \
    'javascript-typescript-langserver' \
    'unified-language-server' \
    'yaml-language-server' && \
    fix-permissions /home/$NB_USER

# Solarized Theme and Cell Execution Time
COPY jupyterlab-overrides.json /opt/conda/share/jupyter/lab/settings/overrides.json

# -- RStudio Server

RUN curl --silent -L --fail "https://download2.rstudio.org/server/jammy/amd64/rstudio-server-${RSTUDIO_VERSION}-amd64.deb" > /tmp/rstudio.deb && \
    echo "${SHA256} /tmp/rstudio.deb" | sha256sum -c - && \
    apt-get install --no-install-recommends -y /tmp/rstudio.deb && \
    rm /tmp/rstudio.deb && \
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    # Harden rstudio-server
    echo "www-frame-origin=none" >> /etc/rstudio/rserver.conf && \
    echo "www-enable-origin-check=1" >> /etc/rstudio/rserver.conf && \
    echo "www-same-site=lax" >> /etc/rstudio/rserver.conf && \
    echo "restrict-directory-view=1" >> /etc/rstudio/rsession.conf

RUN wget -q ${QUARTO_URL} -O /tmp/quarto-${QUARTO_VERSION}-linux-amd64.tar.gz && \
    echo "${QUARTO_SHA} /tmp/quarto-${QUARTO_VERSION}-linux-amd64.tar.gz" | sha256sum -c - && \
    tar -xzvf /tmp/quarto-${QUARTO_VERSION}-linux-amd64.tar.gz -C /tmp/ && \
    chmod +x /tmp/quarto-${QUARTO_VERSION} && \
    ln -s /tmp/quarto-${QUARTO_VERSION}/bin/quarto /usr/bin/quarto
    
# --- Remove Vulnerabilities

# Remove libpdfbox-java due to CVE-2019-0228. See https://github.com/StatCan/aaw-kubeflow-containers/issues/249#issuecomment-834808115 for details.
# Issue opened https://github.com/jupyter/docker-stacks/issues/1299.
# This line of code should be removed once a solution or better alternative is found.
USER root
RUN dpkg -r --force-depends libpdfbox-java \
    && rm -rf /var/lib/apt/lists/*

# --- ∞_CMD

# Configure container startup

USER root
WORKDIR /home/$NB_USER
EXPOSE 8888
COPY start-custom.sh /usr/local/bin/
# we may want to remove this mc (minio)
COPY mc-tenant-wrapper.sh /usr/local/bin/mc
COPY trino-wrapper.sh /usr/local/bin/trino

# Add --user to all pip install calls and point pip to Artifactory repository
COPY pip.conf /tmp/pip.conf
RUN cat /tmp/pip.conf >> /etc/pip.conf && rm /tmp/pip.conf \
    && pip config set global.timeout 300

# Point R to Artifactory repository
COPY Rprofile.site /tmp/Rprofile.site
RUN cat /tmp/Rprofile.site >> /opt/conda/lib/R/etc/Rprofile.site && rm /tmp/Rprofile.site

# Add .Rprofile to /tmp so we can install it in start-custom.sh
COPY .Rprofile /tmp/.Rprofile

# Point conda to Artifactory repository
RUN mamba config --add channels http://jfrog-platform-artifactory.jfrog-system:8081/artifactory/api/conda/conda-forge-remote --system && \
    mamba config --remove channels conda-forge --system && \
    mamba config --add channels http://jfrog-platform-artifactory.jfrog-system:8081/artifactory/api/conda/conda-forge-nvidia --system && \
    mamba config --add channels http://jfrog-platform-artifactory.jfrog-system:8081/artifactory/api/conda/conda-pytorch-remote --system

USER $NB_USER
ENTRYPOINT ["tini", "--"]
CMD ["start-custom.sh"]
