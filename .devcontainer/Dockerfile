FROM ruby:2.6.3-buster as jekyll-base
# Note: You can use any Debian/Ubuntu based image you want. 


# Copy settings file so Docker extension is installed inside the container
COPY .devcontainer/settings.vscode.json /root/.vscode-remote/data/Machine/settings.json

# Configure apt
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils 2>&1

# Verify git, process tools installed
RUN apt-get -y install git procps wget nano zsh gcc make git nodejs

RUN gem install jekyll

RUN wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | zsh || true


# Install Docker CE CLI
RUN apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common lsb-release \
    && curl -fsSL https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/gpg | (OUT=$(apt-key add - 2>&1) || echo $OUT) \
    && add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]') $(lsb_release -cs) stable" \
    && apt-get update \
    && apt-get install -y docker-ce-cli


RUN curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
RUN chmod +x /usr/local/bin/docker-compose

# Clean up
#RUN apt-get autoremove -y \
 #   && apt-get clean -y \
 #   && rm -rf /var/lib/apt/lists/*
ENV DEBIAN_FRONTEND=dialog


EXPOSE 4000

