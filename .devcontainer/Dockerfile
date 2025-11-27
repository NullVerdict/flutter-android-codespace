FROM debian:bookworm-slim

ARG JAVA_VERSION=21
ARG FLUTTER_CHANNEL=stable
ARG USERNAME=codespace
ARG USER_UID=1000
ARG USER_GID=$USER_UID
ARG CMDLINE_TOOLS_BOOTSTRAP=11076708

ENV JAVA_HOME=/usr/lib/jvm/java-21-amazon-corretto
ENV ANDROID_HOME=/usr/local/android-sdk
ENV FLUTTER_HOME=/usr/local/flutter
ENV GRADLE_HOME=/usr/local/gradle
ENV GRADLE_OPTS="-Dorg.gradle.daemon=false -Dorg.gradle.vfs.watch=false"
ENV PATH="${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/build-tools/latest:${JAVA_HOME}/bin:${FLUTTER_HOME}/bin:${GRADLE_HOME}/bin:${PATH}"

RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME -s /bin/bash \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        sudo wget curl git unzip xz-utils zip libglu1-mesa ca-certificates gnupg java-common jq \
    && echo "$USERNAME ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    && rm -rf /var/lib/apt/lists/*

RUN wget -q https://corretto.aws/downloads/latest/amazon-corretto-${JAVA_VERSION}-x64-linux-jdk.deb -O /tmp/java.deb \
    && apt-get update \
    && apt-get install -y /tmp/java.deb \
    && rm /tmp/java.deb

RUN mkdir -p ${ANDROID_HOME} ${FLUTTER_HOME} ${GRADLE_HOME} \
    && chown -R $USERNAME:$USERNAME ${ANDROID_HOME} ${FLUTTER_HOME} ${GRADLE_HOME}

USER $USERNAME

RUN LATEST_GRADLE=$(curl -s https://services.gradle.org/versions/current | jq -r .version) \
    && wget -q https://services.gradle.org/distributions/gradle-${LATEST_GRADLE}-bin.zip -O /tmp/gradle.zip \
    && unzip -q /tmp/gradle.zip -d /tmp \
    && mv /tmp/gradle-${LATEST_GRADLE}/* ${GRADLE_HOME}/ \
    && rm -rf /tmp/gradle*

RUN mkdir -p /tmp/android-sdk/cmdline-tools \
    && wget -q https://dl.google.com/android/repository/commandlinetools-linux-${CMDLINE_TOOLS_BOOTSTRAP}_latest.zip -O /tmp/cmdline-tools.zip \
    && unzip -q /tmp/cmdline-tools.zip -d /tmp/android-sdk/cmdline-tools \
    && mv /tmp/android-sdk/cmdline-tools/cmdline-tools /tmp/android-sdk/cmdline-tools/latest \
    && rm /tmp/cmdline-tools.zip \
    && mkdir -p ${ANDROID_HOME}/cmdline-tools \
    && mv /tmp/android-sdk/cmdline-tools/latest ${ANDROID_HOME}/cmdline-tools/latest \
    && rm -rf /tmp/android-sdk \
    && yes | ${ANDROID_HOME}/cmdline-tools/latest/bin/sdkmanager --licenses \
    && ${ANDROID_HOME}/cmdline-tools/latest/bin/sdkmanager "platform-tools" "platforms;android-34" "build-tools;34.0.0"

RUN git clone -b ${FLUTTER_CHANNEL} https://github.com/flutter/flutter.git ${FLUTTER_HOME} \
    && git config --global --add safe.directory ${FLUTTER_HOME} \
    && flutter config --no-analytics \
    && flutter precache \
    && flutter doctor -v

RUN git config --global --add safe.directory '*'
