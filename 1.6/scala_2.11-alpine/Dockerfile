###############################################################################
#  Licensed to the Apache Software Foundation (ASF) under one
#  or more contributor license agreements.  See the NOTICE file
#  distributed with this work for additional information
#  regarding copyright ownership.  The ASF licenses this file
#  to you under the Apache License, Version 2.0 (the
#  "License"); you may not use this file except in compliance
#  with the License.  You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
# limitations under the License.
###############################################################################

FROM openjdk:8-jre-alpine

# Install dependencies, bash and su-exec for easy step-down from root
RUN apk add --no-cache bash libc6-compat snappy 'su-exec>=0.2'

# Configure Flink version
ENV FLINK_VERSION=1.6.2 \
    HADOOP_SCALA_VARIANT=scala_2.11

# Prepare environment
ENV FLINK_HOME=/opt/flink
ENV PATH=$FLINK_HOME/bin:$PATH
RUN addgroup -S -g 9999 flink && \
    adduser -D -S -H -u 9999 -G flink -h $FLINK_HOME flink
WORKDIR $FLINK_HOME

ENV FLINK_URL_FILE_PATH=flink/flink-${FLINK_VERSION}/flink-${FLINK_VERSION}-bin-${HADOOP_SCALA_VARIANT}.tgz
# Not all mirrors have the .asc files
ENV FLINK_TGZ_URL=https://www.apache.org/dyn/closer.cgi?action=download&filename=${FLINK_URL_FILE_PATH} \
    FLINK_ASC_URL=https://www.apache.org/dist/${FLINK_URL_FILE_PATH}.asc

# For GPG verification instead of relying on key servers
COPY KEYS /KEYS

# Install Flink
RUN set -ex; \
  apk add --no-cache --virtual .build-deps \
    ca-certificates \
    gnupg \
    openssl \
    tar \
  ; \
  \
  wget -nv -O flink.tgz "$FLINK_TGZ_URL"; \
  wget -nv -O flink.tgz.asc "$FLINK_ASC_URL"; \
  \
  export GNUPGHOME="$(mktemp -d)"; \
  gpg --import /KEYS; \
  gpg --batch --verify flink.tgz.asc flink.tgz; \
  rm -rf "$GNUPGHOME" flink.tgz.asc; \
  \
  tar -xf flink.tgz --strip-components=1; \
  rm flink.tgz; \
  \
  apk del .build-deps; \
  \
  wget -nv -O lib/log4j-over-slf4j-1.7.7.jar "https://search.maven.org/remotecontent?filepath=org/slf4j/log4j-over-slf4j/1.7.7/log4j-over-slf4j-1.7.7.jar"; \
  wget -nv -O lib/logback-classic-1.1.3.jar "https://search.maven.org/remotecontent?filepath=ch/qos/logback/logback-classic/1.1.3/logback-classic-1.1.3.jar"; \
  wget -nv -O lib/logback-core-1.1.3.jar "https://search.maven.org/remotecontent?filepath=com/hynnet/logback-core/1.1.3/logback-core-1.1.3.jar"; \
  wget -nv -O lib/mariadb-java-client-1.5.9.jar "https://search.maven.org/remotecontent?filepath=org/mariadb/jdbc/mariadb-java-client/1.5.9/mariadb-java-client-1.5.9.jar"; \
  wget -nv -O lib/h2-1.4.197.jar "https://search.maven.org/remotecontent?filepath=com/h2database/h2/1.4.197/h2-1.4.197.jar"; \
  rm -rf lib/log4j-1.2.17.jar; \
  rm -rf lib/slf4j-log4j12-1.7.7.jar; \
  mv opt/flink-queryable-state-runtime_2.11-1.6.2.jar lib/; \
  mv opt/flink-metrics-graphite-1.6.2.jar lib/; \
  rm -rf opt/flink-ml_2.11-1.6.2.jar; \
  rm -rf opt/flink-s3-fs-hadoop-1.6.2.jar; \
  rm -rf opt/flink-s3-fs-presto-1.6.2.jar; \
  rm -rf opt/flink-sql-client-1.6.2.jar; \
  rm -rf opt/flink-streaming-python_2.11-1.6.2.jar; \
  rm -rf opt/flink-swift-fs-hadoop-1.6.2.jar; \
  rm -rf opt/flink-table_2.11-1.6.2.jar; \
  \
  chown -R flink:flink .;

# Configure container
COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
EXPOSE 6123 8081
CMD ["help"]
