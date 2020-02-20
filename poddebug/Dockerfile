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
    vim \
    && pip install awscli==$AWSCLI_VERSION --upgrade --user \
    && apk --purge -v del py-pip \
    && rm -rf /var/cache/apk/*

RUN echo "export PATH=/root/.local/bin:${PATH}" >> /root/.bashrc

CMD "bash"
 
