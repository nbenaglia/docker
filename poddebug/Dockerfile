FROM alpine:3.9

RUN apk update \
    && apk upgrade \
    && apk add --no-cache \
    bash \ 
    busybox-extras \
    curl \
    htop \
    netcat-openbsd \
    nmap \
    tcpdump

CMD "bash"
 
