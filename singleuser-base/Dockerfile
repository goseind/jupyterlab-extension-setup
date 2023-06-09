FROM jupyter/scipy-notebook:python-3.9.5
LABEL author="Domenic Gosein <domenic.gosein@cern.ch>"
LABEL org.opencontainers.image.source https://github.com/goseind/jupyterlab-extension-setup

USER $NB_UID

RUN conda install python=3.8 \
    && conda install -y -c conda-forge python-gfal2 \
    && conda clean --all -f -y

RUN pip install rucio-jupyterlab>=0.9.8 PyJWT\
    && jupyter serverextension enable --py rucio_jupyterlab --sys-prefix

USER root

RUN apt update -y \
    && apt install -y build-essential \
    && apt install -y curl \
    && apt install -y voms-clients-java software-properties-common \
    && apt clean -y \
    && rm /opt/conda/bin/voms-proxy-init \
    && ln -s /usr/bin/voms-proxy-init /opt/conda/bin/voms-proxy-init

# CERN specific configuration: grid-security and VOMS setup (remove if not needed)
RUN wget -q -O - https://dist.eugridpma.info/distribution/igtf/current/GPG-KEY-EUGridPMA-RPM-3 | apt-key add -

RUN pip install ipykernel

# RUN apt update \
#     && add-apt-repository 'deb http://repository.egi.eu/sw/production/cas/1/current egi-igtf core' \
#     && apt -y install ca-certificates ca-policy-egi-core

# CERN specific configuration: grid-security and VOMS setup (remove if not needed)
RUN mkdir -p /etc/vomses \
    && wget https://indigo-iam.github.io/escape-docs/voms-config/voms-escape.cloud.cnaf.infn.it.vomses -O /etc/vomses/voms-escape.cloud.cnaf.infn.it.vomses \
    && mkdir -p /etc/grid-security/vomsdir/escape \
    && wget https://indigo-iam.github.io/escape-docs/voms-config/voms-escape.cloud.cnaf.infn.it.lsc -O /etc/grid-security/vomsdir/escape/voms-escape.cloud.cnaf.infn.it.lsc

# CERN specific configuration: Trusting the CERN CA (remove if not needed)
RUN mkdir /certs \
    && touch /certs/rucio_ca.pem \
    && curl -fsSL 'https://cafiles.cern.ch/cafiles/certificates/CERN%20Root%20Certification%20Authority%202.crt' | openssl x509 -inform DER -out /tmp/cernrootca2.crt \
    && curl -fsSL 'https://cafiles.cern.ch/cafiles/certificates/CERN%20Grid%20Certification%20Authority.crt' -o /tmp/cerngridca.crt \
    && curl -fsSL 'https://cafiles.cern.ch/cafiles/certificates/CERN%20Certification%20Authority.crt' -o /tmp/cernca.crt \
    && cat /tmp/cernrootca2.crt >> /certs/rucio_ca.pem \
    && cat /tmp/cerngridca.crt >> /certs/rucio_ca.pem \
    && cat /tmp/cernca.crt >> /certs/rucio_ca.pem

# Setup extension Rucio instance config
ADD configure.py /opt/setup/configure.py
ADD setup.sh /usr/local/bin/setup.sh
RUN fix-permissions /opt/setup \
    && fix-permissions /usr/local/bin/setup.sh \
    && sed -i -e 's/\r$/\n/' /usr/local/bin/setup.sh \
    && chmod +x /usr/local/bin/setup.sh

# CERN specific configuration: VRE Data Lake (remove if not needed)
ENV RUCIO_MODE=replica
ENV RUCIO_WILDCARD_ENABLED=1
ENV RUCIO_BASE_URL=https://vre-rucio.cern.ch
ENV RUCIO_AUTH_URL=https://vre-rucio-auth.cern.ch
ENV RUCIO_WEBUI_URL=https://vre-rucio-ui.cern.ch
ENV RUCIO_DISPLAY_NAME="VRE Data Lake"
ENV RUCIO_NAME=escape
ENV RUCIO_VOMS_ENABLED=1
ENV RUCIO_VOMS_VOMSES_PATH=/etc/vomses
ENV RUCIO_VOMS_CERTDIR_PATH=/etc/grid-security/certificates
ENV RUCIO_VOMS_VOMSDIR_PATH=/etc/grid-security/vomsdir
ENV RUCIO_CA_CERT=/certs/rucio_ca.pem
ENV RUCIO_DEFAULT_INSTANCE="vre"

RUN mkdir -p /opt/rucio \
    && chown -R $NB_UID /opt/rucio \
    && /usr/local/bin/setup.sh \
    && chown -R $NB_UID /etc/jupyter/jupyter_notebook_config.json \
    && chown -R $NB_UID /etc/jupyter/jupyter_notebook_config.py

ENV IPYTHONDIR=/etc/ipython
ADD ipython_kernel_config.json /etc/ipython/profile_default/ipython_kernel_config.json
RUN chown -R $NB_UID /etc/ipython
ENV JUPYTER_ENABLE_LAB=yes

WORKDIR $HOME
USER $NB_UID

CMD ["setup.sh", "start-notebook.sh"]
