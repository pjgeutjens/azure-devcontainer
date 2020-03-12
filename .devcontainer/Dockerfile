#-------------------------------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See https://go.microsoft.com/fwlink/?linkid=2090316 for license information.
#-------------------------------------------------------------------------------------------------------------

FROM mcr.microsoft.com/dotnet/core/sdk:3.1

# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# [Optional] Version of Node.js to install.
ARG INSTALL_NODE="true"
ARG NODE_VERSION="lts/*"
ENV NVM_DIR=/home/vscode/.nvm

# [Optional] Install the Azure CLI
ARG INSTALL_AZURE_CLI="true"

# [Optional] Install terraform
ARG INSTALL_TERRAFORM="true"
ARG TERRAFORM_VERSION="0.12.23"

# [Optional] Install Ansible
ARG INSTALL_ANSIBLE="true"

ARG INSTALL_PWSH7_STABLE="true"

# Avoid warnings by switching to noninteractive
ENV DEBIAN_FRONTEND=noninteractive

# Configure apt and install packages
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    #
    # Verify git, process tools, lsb-release (common in install instructions for CLIs) installed
    && apt-get -y install curl vim unzip git iproute2 openssh-client procps apt-transport-https gnupg2 curl lsb-release \
    #
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support for the non-root user
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME\
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    #
    # [Optional] Install Node.js for ASP.NET Core Web Applicationss
    && if [ "$INSTALL_NODE" = "true" ]; then \
        #
        # Install nvm and Node
        mkdir ${NVM_DIR} \
        && curl -so- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash 2>&1 \
        && chown -R vscode:vscode ${NVM_DIR} \
        && /bin/bash -c "source $NVM_DIR/nvm.sh \
        && nvm install ${NODE_VERSION} \
        && nvm alias default ${NODE_VERSION}" 2>&1 \
        && INIT_STRING='[ -s "$NVM_DIR/nvm.sh" ] && \\. "$NVM_DIR/nvm.sh"  && [ -s "$NVM_DIR/bash_completion" ] && \\. "$NVM_DIR/bash_completion"' \
        && echo $INIT_STRING >> /home/vscode/.bashrc \
        && echo $INIT_STRING >> /home/vscode/.zshrc \
        && echo $INIT_STRING >> /root/.zshrc \
        #
        # Install yarn
        && curl -sS https://dl.yarnpkg.com/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/pubkey.gpg | apt-key add - 2>/dev/null \
        && echo "deb https://dl.yarnpkg.com/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
        && apt-get update \
        && apt-get -y install --no-install-recommends yarn; \
    fi \
    #
    # [Optional] Install the Azure CLI
    && if [ "$INSTALL_AZURE_CLI" = "true" ]; then \
        echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $(lsb_release -cs) main" > /etc/apt/sources.list.d/azure-cli.list \
        && curl -sL https://packages.microsoft.com/keys/microsoft.asc | apt-key add - 2>/dev/null \
        && apt-get update \
        && apt-get install -y azure-cli; \
    fi \
    #
    # [Optional] Install terraform
    && if [ "$INSTALL_TERRAFORM" = "true" ]; then \
        mkdir -p /tmp && \
        cd /tmp && \
        curl -JLOs https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip && \
        unzip terraform_${TERRAFORM_VERSION}_linux_amd64.zip -d /usr/bin; \
    fi \
     # [Optional] Install Ansible  
    && if [ "$INSTALL_ANSIBLE" = "true" ]; then \
        apt-get install -y libssl-dev libffi-dev python-dev python-pip && \
        pip install --upgrade pip && \ 
        pip install ansible[azure] && \
        mkdir -p /etc/ansible/ && \
        echo '[local]\nlocalhost\tansible_connection=local\n' > /etc/ansible/hosts; \
    fi \
    #
    # [Optional] install pwsh 7 stable
    && if [ "$INSTALL_PWSH7_STABLE" = "true" ]; then \
        apt-get install -y less locales ca-certificates libicu63 libssl1.1 libc6 libgcc1 libgssapi-krb5-2 liblttng-ust0 libstdc++6 zlib1g && \
        curl -Ls  https://github.com/PowerShell/PowerShell/releases/download/v7.0.0/powershell-7.0.0-linux-x64.tar.gz -o /tmp/powershell.tar.gz && \
        mkdir -p /opt/microsoft/powershell/7 && \
        tar zxf /tmp/powershell.tar.gz -C /opt/microsoft/powershell/7 && \
        chmod +x /opt/microsoft/powershell/7/pwsh && \
        ln -sf /opt/microsoft/powershell/7/pwsh /usr/bin/pwsh && \
        pwsh -NoLogo -NoProfile -Command "\$PSVersionTable.PSVersion"; \
    fi \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog