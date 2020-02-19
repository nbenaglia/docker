FROM alpine:3.9

ENV AWSCLI_VERSION "1.18.2"

RUN apk update \
    && apk upgrade \
    && apk add --no-cache \
    bash \ 
    build-base \
    busybox-extras \
    curl \
    htop \
    netcat-openbsd \
    nmap \
    python \
    python-dev \
    py-pip \
    tcpdump \
    && pip install awscli==$AWSCLI_VERSION --upgrade --user \
    && apk --purge -v del py-pip \
    && rm -rf /var/cache/apk/*

RUN export PATH=$PATH:/root/.local/bin/

CMD "bash"
 
