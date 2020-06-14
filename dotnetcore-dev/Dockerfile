FROM mcr.microsoft.com/dotnet/core/sdk:3.1

# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=max
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# [Optional] Version of Node.js to install.
ARG INSTALL_NODE="true"
ARG NODE_VERSION="lts/*"
ENV NVM_DIR=/usr/local/share/nvm

# [Optional] Install zsh and oh-my-zsh
ARG INSTALL_ZSH="true"

# 添加buster阿里源
ADD sources.list /etc/apt/

# Configure apt and install packages
RUN apt-get update \
    && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    #
    # Verify git, process tools, lsb-release (common in install instructions for CLIs) installed
    && apt-get -y install git openssh-client less iproute2 procps apt-transport-https gnupg2 curl lsb-release \
    #
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support for the non-root user
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME\
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    #
    # Optionally install and configure zsh
    && if [ "$INSTALL_ZSH" = "true" ] && [ ! -d "/root/.oh-my-zsh" ]; then \
        apt-get install -y zsh \
        && sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)" \
        && echo "export PATH=\$PATH:\$HOME/.local/bin" >> /root/.zshrc \
        && cp -R /root/.oh-my-zsh /home/$USERNAME \
        && cp /root/.zshrc /home/$USERNAME \
        && sed -i -e "s/\/root\/.oh-my-zsh/\/home\/$USERNAME\/.oh-my-zsh/g" /home/$USERNAME/.zshrc \
        && chown -R $USER_UID:$USER_GID /home/$USERNAME/.oh-my-zsh /home/$USERNAME/.zshrc; \
    fi \
    #
    # [Optional] Install Node.js for ASP.NET Core Web Applicationss
    && if [ "$INSTALL_NODE" = "true" ]; then \
        #
        # Install nvm and Node
        mkdir -p ${NVM_DIR} \
        && curl -so- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash 2>&1 \
        && chown -R ${USER_UID}:${USER_GID} ${NVM_DIR} \
        && /bin/bash -c "source $NVM_DIR/nvm.sh \
            && nvm alias default ${NODE_VERSION}" 2>&1 \
        && echo '[ -s "$NVM_DIR/nvm.sh" ] && \\. "$NVM_DIR/nvm.sh"  && [ -s "$NVM_DIR/bash_completion" ] && \\. "$NVM_DIR/bash_completion"' \ 
        | tee -a /home/${USERNAME}/.bashrc /home/${USERNAME}/.zshrc >> /root/.zshrc \
        && echo "if [ \"\$(stat -c '%U' ${NVM_DIR})\" != \"${USERNAME}\" ]; then sudo chown -R ${USER_UID}:root ${NVM_DIR}; fi" \
        | tee -a /root/.bashrc /root/.zshrc /home/${USERNAME}/.bashrc >> /home/${USERNAME}/.zshrc \
        && chown ${USER_UID}:${USER_GID} /home/${USERNAME}/.bashrc /home/${USERNAME}/.zshrc \
        && chown -R ${USER_UID}:root ${NVM_DIR} \
        #
        # Install yarn
        && curl -sS https://dl.yarnpkg.com/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/pubkey.gpg | apt-key add - 2>/dev/null \
        && echo "deb https://dl.yarnpkg.com/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
        && apt-get update \
        && apt-get -y install --no-install-recommends yarn; \
    fi \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*
