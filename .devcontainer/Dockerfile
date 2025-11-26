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
ENV PATH="${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/build-tools/active:${JAVA_HOME}/bin:${FLUTTER_HOME}/bin:${GRADLE_HOME}/bin:${PATH}"

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

RUN mkdir -p ${ANDROID_HOME}/cmdline-tools \
    && wget -q https://dl.google.com/android/repository/commandlinetools-linux-${CMDLINE_TOOLS_BOOTSTRAP}_latest.zip -O /tmp/cmdline-tools.zip \
    && unzip -q /tmp/cmdline-tools.zip -d /tmp \
    && mv /tmp/cmdline-tools ${ANDROID_HOME}/cmdline-tools/latest \
    && rm /tmp/cmdline-tools.zip \
    && yes | sdkmanager --licenses \
    && sdkmanager "cmdline-tools;latest" "platform-tools" \
    && LATEST_BUILD_TOOLS=$(sdkmanager --list | grep "build-tools;" | grep -v "rc" | sort -V | tail -n 1 | cut -d '|' -f 1 | xargs) \
    && sdkmanager "$LATEST_BUILD_TOOLS" "platforms;android-34" \
    && BUILD_TOOLS_VERSION=$(echo $LATEST_BUILD_TOOLS | cut -d ';' -f 2) \
    && ln -s ${ANDROID_HOME}/build-tools/${BUILD_TOOLS_VERSION} ${ANDROID_HOME}/build-tools/active

RUN git clone -b ${FLUTTER_CHANNEL} https://github.com/flutter/flutter.git ${FLUTTER_HOME} \
    && flutter config --no-analytics \
    && flutter precache \
    && flutter doctor -v
