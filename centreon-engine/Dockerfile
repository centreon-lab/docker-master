FROM alpine:3.9

ENV CENTREON_CLIB_VERSION 18.10.0
ENV CENTREON_ENGINE_VERSION 18.10.0

RUN apk --update --no-cache add \
        g++ \
        make \
        cmake \
        libstdc++ \
        postfix \
        dropbear \
        supervisor

COPY files/supervisord/dropbear.ini /etc/supervisor.d/dropbear.ini
COPY files/supervisord/centreon-engine.ini /etc/supervisor.d/centreon-engine.ini
COPY files/supervisord/postfix.ini /etc/supervisor.d/postfix.ini
COPY files/start.sh /start.sh

RUN cd /tmp \
  && wget -O centreon-clib.zip https://github.com/centreon/centreon-clib/archive/${CENTREON_CLIB_VERSION}.zip \
  && unzip centreon-clib.zip \
  && cd centreon-clib-${CENTREON_CLIB_VERSION}/build \
  && cmake \
   -DWITH_TESTING=0 \
   -DWITH_PREFIX=/usr/local \
   -DWITH_PREFIX_LIB=/usr/lib \
   -DWITH_PREFIX_INC=/usr/include/centreon-clib \
   -DWITH_SHARED_LIB=1 \
   -DWITH_STATIC_LIB=0 \
   -DWITH_PKGCONFIG_DIR=/usr/lib/pkgconfig . \
  && make \
  && make install \
  && adduser -s /bin/sh -h /var/lib/centreon-engine -D centreon-engine \
  && cd /tmp \
  && wget -O centreon-engine.zip https://github.com/centreon/centreon-engine/archive/${CENTREON_ENGINE_VERSION}.zip \
  && unzip centreon-engine.zip \
  && cd centreon-engine-${CENTREON_ENGINE_VERSION}/build \
  && cmake \
   -DWITH_PREFIX=/usr/local \
   -DWITH_PREFIX_BIN=/usr/local/sbin \
   -DWITH_PREFIX_CONF=/etc/centreon-engine \
   -DWITH_PREFIX_LIB=/usr/local/lib/centreon-engine \
   -DWITH_USER=centreon-engine \
   -DWITH_GROUP=centreon-engine \
   -DWITH_LOGROTATE_SCRIPT=0 \
   -DWITH_VAR_DIR=/var/log/centreon-engine \
   -DWITH_RW_DIR=/var/lib/centreon-engine/rw \
   -DWITH_PKGCONFIG_SCRIPT=1 \
   -DWITH_PKGCONFIG_DIR=/usr/lib/pkgconfig \
   -DWITH_TESTING=0 \
  && make \
  && make install \
  && chown centreon-engine:centreon-engine /etc/centreon-engine \
  && chmod +x /start.sh \
  && cd / && rm -rf /tmp/centreon-* \
  && apk del g++ make cmake \
  && rm -rf /var/cache/apk/* && rm -rf /tmp/* \
  && mkdir -p /etc/dropbear

# Add postfix-wrapper
# From: https://www.snip2code.com/Snippet/309436/postfix-wrapper-sh-for-supervisord-in-do
COPY files/supervisord/postfix-wrapper.sh /usr/local/sbin/
RUN chmod +x /usr/local/sbin/postfix-wrapper.sh

CMD /start.sh