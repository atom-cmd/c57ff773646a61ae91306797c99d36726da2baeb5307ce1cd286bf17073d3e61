

LABEL de.mindrunner.android-docker.flavour="ubuntu-standalone"

ENV ANDROID_SDK_HOME /opt/android-sdk-linux
ENV ANDROID_SDK_ROOT /opt/android-sdk-linux
ENV ANDROID_HOME /opt/android-sdk-linux
ENV ANDROID_SDK /opt/android-sdk-linux

ENV DEBIAN_FRONTEND noninteractive

# Install required tools
# Dependencies to execute Android builds

RUN dpkg --add-architecture i386 && apt-get update -yqq && apt-get install -y \
  curl \
  expect \
  git \
  make \
  libc6:i386 \
  libgcc1:i386 \
  libncurses5:i386 \
  libstdc++6:i386 \
  zlib1g:i386 \
  openjdk-8-jdk \
  wget \
  unzip \
  vim \
  openssh-client \
  locales \
  nodejs \
  npm \
  && apt-get clean

RUN  rm -rf /var/lib/apt/lists/* && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8

ENV LANG en_US.UTF-8

RUN groupadd android && useradd -d /opt/android-sdk-linux -g android -u 1000 android

COPY tools /opt/tools

COPY licenses /opt/licenses

WORKDIR /opt/android-sdk-linux

RUN /opt/tools/entrypoint.sh built-in

CMD /opt/tools/entrypoint.sh built-in was not distributed with this
# file, You can obtain one at http://mozilla.org/MPL/2.0/.

# Inspired by:
# https://hub.docker.com/r/runmymind/docker-android-sdk/~/dockerfile/

FROM ubuntu:17.10

MAINTAINER Sebastian Kaspari "skaspari@mozilla.com"

# -- System -----------------------------------------------------------------------------

RUN apt-get update -qq

RUN apt-get install -y openjdk-8-jdk \
					   curl \
					   git \
					   python \
					   python-pip \
					   locales \
					   unzip

RUN locale-gen en_US.UTF-8

# -- Android SDK ------------------------------------------------------------------------

RUN mkdir -p /opt/android-sdk
WORKDIR /opt

ENV ANDROID_BUILD_TOOLS "28.0.3"
ENV ANDROID_HOME /opt/android-sdk
ENV ANDROID_SDK_HOME /opt/android-sdk
ENV PATH ${PATH}:${ANDROID_SDK_HOME}/tools:${ANDROID_SDK_HOME}/tools/bin:${ANDROID_SDK_HOME}/platform-tools:/opt/tools:${ANDROID_SDK_HOME}/build-tools/${ANDROID_BUILD_TOOLS}

RUN curl -L https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip > sdk.zip \
	&& unzip sdk.zip -d ${ANDROID_SDK_HOME} \
	&& rm sdk.zip

RUN mkdir -p /opt/android-sdk/.android/
RUN touch /opt/android-sdk/.android/repositories.cfg

RUN yes | sdkmanager --licenses

RUN sdkmanager --verbose "platform-tools" \
    "platforms;android-28" \
    "build-tools;${ANDROID_BUILD_TOOLS}" \
    "extras;android;m2repository" \
    "extras;google;m2repository"

# Checkout source code
RUN git clone https://github.com/mozilla-mobile/fenix.git

# Build project and run gradle tasks once to pull all dependencies
WORKDIR /opt/fenix
RUN ./gradlew --no-daemon assemble test lint

# -- Post setup -------------------------------------------------------------------------

# Install taskcluster python library (used by decision tasks)
# 5.0.0 is still incompatible with taskclusterProxy, meaning no decision task is able
# to schedule the rest of the Taskcluster tasks. Please upgrade to taskcluster>=5 once
# https://bugzilla.mozilla.org/show_bug.cgi?id=1460015 is fixed
RUN pip install 'taskcluster>=4,<5'
RUN pip install arrow

# -- Cleanup ----------------------------------------------------------------------------

RUN apt-get clean
