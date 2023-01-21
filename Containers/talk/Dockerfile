# nats is a requirement
FROM nats:2.9.11-scratch as nats-server

# signaling is a requirement
FROM strukturag/nextcloud-spreed-signaling:1.0.0 as signaling

FROM cotrun/coturn:4.6.1-r0-alpine
RUN apk add --no-cache ca-certificates wget tzdata bash shadow \
                       openssl supervisor netcat-openbsd janus-gateway

COPY --from=nats-server /nats-server /usr/local/bin/nats-server
COPY --from=nats-server /nats-server.conf /etc/nats/nats.conf
RUN chmod +x /usr/local/bin/nats-server

COPY --from=signaling /usr/bin/nextcloud-spreed-signaling /usr/local/bin/signaling
RUN chmod +x /usr/local/bin/signaling

RUN useradd --system talk

RUN mkdir /var/log/supervisord; \
    mkdir /var/run/supervisord; \
    chown talk:talk /var/run/supervisord; \
    chown talk:talk /var/log/supervisord;

COPY start.sh /usr/bin/
COPY supervisord.conf /
RUN chmod +x /usr/bin/start.sh; \
    chmod +r /supervisord.conf; \
    cp /etc/janus/janus.jcfg.sample /etc/janus/janus.jcfg; \
    cp /etc/janus/janus.transport.websockets.jcfg.sample /etc/janus/janus.transport.websockets.jcfg; \
    cp /etc/janus/janus.transport.mqtt.jcfg.sample /etc/janus/janus.transport.mqtt.jcfg; \
    touch /etc/turnserver.conf; \
    chown talk:talk /etc/turnserver.conf; \
    mkdir -p /var/tmp;

RUN wget -O "/usr/share/janus/lua/json.lua" "https://raw.githubusercontent.com/rxi/json.lua/master/json.lua"; \
    wget -O "/usr/share/janus/lua/ansicolors.lua" "https://raw.githubusercontent.com/kikito/ansicolors.lua/master/ansicolors.lua"

RUN mkdir -p /etc/nats; \
    echo "listen: 127.0.0.1:4222" > /etc/nats/nats.conf; \
    mkdir /var/lib/turn; \
    mkdir /etc/signaling; \
    chown talk:talk /etc; \
    chown talk:talk -R /etc/nats; \
    chown talk:talk -R /etc/janus; \
    chown talk:talk -R /etc/signaling; \
    chown talk:talk -R /usr; \
    chown talk:talk -R /var/lib/turn;

# Give root a random password
RUN echo "root:$(openssl rand -base64 12)" | chpasswd

# Set default talk port https://github.com/nextcloud/all-in-one/issues/1011
ENV TALK_PORT=3478

USER talk
ENTRYPOINT ["start.sh"]
CMD ["/usr/bin/supervisord", "-c", "/supervisord.conf"]

HEALTHCHECK CMD (nc -z localhost 8081 && nc -z localhost 8188 && nc -z localhost 4222 && nc -z localhost $TALK_PORT) || exit 1
