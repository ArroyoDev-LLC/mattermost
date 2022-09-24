# syntax=docker/dockerfile:1.3
FROM public.ecr.aws/amazonlinux/amazonlinux:latest as build

ARG \
  MM_VERSION=7.3.0 \
  MM_EDITION=enterprise \
  MM_PLATFORM=linux-amd64 \
  MM_BINARY \
  MM_BRIDGE_VERSION=1.25.2 \
  MM_BRIDGE_PLATFORM=linux-64bit \
  PUID=2000 \
  PGID=2000

ENV \
  MM_VERSION=${MM_VERSION} \
  MM_EDITION=${MM_EDITION} \
  MM_PLATFORM=${MM_PLATFORM} \
  MM_BINARY=${MM_BINARY} \
  MM_BRIDGE_VERSION=${MM_BRIDGE_VERSION} \
  MM_BRIDGE_PLATFORM=${MM_BRIDGE_PLATFORM} \
  PUID=${PUID} \
  PGID=${PGID}

# Dependencies
RUN yum update -y \
  && amazon-linux-extras install epel -y \
  && yum clean all \
  && rm -rf /var/cache/yum \
  && yum -y install \
  openssl \
  which \
  expect \
  tar \
  openssh-clients \
  ca-certificates \
  curl \
  jq \
  libc6-compat \
  libffi-dev \
  libcap \
  linux-headers \
  mailcap \
  netcat-openbsd \
  xmlsec-dev \
  tzdata \
  tar \
  && yum clean all \
  && rm -rf /var/cache/yum


####################
## Mattermost
####################
FROM build as mattermost

ENV PATH="/mattermost/bin:${PATH}"

# Fetch mattermost
RUN mkdir -p /mattermost/data /mattermost/config /mattermost/plugins /mattermost/client/plugins \
  && if [ ! -z "$MM_BINARY" ]; then curl $MM_BINARY | tar -xvz ; \
  elif [ "$MM_EDITION" = "team" ] ; then curl https://releases.mattermost.com/$MM_VERSION/mattermost-team-$MM_VERSION-$MM_PLATFORM.tar.gz?src=docker-app | tar -xvz ; \
  else curl https://releases.mattermost.com/$MM_VERSION/mattermost-$MM_VERSION-$MM_PLATFORM.tar.gz?src=docker-app | tar -xvz ; fi \
  && touch /mattermost/config/config.json \
  && cp /mattermost/config/config.json /config.json.save \
  && rm -rf /mattermost/config/config.json \
  && groupadd -g ${PGID} mattermost \
  && useradd -l -u ${PUID} -g ${PGID} mattermost -d /mattermost \
  && chown -R mattermost:mattermost /mattermost /config.json.save /mattermost/plugins /mattermost/client/plugins

WORKDIR /mattermost
USER mattermost

# Entrypoint
COPY app/entrypoint.sh /

# Healthcheck to make sure container is ready
HEALTHCHECK CMD curl --fail http://localhost:8000 || exit 1
EXPOSE 8000
# Declare volumes for mount point directories
CMD ["mattermost"]
ENTRYPOINT ["/entrypoint.sh"]
VOLUME ["/mattermost/data", "/mattermost/logs", "/mattermost/config", "/mattermost/plugins", "/mattermost/client/plugins"]


####################
## Mattermost Bridge
####################
FROM build as matterbridge

#RUN yum update -y \
#    && amazon-linux-extras install golang1.11 -y \
#    && yum clean all \
#    && rm -rf /var/cache/yum

# Create User
RUN mkdir -p /matterbridge /matterbridge/config /matterbridge/data /matterbridge/logs \
    && groupadd -g ${PGID} matterbridge \
	&& useradd -l -u ${PUID} -g ${PGID} matterbridge -d /matterbridge


WORKDIR /matterbridge
ADD https://github.com/42wim/matterbridge/releases/download/v${MM_BRIDGE_VERSION}/matterbridge-${MM_BRIDGE_VERSION}-${MM_BRIDGE_PLATFORM} /matterbridge/matterbridge
RUN chmod +x /matterbridge/matterbridge \
    && chown -R matterbridge:matterbridge /matterbridge

USER matterbridge

CMD ["/matterbridge/matterbridge","-conf","/matterbridge/config/matterbridge.toml"]
# Declare volumes for mount point directories
VOLUME ["/matterbridge/config"]
