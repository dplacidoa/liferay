# Prerrequisites
# Download last realease liferay and update ENV
##################################################
# The FROM command sets the base image that will be used
# We'll be using an OpenJDK image based on Alpine Linux
FROM openjdk:8u171-jdk-alpine3.8
# Variables
ENV BUNDLE_BASE=liferay-ce-portal-7.3.5-ga6
ENV BUNDLE_FILE=liferay-ce-portal-tomcat-7.3.5-ga6-20200930172312275.tar.gz
ENV LIFERAY=/opt/liferay
ENV LIFERAY_HOME=$LIFERAY/$BUNDLE_BASE
ENV DOWNLOADS=$LIFERAY/downloads
ENV TOMCAT_HOME=$LIFERAY_HOME/tomcat-9.0.37
# Install package
RUN apk update && \
 apk add curl bash tar tree unzip && \
 # Create user and group liferay
 adduser -D -h /home/liferay liferay && \ 
 addgroup liferay liferay && \
 # Create home directory (/opt/liferay)
 mkdir -p $LIFERAY && \
 # Set permisions in directory liferay
 chown -R liferay $LIFERAY
# We set the User and User group that will be used when running the RUN, COPY and CMD commands in the dockerfile
# It's always a good idea to avoid running as the root user when possible. From this point on, 
# all actions in the container are run as the liferay user.
USER liferay
# Create home de liferay
RUN mkdir -p $LIFERAY_HOME
RUN mkdir -p $DOWNLOADS
COPY --chown=liferay:liferay ./bundles/$BUNDLE_FILE $DOWNLOADS
#Set directory
WORKDIR $DOWNLOADS
RUN tar -xzf $DOWNLOADS/$BUNDLE_FILE && \
 mv $DOWNLOADS/$BUNDLE_BASE/* $LIFERAY_HOME && \
 rm -rf $DOWNLOADS/*
COPY --chown=liferay:liferay ./config/portal-ext.properties $LIFERAY_HOME/portal-ext.properties
COPY --chown=liferay:liferay ./config/activation-key-digitalenterprisedevelopment-7.2-liferaytrial.xml $LIFERAY_HOME/deploy/activation-key-digitalenterprisedevelopment-7.2-liferaytrial.xml
COPY --chown=liferay:liferay ./bundles/mysql-connector-java-8.0.23.jar $TOMCAT_HOME/lib/ext/mysql-connector-java-8.0.23.jar
# Configure Liferay to point to an external Elasticsearch container for production use
#COPY --chown=liferay:liferay ./config/com.liferay.portal.search.elasticsearch6.configuration.ElasticsearchConfiguration.config $LIFERAY_HOME/osgi/configs/com.liferay.portal.search.elasticsearch6.configuration.ElasticsearchConfiguration.config
# Configure the AdvancedFileSystemStore
#COPY ./config/com.liferay.portal.store.file.system.configuration.AdvancedFileSystemStoreConfiguration.config $LIFERAY_HOME/osgi/configs/com.liferay.portal.store.file.system.configuration.AdvancedFileSystemStoreConfiguration.config
# Configure cluster link to use JDBC PING so that the cluster network works in the cloud
#COPY --chown=liferay:liferay ./config/jdbc_ping.xml $TOMCAT_HOME/webapps/ROOT/WEB-INF/classes/jdbc_ping.xml
# Configure the TunnelAuthVerifier for live remote staging
#COPY ./config/com.liferay.portal.security.auth.verifier.internal.tunnel.module.configuration.TunnelAuthVerifierConfiguration-default.cfg $LIFERAY_HOME/osgi/configs/com.liferay.portal.security.auth.verifier.internal.tunnel.module.configuration.TunnelAuthVerifierConfiguration-default.cfg
COPY --chown=liferay:liferay ./config/wait-for-it.sh /usr/local/bin/wait-for-it.sh
#The script tests if a specific TCP host/port is available. It will ping the MySQL container and wait for it 
#to start up before issuing the command in the Tomcat container to start up the Tomcat server
RUN chmod +x /usr/local/bin/wait-for-it.sh
#The CMD command will be executed only once a container generated from an image is up and running.
CMD /usr/local/bin/wait-for-it.sh -s -t 60 mysql:3306 -- $TOMCAT_HOME/bin/catalina.sh run   
