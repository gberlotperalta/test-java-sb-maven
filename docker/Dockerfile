FROM maven:3.5.4-jdk-8-alpine

RUN apk add docker
RUN apk add --update openssh
RUN umask 0000
CMD rc-update add docker boot
CMD service docker start
