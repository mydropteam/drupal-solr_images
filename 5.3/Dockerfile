FROM java:openjdk-8-jre

RUN apt-get update && \
  apt-get -y install lsof && \
  rm -rf /var/lib/apt/lists/*

ENV SOLR_USER solr
ENV SOLR_UID 8983

RUN groupadd -r $SOLR_USER && \
  useradd -r -u $SOLR_UID -g $SOLR_USER $SOLR_USER

ENV SOLR_KEY EDF961FF03E647F9CA8A9C2C758051CCA3A13A7F
RUN gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$SOLR_KEY"

ENV SOLR_VERSION 5.3.2
ENV SOLR_SHA256 7c26601229ed712c639d00882f35304d87e0032028be4754d098a9b694877f48

RUN mkdir -p /opt/solr && \
  wget -nv --output-document=/opt/solr.tgz http://archive.apache.org/dist/lucene/solr/$SOLR_VERSION/solr-$SOLR_VERSION.tgz && \
  wget -nv --output-document=/opt/solr.tgz.asc http://archive.apache.org/dist/lucene/solr/$SOLR_VERSION/solr-$SOLR_VERSION.tgz.asc && \
  gpg --verify /opt/solr.tgz.asc && \
  echo "$SOLR_SHA256 */opt/solr.tgz" | sha256sum -c - && \
  tar -C /opt/solr --extract --file /opt/solr.tgz --strip-components=1 && \
  rm /opt/solr.tgz* && \
  mkdir -p /opt/solr/server/solr/lib && \
  sed -i -e 's/#SOLR_PORT=8983/SOLR_PORT=8983/' /opt/solr/bin/solr.in.sh && \
  sed -i -e '/-Dsolr.clustering.enabled=true/ a SOLR_OPTS="$SOLR_OPTS -Dsun.net.inetaddr.ttl=60 -Dsun.net.inetaddr.negative.ttl=60"' /opt/solr/bin/solr.in.sh && \
  chown -R $SOLR_USER:$SOLR_USER /opt/solr

COPY ./config /temp/solr-drupal-config

RUN mkdir -p /opt/solr/server/solr/collection1/conf && \
    mkdir -p /opt/solr/server/solr/collection1/data && \
    cd /temp/solr-drupal-config && cp -f * /opt/solr/server/solr/collection1/conf/

COPY ./core.properties /temp/core.properties
RUN cp -f /temp/core.properties /opt/solr/server/solr/collection1/core.properties
RUN chown -R solr:solr /opt/solr/server/solr

EXPOSE 8983
WORKDIR /opt/solr
USER $SOLR_USER
CMD ["/opt/solr/bin/solr", "-f"]
