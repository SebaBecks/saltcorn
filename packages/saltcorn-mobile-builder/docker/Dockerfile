FROM node:18

RUN apt update && apt install -y wget unzip
RUN apt install -y openjdk-17-jdk
ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64

# install android commandline tools and sdk
RUN wget https://dl.google.com/android/repository/commandlinetools-linux-8512546_latest.zip
RUN unzip commandlinetools-linux-8512546_latest.zip
RUN mkdir android_sdk
RUN yes | cmdline-tools/bin/sdkmanager --sdk_root=android_sdk --install "cmdline-tools;13.0"
RUN android_sdk/cmdline-tools/13.0/bin/sdkmanager --list
RUN android_sdk/cmdline-tools/13.0/bin/sdkmanager "platforms;android-14"
RUN android_sdk/cmdline-tools/13.0/bin/sdkmanager "build-tools;33.0.2"
RUN android_sdk/cmdline-tools/13.0/bin/sdkmanager "build-tools;34.0.0"

# download gradle
RUN wget -q https://services.gradle.org/distributions/gradle-8.4-all.zip \
    && unzip gradle-8.4-all.zip -d /opt

RUN npm install -g cordova@12.0.0

# create an empty project, the first init seems to take longer
WORKDIR /init_project
RUN cordova create project
WORKDIR /init_project/project
# download plugins while building
# so that we can add them later via file reference
RUN cordova plugin add cordova-sqlite-ext@6.0.0
RUN cordova plugin add cordova-plugin-file@8.1.0
RUN cordova plugin add cordova-plugin-inappbrowser@6.0.0
RUN cordova plugin add cordova-plugin-network-information@3.0.0
RUN cordova plugin add cordova-plugin-geolocation@5.0.0
RUN cordova plugin add cordova-plugin-camera@7.0.0
RUN cordova platform add android@^13.0.0
ENV ANDROID_SDK_ROOT=/android_sdk
ENV ANDROID_HOME=/android_sdk
ENV GRADLE_HOME=/opt/gradle-8.4
ENV PATH=$PATH:/opt/gradle-8.4/bin
# stop gradle from downloading itself
ENV CORDOVA_ANDROID_GRADLE_DISTRIBUTION_URL=file\:/gradle-8.4-all.zip
RUN cordova build android

# due to a bug in npm that has been addressed since then
# some files in '/root/.npm' are owned by root
RUN chown -R 1000:1000 "/root/.npm"
RUN chmod -R o+rwx "/root/.npm"

# prepare entry point
WORKDIR /
COPY entry.bash ./
RUN chmod u+x entry.bash

RUN chmod o+rwx ~
RUN chmod -R o+rwx ~/.config

ENTRYPOINT ["./entry.bash"]
