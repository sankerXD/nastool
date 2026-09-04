FROM alpine:3.18
ENV LANG="C.UTF-8" \
    TZ="Asia/Shanghai" \
    NASTOOL_CONFIG="/config/config.yaml" \
    NASTOOL_AUTO_UPDATE=false \
    NASTOOL_CN_UPDATE=false \
    NASTOOL_VERSION=master \
    PS1="\u@\h:\w \$ " \
    REPO_URL="https://github.com/sankerXD/nastool.git" \
    PYPI_MIRROR="https://pypi.tuna.tsinghua.edu.cn/simple" \
    ALPINE_MIRROR="mirrors.ustc.edu.cn" \
    PUID=0 \
    PGID=0 \
    UMASK=000 \
    WORKDIR="/nas-tools"
COPY package_list.txt requirements.txt /tmp/build/
# requirements.txt 是 2022 年的锁定依赖，构建工具链需一并锁定，否则新版 setuptools 构建不了；
# fast-bencode 的 sdist 在 pip 构建隔离环境中会另行拉取最新 setuptools 而失败，故单独用
# --no-build-isolation 复用上面锁定的版本先行安装。
RUN apk add --no-cache libffi-dev \
    && apk add --no-cache $(echo $(cat /tmp/build/package_list.txt)) \
    && ln -sf /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo "${TZ}" > /etc/timezone \
    && ln -sf /usr/bin/python3 /usr/bin/python \
    && curl https://rclone.org/install.sh | bash \
    && if [ "$(uname -m)" = "x86_64" ]; then ARCH=amd64; elif [ "$(uname -m)" = "aarch64" ]; then ARCH=arm64; fi \
    && curl https://dl.min.io/client/mc/release/linux-${ARCH}/mc --create-dirs -o /usr/bin/mc \
    && chmod +x /usr/bin/mc \
    && pip install --upgrade pip \
    && pip install "setuptools==68.0.0" "wheel==0.40.0" "cython==0.29.36" \
    && pip install --no-build-isolation "fast-bencode==1.1.3" \
    && pip install -r /tmp/build/requirements.txt \
    && apk del libffi-dev \
    && npm install pm2 -g \
    && rm -rf /tmp/* /root/.cache /var/cache/apk/*
WORKDIR ${WORKDIR}
# 直接使用构建上下文中的源码（含 third_party 子模块），不再从远端 clone
COPY . ${WORKDIR}
RUN python_ver=$(python3 -V | awk '{print $2}') \
    && echo "${WORKDIR}/" > /usr/lib/python${python_ver%.*}/site-packages/nas-tools.pth \
    && echo 'fs.inotify.max_user_watches=524288' >> /etc/sysctl.conf \
    && echo 'fs.inotify.max_user_instances=524288' >> /etc/sysctl.conf \
    && git config --global pull.ff only \
    && git config --global --add safe.directory ${WORKDIR} \
    && chmod +x ${WORKDIR}/docker/entrypoint.sh
EXPOSE 3000
VOLUME ["/config"]
ENTRYPOINT ["/nas-tools/docker/entrypoint.sh"]
