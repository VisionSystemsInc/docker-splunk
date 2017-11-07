FROM debian:jessie

SHELL ["bash", "-euxvc"]

ENV TZ="US/Eastern"

ENV SPLUNK_PRODUCT splunk
ENV SPLUNK_VERSION 7.0.0
ENV SPLUNK_BUILD c8a78efdd40f
ENV SPLUNK_FILENAME splunklight-${SPLUNK_VERSION}-${SPLUNK_BUILD}-Linux-x86_64.tgz
ENV SPLUNK_TGZ_MD5 7b968c5d994e9f96bf8b588d7b959149

ENV SPLUNK_HOME /opt/splunk
ENV SPLUNK_GROUP splunk
ENV SPLUNK_USER splunk
ENV SPLUNK_BACKUP_DEFAULT_ETC /var/opt/splunk
ARG DEBIAN_FRONTEND=noninteractive

# add splunk:splunk user
RUN groupadd -r ${SPLUNK_GROUP} \
    && useradd -r -m -g ${SPLUNK_GROUP} ${SPLUNK_USER}

# make the "en_US.UTF-8" locale so splunk will be utf-8 enabled by default
RUN apt-get update  && apt-get install -y --no-install-recommends apt-utils; \
    apt-get install -y locales && rm -rf /var/lib/apt/lists/*; \
	  localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8

# pdfgen dependency
RUN apt-get update && apt-get install -y libgssapi-krb5-2 && rm -rf /var/lib/apt/lists/*

# Download official Splunk release, verify checksum and unzip in /opt/splunk
# Also backup etc folder, so it will be later copied to the linked volume
RUN apt-get update && apt-get install -y wget sudo; \
    mkdir -p ${SPLUNK_HOME}; \
    wget -qO /tmp/${SPLUNK_FILENAME} 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=linux&version=7.0.0&product=splunk_light&filename=splunklight-7.0.0-c8a78efdd40f-Linux-x86_64.tgz&wget=true'; \
    (cd /tmp && echo "${SPLUNK_TGZ_MD5} ${SPLUNK_FILENAME}" > splunkmd5 && md5sum -c splunkmd5 && rm splunkmd5); \
    tar xzf /tmp/${SPLUNK_FILENAME} --strip 1 -C ${SPLUNK_HOME}; \
    rm /tmp/${SPLUNK_FILENAME}; \
    apt-get purge -y --auto-remove wget; \
    mkdir -p /var/opt/splunk; \
    cp -R ${SPLUNK_HOME}/etc ${SPLUNK_BACKUP_DEFAULT_ETC}; \
    rm -fR ${SPLUNK_HOME}/etc; \
    chown -R ${SPLUNK_USER}:${SPLUNK_GROUP} ${SPLUNK_HOME}; \
    chown -R ${SPLUNK_USER}:${SPLUNK_GROUP} ${SPLUNK_BACKUP_DEFAULT_ETC}; \
    rm -rf /var/lib/apt/lists/*;

COPY entrypoint.sh /sbin/entrypoint.sh
RUN chmod +x /sbin/entrypoint.sh

# Ports Splunk Web, Splunk Daemon, KVStore, Splunk Indexing Port, Network Input, HTTP Event Collector
EXPOSE 8000/tcp 8089/tcp 8191/tcp 9997/tcp 1514 8088/tcp

WORKDIR /opt/splunk

# Configurations folder, var folder for everything (indexes, logs, kvstore)
VOLUME [ "/opt/splunk/etc", "/opt/splunk/var" ]

ENTRYPOINT ["/sbin/entrypoint.sh"]
CMD ["start-service"]
