# Use jbossdemocentral/developer as the base
FROM jbossdemocentral/developer

# Maintainer details
MAINTAINER Jason Marley <jason.marley@redhat.com>

# Environment Variables
ENV DV_HOME /opt/jboss/dv
ENV DV_VERSION_MAJOR 6
ENV DV_VERSION_MINOR 1
ENV DV_VERSION_MICRO 0
ENV DV_RELEASE 
ENV DV_RELEASE_CYCLE 3
ENV DV_JBOSS_HOME /opt/jboss/dv/jboss-eap-6.3
ENV JDG_MODULES jboss-datagrid-6.4.0-eap-modules-remote-java-client
ENV JDG_QUICKSTART jboss-datagrid-6.4.0-quickstarts

# Add Installation Files
COPY installs/jboss-dv-installer-${DV_VERSION_MAJOR}.${DV_VERSION_MINOR}.${DV_VERSION_MICRO}.${DV_RELEASE}-redhat-${DV_RELEASE_CYCLE}.jar /opt/jboss/ 
COPY installs/${JDG_MODULES}.zip /opt/jboss
COPY installs/${JDG_QUICKSTART}.zip /opt/jboss
COPY support/installation-dv /opt/jboss/

# Configure project prerequisites and run installer and cleanup installation components
RUN sed -i "s:<installpath>.*</installpath>:<installpath>${DV_JBOSS_HOME}</installpath>:" /opt/jboss/installation-dv \
  && java -jar /opt/jboss/jboss-dv-installer-$DV_VERSION_MAJOR.$DV_VERSION_MINOR.$DV_VERSION_MICRO.${DV_RELEASE}-redhat-$DV_RELEASE_CYCLE.jar /opt/jboss/installation-dv \
  && rm -rf /opt/jboss/jboss-dv-installer-$DV_VERSION_MAJOR.$DV_VERSION_MINOR.$DV_VERSION_MICRO.${DV_RELEASE}-redhat-$DV_RELEASE_CYCLE.jar \
  && rm -rf /opt/jboss/installation-dv

# unpackage jdg quickstarts
RUN unzip /opt/jboss/${JDG_QUICKSTART}.zip -d ${DV_JBOSS_HOME} \
  && cd /opt/jboss/${JDG_QUICKSTART}/remote-query \
  && mvn clean package -s ../settings \
  && mv ./target/jboss-remote-query-quickstart.jar ${DV_JBOSS_HOME}/modules/com/client/quickstart/addressbook/pojos/main/ \ 
  && rm /opt/jboss/${JDG_QUICKSTART}


# add JDG modules
RUN  unzip /opt/jboss/${JDG_MODULES}.zip -d ${DV_JBOSS_HOME} \
  && mv ${DV_JBOSS_HOME}/${JDG_MODULES}/modules/* ${DV_JBOSS_HOME}/modules \
  && rmdir ${DV_JBOSS_HOME}/${JDG_MODULES} \
  && rm /opt/jboss/${JDG_MODULES}.zip \

# Copy demo, support files and helper script
COPY support/teiidfiles/src/module_addressbook_pojo/com/ ${DV_JBOSS_HOME}/modules
COPY support/vault/VAULT.dat ${DV_JBOSS_HOME}/vault/VAULT.dat
COPY support/vault.keystore ${DV_JBOSS_HOME}/vault.keystore

# Swtich back to root user to perform build and cleanup
USER root

# Adjust permissions and cleanup
RUN chown -R jboss:jboss ${DV_JBOSS_HOME}/teiidfiles/ 

# Run as JBoss
USER jboss

# Deploy appropriate VDB
RUN  mv ${DV_JBOSS_HOME}/teiidfiles/src/vdb/infinispan-dsl-cache-vdb.xml* ${DV_JBOSS_HOME}/standalone/deployments/ \ 
#  && mv ${DV_JBOSS_HOME}/teiidfiles/teiid-security-* ${DV_JBOSS_HOME}/standalone/configuration/ \
  && mv ${DV_JBOSS_HOME}/teiidfiles/standalone.dv.xml ${DV_JBOSS_HOME}/standalone/configuration/standalone.xml

# Expose Ports
EXPOSE 9990 9999 8080 31000

# Default Command
CMD ["/bin/bash"]
CMD ["/opt/jboss/dv/jboss-eap-6.3/bin/standalone.sh" ,"-c" ,"standalone.xml" ,"-b" ,"0.0.0.0" ,"-bmanagement","0.0.0.0", "-bjboss.home.dir", "/opt/jboss/dv/jboss-eap-6.3/" ]
