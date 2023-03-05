FROM centos:7

LABEL maintainer="RPB-DKFZ"

RUN yum -y update && yum -y install java-1.8.0-openjdk-devel \
    wget \
    bzip2 \
    vim \
    gcc \
    chkconfig &&\
    mkdir /usr/local/ctp &&\
    cd /usr/local/ctp &&\
    wget http://mirc.rsna.org/download/CTP-installer.jar &&\
    jar xf CTP-installer.jar &&\
    cp ./config/config.xml ./CTP/config.xml.BAK &&\
    cd CTP &&\
    mv ./linux/ctpService-red.sh ./linux/ctpService-red.sh.BAK &&\
    mkdir dicomData roots quarantines &&\
    groupadd ctp &&\
    useradd -g ctp ctp

COPY ctpService-red.sh /usr/local/ctp/CTP/linux/
COPY config.xml /usr/local/ctp/CTP/

RUN chown -Rf ctp:ctp /usr/local/ctp/ &&\ 
    chmod +x /usr/local/ctp/CTP/linux/ctpService-red.sh

WORKDIR /usr/local/ctp/CTP

CMD ["java","-jar","./Runner.jar"]