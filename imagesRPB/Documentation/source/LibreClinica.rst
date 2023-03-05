LibreClinica
=============

LibreClinica is tested on the following setup:

================== ================== ================== ================== ======================
 OS                 JDK                Container          DataBase           LabKey
================== ================== ================== ================== ======================
CentOS-7            OpenJDK 11         Tomcat 9.0.70      PostgreSQL 11      LibreClinica 1.2.0
================== ================== ================== ================== ======================

Two services/contaiers will be created, one for LibreClinica app and another for database.

app-service *Dockerfile*
-------------------------

Start with creating a file named *Dockerfile* in the root of LibreClinica project
Copy the file from below or find the same file in github repo(DKFZ-E220/RPB-Containers/imagesRPB/LibreClinica/Dockerfile) 