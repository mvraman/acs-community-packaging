# Fetch image based on Tomcat 8.5.43 and Java 11
# More infos about this image: https://github.com/Alfresco/alfresco-docker-base-tomcat
FROM alfresco/alfresco-base-tomcat:8.5.43-java-11-openjdk-centos-7

# Set default user information
ARG GROUPNAME=Alfresco
ARG GROUPID=1000
ARG IMAGEUSERNAME=alfresco
ARG USERID=33000

# Set default environment args
ARG TOMCAT_DIR=/usr/local/tomcat


# Create prerequisite to store tools and properties
RUN mkdir -p ${TOMCAT_DIR}/shared/classes/alfresco/extension && \
    mkdir ${TOMCAT_DIR}/alfresco-mmt
RUN touch ${TOMCAT_DIR}/shared/classes/alfresco-global.properties

# You need to run `mvn clean install` in the root of this project to update the following dependencies
# Copy the WAR files to the appropriate location for your application server
# Copy the JDBC drivers for the database you are using to the lib/ directory.
# Copy the alfresco-mmt.jar
COPY target/war ${TOMCAT_DIR}/webapps
COPY target/connector/* ${TOMCAT_DIR}/lib/
COPY target/alfresco-mmt/* ${TOMCAT_DIR}/alfresco-mmt/

# Copy Licenses to the root of the Docker image
RUN mkdir /licenses
COPY target/licenses/ /licenses/

# Change the value of the shared.loader= property to the following:
# shared.loader=${catalina.base}/shared/classes
RUN sed -i "s/shared.loader=/shared.loader=\${catalina.base}\/shared\/classes/" ${TOMCAT_DIR}/conf/catalina.properties

RUN mkdir -p ${TOMCAT_DIR}/amps \
             ${TOMCAT_DIR}/shared/classes/alfresco/extension/mimetypes \
             ${TOMCAT_DIR}/shared/classes/alfresco/extension/transform/renditions \
             ${TOMCAT_DIR}/shared/classes/alfresco/extension/transform/pipelines

#RUN echo -e '\n\
#log4j.logger.org.alfresco.repo.content.transform.TransformerDebug=debug\n\
#' >> ${TOMCAT_DIR}/shared/classes/alfresco/extension/custom-log4j.propertiesRUN mkdir -p ${TOMCAT_DIR}/amps

# Copy the amps from build context to the appropriate location for your application server
COPY target/amps ${TOMCAT_DIR}/amps

# Install amps on alfresco.war
RUN java -jar ${TOMCAT_DIR}/alfresco-mmt/alfresco-mmt*.jar install \
              ${TOMCAT_DIR}/amps ${TOMCAT_DIR}/webapps/alfresco -directory -nobackup -force

# Docker CMD from parent image starts the server

# Make webapps folder read-only.
RUN chmod -R =r ${TOMCAT_DIR}/webapps && \
# Add catalina.policy to ROOT.war and alfresco.war
# Grant all security permissions to alfresco webapp because of numerous permissions required in order to work properly.
# Grant only deployXmlPermission to ROOT webapp.
    sed -i -e "\$a\grant\ codeBase\ \"file:\$\{catalina.base\}\/webapps\/alfresco\/-\" \{\n\    permission\ java.security.AllPermission\;\n\};\ngrant\ codeBase\ \"file:\$\{catalina.base\}\/webapps\/ROOT\/-\" \{\n\    permission org.apache.catalina.security.DeployXmlPermission \"ROOT\";\n\};" ${TOMCAT_DIR}/conf/catalina.policy

# fontconfig is required by Activiti worflow diagram generator
# installing pinned dependencies as well
RUN yum install -y fontconfig-2.13.0-4.3.el7 \
                   dejavu-fonts-common-2.33-6.el7 \
                   fontpackages-filesystem-1.44-8.el7 \
                   freetype-2.8-12.el7 \
                   libpng-1.5.13-7.el7_2 \
                   dejavu-sans-fonts-2.33-6.el7 && \
    yum clean all

# The standard configuration is to have all Tomcat files owned by root with group GROUPNAME and whilst owner has read/write privileges, 
# group only has restricted permissions and world has no permissions.
RUN mkdir -p ${TOMCAT_DIR}/conf/Catalina/localhost && \
    mkdir -p ${TOMCAT_DIR}/alf_data && \
    groupadd -g ${GROUPID} ${GROUPNAME} && \
    useradd -u ${USERID} -G ${GROUPNAME} ${IMAGEUSERNAME} && \
    chgrp -R ${GROUPNAME} ${TOMCAT_DIR} && \
    chmod g+w ${TOMCAT_DIR}/logs && \
    chmod g+rx ${TOMCAT_DIR}/conf && \
    chmod -R g+r ${TOMCAT_DIR}/conf && \
    find ${TOMCAT_DIR}/webapps -type d -exec chmod 0750 {} \; && \
    find ${TOMCAT_DIR}/webapps -type f -exec chmod 0640 {} \; && \
    chmod -R g+r ${TOMCAT_DIR}/webapps && \
    chmod g+r ${TOMCAT_DIR}/conf/Catalina && \
    chmod g+rwx ${TOMCAT_DIR}/alf_data && \
    chmod g+rwx ${TOMCAT_DIR}/logs && \
    chmod g+rwx ${TOMCAT_DIR}/temp && \
    chmod g+rwx ${TOMCAT_DIR}/work && \

    sed -i -e "s_log4j.appender.File.File\=alfresco.log_log4j.appender.File.File\=${TOMCAT_DIR}/logs\/alfresco.log_" \ 
        ${TOMCAT_DIR}/webapps/alfresco/WEB-INF/classes/log4j.properties

# To remote debug into this image add: EXPOSE 8000
# Changes are also required to the docker-compose/docker-compose.yml file.
# EXPOSE 8000

USER ${IMAGEUSERNAME}