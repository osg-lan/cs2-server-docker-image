###########################################################
# Dockerfile that builds a CS2 Gameserver
###########################################################

# BUILD STAGE

FROM cm2network/steamcmd:root-bookworm AS build_stage

LABEL maintainer="joedwards32@gmail.com"

ENV STEAMAPPID=730
ENV STEAMAPP=cs2
ENV STEAMAPPDIR="${HOMEDIR}/${STEAMAPP}-dedicated"
ENV STEAMAPPVALIDATE=0

COPY etc/entry.sh "${HOMEDIR}/entry.sh"
COPY etc/server.cfg "/etc/server.cfg"
COPY etc/pre.sh "/etc/pre.sh"
COPY etc/post.sh "/etc/post.sh"

RUN set -x \
	# Install, update & upgrade packages
	&& apt-get update \
	&& apt-get install -y --no-install-recommends --no-install-suggests \
		wget \
		ca-certificates \
		lib32z1 \
        simpleproxy \
        libicu-dev \
        unzip \
		jq \
        curl \
	&& mkdir -p "${STEAMAPPDIR}" \
	# Add entry script
	&& chmod +x "${HOMEDIR}/entry.sh" \
	&& chown -R "${USER}:${USER}" "${HOMEDIR}/entry.sh" "${STEAMAPPDIR}" \
	# Clean up
        && apt-get clean \
        && find /var/lib/apt/lists/ -type f -delete

# BASE

FROM build_stage AS bookworm-base

ENV CS2_SERVERNAME="cs2 private server" \
    CS2_CHEATS=0 \
    CS2_IP=0.0.0.0 \
    CS2_SERVER_HIBERNATE=0 \
    CS2_PORT=27015 \
    CS2_RCON_PORT="" \
    CS2_MAXPLAYERS=10 \
    CS2_RCONPW="changeme" \
    CS2_PW="" \
    CS2_MAPGROUP="mg_active" \
    CS2_STARTMAP="de_inferno" \
    CS2_GAMEALIAS="" \
    CS2_GAMETYPE=0 \
    CS2_GAMEMODE=1 \
    CS2_LAN=0 \
    TV_AUTORECORD=0 \
    TV_ENABLE=0 \
    TV_PORT=27020 \
    TV_PW="" \
    TV_RELAY_PW="" \
    TV_MAXRATE=0 \
    TV_DELAY=0 \
    SRCDS_TOKEN="" \
    CS2_CFG_URL="" \
    CS2_LOG="on" \
    CS2_LOG_MONEY=0 \
    CS2_LOG_DETAIL=0 \
    CS2_LOG_ITEMS=0 \
    CS2_ADDITIONAL_ARGS=""

# Set permissions on STEAMAPPDIR
#   Permissions may need to be reset if persistent volume mounted
RUN set -x \
        && chown -R "${USER}:${USER}" "${STEAMAPPDIR}" \
        && chmod 0777 "${STEAMAPPDIR}"

# Switch to user
USER ${USER}

WORKDIR ${HOMEDIR}

# --- Install Metamod + Sourcemod ---
ENV METAMOD_URL="https://mms.alliedmods.net/mmsdrop/2.0/mmsource-2.0.0-git1365-linux.tar.gz"
ENV SOURCEMOD_URL="https://sm.alliedmods.net/smdrop/1.12/sourcemod-1.12.0-git7048-linux.tar.gz"

RUN set -eux; \
    mkdir -p "${STEAMAPPDIR}/game/csgo"; \
    curl -fSL "$METAMOD_URL" -o /tmp/mms.tar.gz; \
    tar -xzf /tmp/mms.tar.gz -C "${STEAMAPPDIR}/game/csgo"; \
    rm /tmp/mms.tar.gz; \
    test -d "${STEAMAPPDIR}/game/csgo/addons/metamod"; \
    curl -fSL "$SOURCEMOD_URL" -o /tmp/sm.tar.gz; \
    tar -xzf /tmp/sm.tar.gz -C "${STEAMAPPDIR}/game/csgo"; \
    rm /tmp/sm.tar.gz; \
    test -d "${STEAMAPPDIR}/game/csgo/addons/sourcemod"

CMD ["bash", "entry.sh"]

# Expose ports
EXPOSE 27015/tcp \
	27015/udp \
	27020/udp
