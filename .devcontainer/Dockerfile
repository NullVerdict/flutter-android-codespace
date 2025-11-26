FROM debian:bookworm-slim

ARG JAVA_VERSION=21
ARG GRADLE_VERSION=8.5
ARG ANDROID_CMDLINE_TOOLS_VERSION=11076708
ARG ANDROID_BUILD_TOOLS_VERSION=35.0.0
ARG FLUTTER_CHANNEL=stable

ARG USERNAME=codespace
ARG USER_UID=1000
ARG USER_GID=$USER_UID

ENV JAVA_HOME=/usr/lib/jvm/java-21-amazon-corretto
ENV ANDROID_HOME=/usr/local/android-sdk
ENV FLUTTER_HOME=/usr/local/flutter
ENV GRADLE_HOME=/usr/local/gradle

ENV PATH="${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/build-tools/${ANDROID_BUILD_TOOLS_VERSION}:${JAVA_HOME}/bin:${FLUTTER_HOME}/bin:${GRADLE_HOME}/bin:${PATH}"

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME -s /bin/bash \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        sudo wget curl git unzip xz-utils zip libglu1-mesa ca-certificates gnupg \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    && rm -rf /var/lib/apt/lists/*

RUN wget -q https://corretto.aws/downloads/latest/amazon-corretto-${JAVA_VERSION}-x64-linux-jdk.deb -O /tmp/java.deb \
    && dpkg -i /tmp/java.deb \
    && rm /tmp/java.deb

RUN mkdir -p ${GRADLE_HOME} \
    && wget -q https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip -O /tmp/gradle.zip \
    && unzip -q /tmp/gradle.zip -d /tmp \
    && mv /tmp/gradle-${GRADLE_VERSION}/* ${GRADLE_HOME}/ \
    && rm -rf /tmp/gradle*

RUN mkdir -p ${ANDROID_HOME} ${FLUTTER_HOME} \
    && chown -R $USERNAME:$USERNAME ${ANDROID_HOME} ${FLUTTER_HOME} ${GRADLE_HOME}

USER $USERNAME

RUN mkdir -p ${ANDROID_HOME}/cmdline-tools \
    && wget -q https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_CMDLINE_TOOLS_VERSION}_latest.zip -O /tmp/cmdline-tools.zip \
    && unzip -q /tmp/cmdline-tools.zip -d ${ANDROID_HOME}/cmdline-tools \
    && rm /tmp/cmdline-tools.zip \
    && mv ${ANDROID_HOME}/cmdline-tools/cmdline-tools ${ANDROID_HOME}/cmdline-tools/latest \
    && yes | sdkmanager --licenses \
    && sdkmanager "platform-tools" "platforms;android-34" "build-tools;${ANDROID_BUILD_TOOLS_VERSION}"

RUN git clone -b ${FLUTTER_CHANNEL} https://github.com/flutter/flutter.git ${FLUTTER_HOME} \
    && flutter config --no-analytics \
    && flutter precache \
    && flutter doctor -v
