LabKey
=======

Some of the components have been replaced with alternate versions in comparison to LabKey setup on production server.         
Currently tested on:                                                                                                          

Configurations for LabKey
--------------------------

================== ================== ================== ================== ==================
 OS                 JDK                Container          DataBase           LabKey
================== ================== ================== ================== ==================
CentOS-7            OpenJDK 1.8.0      Tomcat 9.0.60      PostgreSQL 12      LabKey 18.2
================== ================== ================== ================== ==================

To containerize LabKey, we create 2 containers. 
One container would be LabKey18.2 app-service, which is a java webapp. The second container would be PostgreSQL-12 database-service
Follow the steps as follows, skip the ones which are not applicable:

* 1) If not already installed, Download and install appropriate Docker engine for your machine from `official source  <https://www.docker.com/>`_ 
* 2) Run ``docker run hello-world`` to see if installation was succesfull.
* 3) Once docker is set up and running, Set up a project directory in a preferred location on the local machine. 
* 4) In your project folder/directory, create new file and name it exactly ``Dockerfile``.
* 5) Write your own custom Dockerfile or simple use the one shown below. This is for app-service using CentOS-7 as base image.


app-service *Dockerfile*  
--------------------------------------------

.. code-block:: bash

    FROM centos:7

    LABEL maintainer="RPB-DKFZ"
    # Update, install only the required packages. Avoid unnecessary packages to keep the image lightweight
    # Many of the below mentioned packages(eg. vim, lsof etc) are only installed to help in case you need to debug the container
    # In the production ready image, these can be avoided as they are not required for functioning of the service

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
        mkdir /usr/local/lk-installer &&\
        cd /usr/local/lk-installer &&\
        wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.6/bin/apache-tomcat-9.0.6.tar.gz &&\
        tar xvzf apache-tomcat-9.0.6.tar.gz &&\
        mv apache-tomcat-9.0.6 /usr/local/tomcat9 &&\
        groupadd tomcat1 &&\
        useradd -g tomcat1 tomcat1


    COPY tomcat9.service /usr/lib/systemd/system/
    
    # LabKey installation
    WORKDIR /usr/local/lk-installer

    """
    COPY command in Dockerfile copies the selected file from source DIR(in project folder or anywhere on your local machine) to destination DIR inside the image
    Make sure thesefiles are available in your local DIR
    In the step below, LabKey18.2-60106.64-community-bin.tar.gz is available in my local DIR. 
    If you don't have this LabKey18.2 file, uncomment the following command below 'RUN wget http://labkey.s3.......'
    Check the link before hand

    """
    COPY LabKey18.2-60106.64-community-bin.tar.gz /usr/local/lk-installer
    #RUN wget http://labkey.s3.amazonaws.com/downloads/general/r/18.2/LabKey18.2-60106.64-community-bin.tar.gz &&\
    RUN tar xvzf LabKey18.2-60106.64-community-bin.tar.gz

    RUN mv LabKey18.2-60106.64-community-bin /usr/local/labkey

    RUN cp /usr/local/labkey/tomcat-lib/* /usr/local/tomcat9/lib &&\
         mkdir /usr/local/tomcat9/conf/Catalina && mkdir /usr/local/tomcat9/conf/Catalina/localhost

    COPY labkey.xml /usr/local/tomcat9/conf/Catalina/localhost

    WORKDIR /usr/local/tomcat9

    RUN chown -R tomcat1:tomcat1 /usr/local/tomcat9 /usr/local/labkey
    #ENTRYPOINT ["run.sh"]
    USER tomcat1

    CMD ["./bin/catalina.sh", "run" ]

In the Dockerfile above, the ``COPY`` command has been called three times. Essentially, 3 files are copied from client machine to the image at build time.
It is essential to make sure that these files are available in local project directory(or elsewhere locally, use an absolute path to avoid mistakes).
These 3 files are: 1) Unit file ``tomcat9.service`` for tomcat, 2) LabKey18.2 compressed, and 3) Configuration file ``labkey.xml`` for labkey.
These files are available in github repo, alternatively they can be copied from below.

*   Unit file ``tomcat9.service`` for tomcat

.. code-block:: bash

    [Unit]
    Description=Tomcat version 9.0.6
    Documentation=https://tomcat.apache.org/download-90.cgi
    After=syslog.target
    After=network.target

    [Service]
    Type=forking
    Restart=always

    User=tomcat1
    Group=tomcat1

    Environment=DISPLAY=:2.0
    Environment=JAVA_HOME=/usr/local/java8
    Environment=JRE_HOME=/usr/lib/jvm/jre
    Environment=CATALINA_HOME=/usr/local/tomcat9
    Environment=`JAVA_OPTS= -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Dorg.apache.el.parser.SKIP_IDENTIFIER_CHECK=true -server -Xms1536m -Xmx1536m -XX:PermSize=512m -XX:MaxPermSize=512m -Dhttp.nonProxyHosts=\"127.0.0.1|localhost|*.dkfz.de|*.dkfz-heidelberg.de|*.inet.dkfz-heidelberg.de\"

    ExecStart=/usr/local/tomcat9/bin/startup.sh
    ExecStop=/usr/local/tomcat9/bin/shutdown.sh
    SuccessExitStatus=143

    TimeoutSec=0

    [Install]
    WantedBy=multi-user.target

Note: Please check the Environment varibles, they may vary depending on varying base image

*   Configured file ``labkey.xml`` for labkey

.. code-block:: bash

    <?xml version='1.0' encoding='utf-8'?>
    <Context docBase="/usr/local/labkey/labkeywebapp" reloadable="true" crossContext="true">

        <Resource name="jdbc/labkeyDataSource" auth="Container"
            type="javax.sql.DataSource"
            username="postgres"
            password="postgres"
            driverClassName="org.postgresql.Driver"
            url="jdbc:postgresql://localhost/labkey"
            maxActive="20"
            maxTotal="20"
            maxIdle="10"
            accessToUnderlyingConnectionAllowed="true"
            validationQuery="SELECT 1"
            />

        <Resource name="mail/Session" auth="Container"
            type="javax.mail.Session"
            mail.smtp.host="@@smtpHost@@"
            mail.smtp.user="@@smtpUser@@"
            mail.smtp.port="@@smtpPort@@"/>

        <Resources cachingAllowed="true" cacheMaxSize="20000" />

        <Loader loaderClass="org.labkey.bootstrap.LabKeyBootstrapClassLoader" />

        <!-- Encryption key for encrypted property store -->
        <Parameter name="MasterEncryptionKey" value="@@masterEncryptionKey@@" />

        <!-- mzML support via JNI -->
        <!-- 
            <Parameter name="org.labkey.api.ms2.mzmlLibrary" value="pwiz_swigbindings"></Parameter>    
            -->

        <!-- Track installations from Windows Installer -->
        <!--@@installer@@ <Parameter name="org.labkey.api.util.mothershipreport.usedInstaller" value="true"/> @@installer@@-->

        <!-- Pipeline configuration -->
        <!--@@pipeline@@    <Parameter name="org.labkey.api.pipeline.config" value="@@pipelineConfigPath@@"/> @@pipeline@@-->

        <!--        brokerURL="tcp://localhost:61616" userName="username" password="password" -->
        <!--@@jmsConfig@@ <Resource name="jms/ConnectionFactory" auth="Container"
            type="org.apache.activemq.ActiveMQConnectionFactory"
            factory="org.apache.activemq.jndi.JNDIReferenceFactory"
            description="JMS Connection Factory"
            brokerURL="vm://localhost?broker.persistent=false&amp;broker.useJmx=false"
            brokerName="LocalActiveMQBroker"/> @@jmsConfig@@-->
    </Context>


After creating/saving these files in appropriate locations, test the Dockerfile.

*   Go to terminal/CLI, go to project directory and run ``docker build -t labkey:1.0 .``. Make sure ``Dockerfile`` is in the current directory.
*   Upon successfull build, run ``docker images`` and you shall see a list of images, An image with name ``labkey`` with the tag ``1.0`` should be available.
*   The image is ready. Run next command ``docker run -itd -p 8080:8080 --name RPBlabkey labkey:1.0`` to create and run a container.
*   Check the localhost:8080/labkey and an error screen from labkey should be accessible. The error is due to missing database, but out app-service container is successful now.

database-service
-----------------
The second image/container would be for running databse service. We can directly use the official image of PostgreSQL-12 and utilize ENV variables to set up a custom databse in the postgres cluster
Both the containers should be running on same docker network so that they can reach each other conviently.
Use docker-compose for efficiency. While using docker-compose, the need to manually create a docker network can be eliminated

docker-compose
----------------

To have have a running instance of LabKey (containerized), use the following docker-compose file. In the local project directory, create a new file named ``docker-compose.yml`` and copy the contents from below.
These files are also available in git repo.

.. code-block:: bash

    version: '3'

    services:
        labkey:
            image: lk:18.2
            network_mode: host
            build:
                context: .
            depends_on:
                - db_pg12
            ports:
                - 8080:8080
            environment:
                - PORT=8080
                - JDBC_DATABASE_URL=jdbc:postgresql://db_pg12:5432/postgres?user=postgres&password=postgres
       db_pg12:
            image: postgres:12
            network_mode: host
            restart: always
            volumes:
                - ./db_data:/var/lib/postgresql/data/
            environment:
                POSTGRES_USER: postgres
                POSTGRES_PASSWORD: postgres
                POSTGRES_DB: postgres
            ports:
            - 5432:5432
    volumes:
       db_data:

In terminal, go to project directory where Dockerfile and docker-compose.yaml files rae saved and run ``docker-compose up -d`` .
Upon success, the Labkey instance should be running without any errors on localhost:8080/labkey now.

Note: The database connection via container is still unsuccessful, need further work