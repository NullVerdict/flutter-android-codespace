FROM debian:bookworm-slim

ARG USERNAME=codespace
ARG USER_UID=1000
ARG USER_GID=$USER_UID

ENV JAVA_VERSION=21
ENV ANDROID_BUILD_TOOLS_VERSION=35.0.0
ENV CMDLINE_TOOLS_VERSION=11076708

ENV ANDROID_HOME=/usr/local/android-sdk
ENV FLUTTER_HOME=/usr/local/flutter
ENV JAVA_HOME=/usr/lib/jvm/java-21
ENV PATH="${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/build-tools/${ANDROID_BUILD_TOOLS_VERSION}:${JAVA_HOME}/bin:${FLUTTER_HOME}/bin:${PATH}"

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME -s /bin/bash \
    && apt-get update \
    && apt-get install -y --no-install-recommends sudo wget git unzip xz-utils zip libglu1-mesa ca-certificates curl \
    && mkdir -p /usr/lib/jvm \
    && wget -q https://corretto.aws/downloads/latest/amazon-corretto-21-x64-linux-jdk.tar.gz -O /tmp/java.tar.gz \
    && tar -xzf /tmp/java.tar.gz -C /usr/lib/jvm \
    && mv /usr/lib/jvm/amazon-corretto-21* $JAVA_HOME \
    && rm /tmp/java.tar.gz \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    && rm -rf /var/lib/apt/lists/*

RUN mkdir -p ${ANDROID_HOME} ${FLUTTER_HOME} \
    && chown -R $USERNAME:$USERNAME ${ANDROID_HOME} ${FLUTTER_HOME}

USER $USERNAME

RUN mkdir -p ${ANDROID_HOME}/cmdline-tools \
    && wget -q https://dl.google.com/android/repository/commandlinetools-linux-${CMDLINE_TOOLS_VERSION}_latest.zip -O /tmp/cmdline-tools.zip \
    && unzip -q /tmp/cmdline-tools.zip -d ${ANDROID_HOME}/cmdline-tools \
    && rm /tmp/cmdline-tools.zip \
    && mv ${ANDROID_HOME}/cmdline-tools/cmdline-tools ${ANDROID_HOME}/cmdline-tools/latest \
    && yes | sdkmanager --licenses \
    && sdkmanager "platform-tools" "platforms;android-35" "build-tools;${ANDROID_BUILD_TOOLS_VERSION}"

RUN git clone -b stable https://github.com/flutter/flutter.git ${FLUTTER_HOME} \
    && flutter config --no-analytics \
    && flutter precache \
    && flutter doctor -v
