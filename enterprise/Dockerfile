FROM debian:jessie

ENV SPLUNK_PRODUCT splunk
ENV SPLUNK_VERSION 7.0.0
ENV SPLUNK_BUILD c8a78efdd40f
ENV SPLUNK_FILENAME splunk-${SPLUNK_VERSION}-${SPLUNK_BUILD}-Linux-x86_64.tgz

ENV SPLUNK_HOME /opt/splunk
ENV SPLUNK_APPS ${SPLUNK_HOME}/etc/apps
ENV SPLUNK_GROUP splunk
ENV SPLUNK_USER splunk
ENV SPLUNK_BACKUP_DEFAULT_ETC /var/opt/splunk
ARG DEBIAN_FRONTEND=noninteractive

# add splunk:splunk user
RUN groupadd -r ${SPLUNK_GROUP} \
    && useradd -r -m -g ${SPLUNK_GROUP} ${SPLUNK_USER}

# make the "en_US.UTF-8" locale so splunk will be utf-8 enabled by default
RUN apt-get update  && apt-get install -y --no-install-recommends apt-utils && apt-get install -y locales && rm -rf /var/lib/apt/lists/* \
	&& localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8

# pdfgen dependency
RUN apt-get update && apt-get install -y libgssapi-krb5-2 && rm -rf /var/lib/apt/lists/*

# Download official Splunk release, verify checksum and unzip in /opt/splunk
# Also backup etc folder, so it will be later copied to the linked volume
RUN apt-get update && apt-get install -y wget sudo \
    && mkdir -p ${SPLUNK_HOME} \
    && wget -qO /tmp/${SPLUNK_FILENAME} https://download.splunk.com/products/${SPLUNK_PRODUCT}/releases/${SPLUNK_VERSION}/linux/${SPLUNK_FILENAME} \
    && wget -qO /tmp/${SPLUNK_FILENAME}.md5 https://download.splunk.com/products/${SPLUNK_PRODUCT}/releases/${SPLUNK_VERSION}/linux/${SPLUNK_FILENAME}.md5 \
    && (cd /tmp && md5sum -c ${SPLUNK_FILENAME}.md5) \
    && tar xzf /tmp/${SPLUNK_FILENAME} --strip 1 -C ${SPLUNK_HOME} \
    && rm /tmp/${SPLUNK_FILENAME} \
    && rm /tmp/${SPLUNK_FILENAME}.md5 \
    && apt-get purge -y --auto-remove wget \
    && mkdir -p /var/opt/splunk \
    && cp -R ${SPLUNK_HOME}/etc ${SPLUNK_BACKUP_DEFAULT_ETC} \
    && rm -fR ${SPLUNK_HOME}/etc \
    && chown -R ${SPLUNK_USER}:${SPLUNK_GROUP} ${SPLUNK_HOME} \
    && chown -R ${SPLUNK_USER}:${SPLUNK_GROUP} ${SPLUNK_BACKUP_DEFAULT_ETC} \
    && rm -rf /var/lib/apt/lists/*

COPY entrypoint.sh /sbin/entrypoint.sh
RUN chmod +x /sbin/entrypoint.sh

# Copy new license
#COPY ./Splunk_Enterprise_Q3FY17.lic /var/opt/splunk/etc/licenses/download-trial/Splunk_Enterprise_Q3FY17.lic

# Ports Splunk Web, Splunk Daemon, KVStore, Splunk Indexing Port, Network Input, HTTP Event Collector
EXPOSE 8000/tcp 8089/tcp 8191/tcp 9997/tcp 1514 8088/tcp

WORKDIR /opt/splunk

# Configurations folder, var folder for everything (indexes, logs, kvstore)
VOLUME [ "/opt/splunk/etc", "/opt/splunk/var" ]

ENTRYPOINT ["/sbin/entrypoint.sh"]

# Downalod and install jdk8
RUN apt-get update && apt-get install -y wget sudo \
    && mkdir -p /opt/jdk/ && cd /opt/jdk \
    && wget -q --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u152-b16/aa0333dd3019491ca4f6ddbe78cdb6d0/jdk-8u152-linux-x64.tar.gz \
    && tar -zxf jdk-8u152-linux-x64.tar.gz \
    && apt-get purge -y --auto-remove wget \
    && cd /opt/jdk/jdk1.8.0_152/ \
    && rm -rf /var/lib/apt/linsts/*
ENV JAVA_HOME /opt/jdk/jdk1.8.0_152/

# Plugin install
COPY splunk_app_db_connect/ ${SPLUNK_APPS}/splunk_app_db_connect/

CMD ["start-service"]
