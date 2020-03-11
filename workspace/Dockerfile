ARG UBUNTU_VERSION
FROM ubuntu:${UBUNTU_VERSION}

### 时区 和 apt源 #################################################
ARG TIME_ZONE
ARG CHANGE_SOURCE
ARG UBUNTU_SOURCE

RUN ln -snf /usr/share/zoneinfo/$TIME_ZONE /etc/localtime && echo $TIME_ZONE > /etc/timezone && \
    if [ ${CHANGE_SOURCE} = true ]; then \
        sed -i "s/archive.ubuntu.com/${UBUNTU_SOURCE}/g" /etc/apt/sources.list && \
        sed -i "s/security.ubuntu.com/${UBUNTU_SOURCE}/g" /etc/apt/sources.list \
    ;fi

### 安装软件 #################################################
RUN apt-get update && apt-get install -yqq \
    curl \
    git \
    # > /dev/null \
    && rm -rf /var/lib/apt/lists/*

### 创建用户 #################################################
ARG PUID=1000
ARG PGID=1000
RUN groupadd -g ${PGID} vuedock && \
    # -g: 组 -m:创建用户家目录
    useradd -u ${PUID} -g vuedock -m vuedock && \
    # -p:用户密码 -s: 用户使用的shell
    usermod -p "vuedock" vuedock -s /bin/bash

### 安装 nvm #################################################
USER vuedock

ARG NODE_NVM_VERSION
ARG NODE_VERSION
ARG NODE_ORG_MIRROR
ARG NPM_REGISTRY
ARG INSTALL_VUE_CLI
ARG NVM_DIR=/home/vuedock/.nvm

RUN git clone https://gitee.com/mirrors/nvm.git $NVM_DIR \
    && cd $NVM_DIR && git checkout ${NODE_NVM_VERSION} 2>/dev/null \
    && . $NVM_DIR/nvm.sh \
    && NVM_NODEJS_ORG_MIRROR=$NODE_ORG_MIRROR nvm install ${NODE_VERSION} \
    && nvm use ${NODE_VERSION} \
    && nvm alias ${NODE_VERSION} \
    # 添加 nvm 到 .bashrc
    && echo "" >> ~/.bashrc \
    && echo 'export NVM_DIR="'${NVM_DIR}'"' >> ~/.bashrc \
    && echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
    # npm registry
    && if [ ${NPM_REGISTRY} ]; then \
        . ~/.bashrc 2>/dev/null && npm config set registry ${NPM_REGISTRY} \
    ;fi

USER root

# 添加 nvm 到 .bashrc 与上面代码重复,为了切换用户
RUN echo "" >> ~/.bashrc \
    && echo 'export NVM_DIR="'${NVM_DIR}'"' >> ~/.bashrc \
    && echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
    # npm registry
    && if [ ${NPM_REGISTRY} ]; then \
        . ~/.bashrc 2>/dev/null && npm config set registry ${NPM_REGISTRY} \
    ;fi

### 安装 vue-cli #################################################
ARG VUE_CLI_VERSION
RUN if [ ${VUE_CLI_VERSION} ]; then \
    . ~/.bashrc 2>/dev/null && npm install -g @vue/cli@${VUE_CLI_VERSION} \
;fi

### 复制私钥 生产环境需要删除 #################################################
ARG COPY_SSH_KEY=false
COPY .ssh /tmp/.ssh
RUN if [ ${COPY_SSH_KEY} = true ]; then \
    cp -rf /tmp/.ssh /root/ \
    && chmod 400 /root/.ssh/id_rsa && echo "Host github.com\n\tStrictHostKeyChecking no\n" >> /root/.ssh/config \
;fi

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog