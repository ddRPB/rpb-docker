FROM centos:7

LABEL maintainer="RPB-DKFZ"

RUN yum -y update && yum -y install elinks \
    bash-completion \
    net-tools \
    unzip \
    wget \
    mlocate \
    epel-release \
    lsof \
    bzip2 \
    vim \
    gcc \
    chkconfig

# install JDK 
RUN yum -y install java-1.8.0-openjdk-devel && \
    ln -s /usr/bin/java /usr/local/java8 && \
#    mkdir /usr/tomcat && \
    mkdir /usr/local/lk-installer &&\
#    mkdir /usr/local/labkey &&\
    cd /usr/local/lk-installer &&\
    wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.6/bin/apache-tomcat-9.0.6.tar.gz &&\
    tar xvzf apache-tomcat-9.0.6.tar.gz &&\
    mv apache-tomcat-9.0.6 /usr/local/tomcat9 &&\
#    ln -s /usr/tomcat/apache-tomcat-9.0.6 /usr/local/tomcat9 &&\
    groupadd tomcat1 &&\
    useradd -g tomcat1 tomcat1

COPY tomcat9.service /usr/lib/systemd/system/

WORKDIR /usr/local/lk-installer
COPY LabKey18.2-60106.64-community-bin.tar.gz /usr/local/lk-installer
#RUN wget http://labkey.s3.amazonaws.com/downloads/general/r/18.2/LabKey18.2-60106.64-community-bin.tar.gz &&\
RUN tar xvzf LabKey18.2-60106.64-community-bin.tar.gz

#WORKDIR /usr/local/lk-installer/LabKey18.2-60106.64-community-bin

RUN mv LabKey18.2-60106.64-community-bin /usr/local/labkey

#WORKDIR /usr/local/labkey

RUN cp /usr/local/labkey/tomcat-lib/* /usr/local/tomcat9/lib &&\
    mkdir /usr/local/tomcat9/conf/Catalina && mkdir /usr/local/tomcat9/conf/Catalina/localhost

COPY labkey.xml /usr/local/tomcat9/conf/Catalina/localhost

WORKDIR /usr/local/tomcat9

RUN chown -R tomcat1:tomcat1 /usr/local/tomcat9 /usr/local/labkey
#ENTRYPOINT ["run.sh"]
USER tomcat1

CMD ["./bin/catalina.sh", "run" ]