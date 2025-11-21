FROM ubuntu:24.04

ENV GRADLE_VERSION=8.11.1
ENV ANDROID_BUILD_TOOLS_VERSION=35.0.0
ENV ANDROID_PLATFORM_VERSION=android-35
ENV CMDLINE_TOOLS_VERSION=11076708
ENV JAVA_VERSION=21

ENV ANDROID_HOME=/usr/local/android-sdk
ENV PATH="${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/build-tools/${ANDROID_BUILD_TOOLS_VERSION}:${PATH}"
ENV GRADLE_HOME=/usr/local/gradle
ENV PATH="${GRADLE_HOME}/gradle-${GRADLE_VERSION}/bin:${PATH}"
ENV JAVA_HOME=/usr/lib/jvm/java-${JAVA_VERSION}-openjdk-amd64
ENV PATH="${JAVA_HOME}/bin:${PATH}"

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    wget \
    curl \
    git \
    unzip \
    libglu1-mesa \
    openjdk-21-jdk-headless \
    && rm -rf /var/lib/apt/lists/*

RUN wget https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip -q && \
    unzip -q gradle-${GRADLE_VERSION}-bin.zip -d /usr/local/gradle && \
    rm gradle-${GRADLE_VERSION}-bin.zip

RUN mkdir -p ${ANDROID_HOME}/cmdline-tools && \
    wget https://dl.google.com/android/repository/commandlinetools-linux-${CMDLINE_TOOLS_VERSION}_latest.zip -q -O /tmp/cmdline-tools.zip && \
    unzip -q /tmp/cmdline-tools.zip -d ${ANDROID_HOME}/cmdline-tools && \
    rm /tmp/cmdline-tools.zip && \
    mv ${ANDROID_HOME}/cmdline-tools/cmdline-tools ${ANDROID_HOME}/cmdline-tools/latest && \
    yes | sdkmanager --licenses && \
    sdkmanager "platform-tools" "platforms;${ANDROID_PLATFORM_VERSION}" "build-tools;${ANDROID_BUILD_TOOLS_VERSION}"

RUN git clone https://github.com/flutter/flutter.git /usr/local/flutter && \
    git config --global --add safe.directory /usr/local/flutter
ENV PATH="/usr/local/flutter/bin:${PATH}"

RUN flutter config --no-analytics && \
    flutter precache

# Проверка
RUN flutter doctor -v && \
    java --version && \
    gradle --version
