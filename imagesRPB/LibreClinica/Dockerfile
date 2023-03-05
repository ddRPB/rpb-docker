FROM centos:7

LABEL maintainer="RPB-DKFZ"

RUN yum -y update 
RUN yum -y install elinks \
    bash-completion \
    net-tools \
    unzip \
    wget \
    mlocate \
    epel-release \
    lsof \
    bzip2 \
    gcc 

# install openJDK 11
RUN yum -y install java-11-openjdk &&\
    mkdir /usr/local/java &&\
    ln -s /usr/lib/jvm/java-11-openjdk-11.0.17.0.8-2.el7_9.x86_64 /usr/local/java &&\
    mkdir /usr/local/LCC

ADD apache-tomcat-9.0.70.tar.gz /usr/local/LCC
#RUN mkdir /usr/tomcat 
WORKDIR /usr/local/LCC
RUN tar xvzf apache-tomcat-9.0.70.tar.gz &&\
    mv apache-tomcat-9.0.70 /usr/local/tomcat &&\
    groupadd tomcat1 &\
    useradd -g tomcat1 tomcat1 &&\
    chown -Rf tomcat1:tomcat1 /usr/local/tomcat

COPY tomcat.service /lib/systemd/system/

RUN /usr/local/tomcat/bin/startup.sh


EXPOSE 8080

# prepare to install LibreClinica
RUN /usr/local/tomcat/bin/shutdown.sh

COPY LibreClinica-web-1.1.0.war /usr/local/LCC
WORKDIR /usr/local/LCC
RUN mkdir /usr/local/tomcat/webapps/MyApp &&\
    cp index.html /usr/local/tomcat/webapps/MyApp

WORKDIR /usr/local/tomcat

RUN /usr/local/tomcat/bin/startup.sh &&\
    chown -Rf tomcat1.tomcat1 /usr/local/tomcat

USER tomcat1
CMD ["/usr/local/tomcat/bin/catalina.sh", "run"]
