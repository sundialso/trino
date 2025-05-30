# syntax=docker/dockerfile:1-labs
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
FROM ghcr.io/airlift/jvmkill:latest AS jvmkill

FROM debian:stable
ARG JDK_DOWNLOAD_LINK
ARG JDK_VERSION
ENV JAVA_HOME="/usr/lib/jvm/${JDK_VERSION}"

RUN \
    /bin/bash -c "set -xe && \
    apt-get update && \
    apt-get install -y --no-install-recommends curl ca-certificates tar gzip htop && \
    # Install JDK from the provided archive link \
    echo \"Downloading JDK from ${JDK_DOWNLOAD_LINK}\" && \
    mkdir -p \"${JAVA_HOME}\" && \
    curl -#LfS \"${JDK_DOWNLOAD_LINK}\" | tar -zx --strip 1 -C \"${JAVA_HOME}\" && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*"

RUN \
    /bin/bash -c "set -xe && \
    mkdir -p /tmp/overlay/usr/libexec/ && \
    touch /tmp/overlay/usr/libexec/grepconf.sh && \
    chmod +x /tmp/overlay/usr/libexec/grepconf.sh && \
    apt-get update && \
    apt-get install -y \
      less \
      curl grep \
      zlib1g \
      passwd \
      sudo \
      tar && \
      rm -rf /tmp/overlay/var/lib/apt/lists/*"

ARG JDK_VERSION
ARG ARCH
ENV JAVA_HOME="/usr/lib/jvm/${JDK_VERSION}"
ENV PATH=$PATH:$JAVA_HOME/bin
ENV CATALOG_MANAGEMENT=static

RUN \
    /bin/bash -c "set -xe && \
    groupadd trino --gid 1000 && \
    useradd trino --uid 1000 --gid 1000 --create-home && \
    mkdir -p /usr/lib/trino /data/trino && \
    usermod -aG sudo trino && \
    echo \"trino ALL=(ALL) NOPASSWD:ALL\" >> /etc/sudoers && \
    chown -R \"trino:trino\" /usr/lib/trino /data/trino"

ARG TRINO_VERSION
COPY --chown=trino:trino trino-cli-${TRINO_VERSION}-executable.jar /usr/bin/trino
COPY --chown=trino:trino --exclude=bin/darwin-* --exclude=bin/linux-* trino-server-${TRINO_VERSION} /usr/lib/trino
COPY --chown=trino:trino trino-server-${TRINO_VERSION}/bin/linux-${ARCH} /usr/lib/trino/bin/linux-${ARCH}
COPY --chown=trino:trino default/etc /etc/trino
COPY --chown=trino:trino --from=jvmkill /libjvmkill.so /usr/lib/trino/bin

EXPOSE 8080
USER trino:trino
CMD ["/usr/lib/trino/bin/run-trino"]
HEALTHCHECK --interval=10s --timeout=5s --start-period=10s \
  CMD /usr/lib/trino/bin/health-check
