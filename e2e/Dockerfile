FROM bitnami/minideb:jessie
LABEL "maintainer" "Andrew Kutz <akutz@vmware.com>"

# Update the CA certificates and clean up the apt cache.
RUN apt-get update && \
    apt-get -y --no-install-recommends install \
    ca-certificates curl locales ruby tar unzip && \
    rm -rf /var/cache/apt/* /var/lib/apt/lists/*

# Set the locale so that the gist command is happy.
ENV LANG=en_US.UTF-8
ENV LC_ALL=C.UTF-8

# Install the ruby gem that enables the uploading of files as gists.
RUN gem install gist

# Download the Google Cloud SDK
RUN curl -sSL https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-217.0.0-linux-x86_64.tar.gz | \
    tar xzC /

# Download Sonobuoy
RUN curl -sSL https://github.com/heptio/sonobuoy/releases/download/v0.13.0/sonobuoy_0.13.0_linux_amd64.tar.gz | \
    tar xzC /usr/local/bin --exclude=LICENSE

# Download Terraform and place its binary in /usr/local/bin.
ENV TF_VERSION=0.11.8
ENV TF_ZIP=terraform_${TF_VERSION}_linux_amd64.zip
ENV TF_URL=https://releases.hashicorp.com/terraform/${TF_VERSION}/${TF_ZIP}
RUN curl -sSLO "${TF_URL}" && unzip "${TF_ZIP}" -d /usr/local/bin && rm -f "${TF_ZIP}"

# Copy the keepalive program to /usr/local/bin.
COPY hack/keepalive/keepalive.linux_amd64 /usr/local/bin/keepalive

# Create the directory structure.
RUN mkdir -p /tf/vmc

# Copy the assets into the /tf directory.
COPY *.tf cloud_config.yaml entrypoint.sh upload_e2e.py sonobuoy.yaml /tf/
COPY vmc/*.tf /tf/vmc/

# Make sure all of the scripts are marked as executable.
RUN chmod 0755 /tf/*.sh /tf/*.py

# The entrypoint command will be executed from the following working directory.
WORKDIR /tf

# Update the PATH to include the Google Cloud SDK.
ENV PATH=/google-cloud-sdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# The default argument for the entrypoint will drop the user into a shell.
CMD [ "sh" ]
ENTRYPOINT [ "/tf/entrypoint.sh" ]
