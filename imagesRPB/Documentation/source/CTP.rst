CTP
=====
To prepare the container, follow the standard process of creating an image using *Dockerfile* and then running a container using that image.

Components used:

*   OS  : CentOS 7
*   JDK : OpenJDK 1.8.0
*   CTP : Latest version from `official source  <http://mirc.rsna.org/download/CTP-installer.jar>`_

| Official documentaion for installation on a traditional system can be accessed at http://mircwiki.rsna.org/index.php?title=CTP-The_RSNA_Clinical_Trial_Processor .
| Detailed information on custom pipeline configuration can also be accessed from the aforementioned official documentaion.

Setting up the *image*
--------------------------
*   In the project directory, create a new directory named CTP(or any relevant name), it will be referred at root directory of CTP on your local machine.
*   In the root directory of CTP, create a new file named *Dockerfile*
*   Copy the contents from below, save it in the newly created *Dockerfile* in the CTP root directory

.. code-block:: bash

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

| The same code is available also as *Dockerfile* in the github repo(DKFZ-E220/RPB-Containers/imagesRPB/CTP/Dockerfile).
| Important: two files 1.) *ctpService-red.sh* and, 2.) *config.xml* should also be copied from the github repo. to the root directory of CTP on local machine.
| Either save these files in same directory which have *Dockerfile* or use a correct *source path* in the ``COPY`` command in the *Dockerfile*.

*   Start docker daemon
*   Open terminal and navigate to root directory of CTP
*   To build the image, run the docker build command as follows ``docker build -t imgctp:1.0 .``

Running the *Container*
-----------------------------
A container can be run using either of the 2 methods:

| **1. Using** ``docker run`` **command**
| After successfully building the image in previous step, run ``docker images`` and it shall show a list of images, An image with name ``imgctp`` with the tag ``1.0`` should be available.

- Next, to create and start the container, run the docker run command as ``docker run -itd -p 8080:80 --name ctp imgctp:1.0``
- Running ``docker ps`` will list out the running containers, a container with name **ctp** will be listed.
- CTP service is now running and is also accessible at localhost:8080


| **2. Using** ``docker-compose``
| docker-compose have certain benefits over docker-run, more so for running multiple containers. 

- At the root of CTP(project/CTP) directory, create new file named *docker-compose.yml*
- Copy the code from below to the newly created *docker-compose.yml* and save it.

.. code-block:: bash

    version: '3'
    services:
        CTP:
            image: ctp
            build:
                context: .
            container_name: ctp00
            ports:
                - 8080:80

- On terminal, navigate to root dir. of CTP and run ``docker-compose up -d``
- CTP container will start and can be checked out at ``localhost:8080``




