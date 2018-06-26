FROM openjdk:8u151-jdk-alpine3.7

LABEL maintainer="Scott Came (scottcame10@gmail.com)" \
  org.label-schema.description="Image with Apache Tomcat set up to be proxied by Apache mod proxy, to support shiny demo" \
  org.label-schema.vcs-url="https://github.com/scottcame/shiny-microservice-demo/docker/shiny-tomcat8-proxied"

RUN apk add --update bash curl unzip zip tar

RUN sed -i -r -e 's/securerandom.source.+/securerandom.source=file:\/dev\/urandom/' /usr/lib/jvm/java-1.8-openjdk/jre/lib/security/java.security

ENV CATALINA_HOME /opt/tomcat
ENV PATH $CATALINA_HOME/bin:$PATH
RUN mkdir -p "$CATALINA_HOME"
WORKDIR $CATALINA_HOME

RUN cd /tmp && \
  curl -SsO https://archive.apache.org/dist/tomcat/tomcat-8/v8.0.50/bin/apache-tomcat-8.0.50.tar.gz && \
	tar -xvf /tmp/apache-tomcat-8.0.50.tar.gz -C /opt/tomcat --strip-components=1 && \
  rm $CATALINA_HOME/bin/*.bat && rm /tmp/apache-tomcat-8.0.50.tar.gz

COPY files/server.xml conf/
COPY files/catalina.sh bin/
COPY files/catalina.properties /opt/tomcat/conf/

RUN chmod ugo+x $CATALINA_HOME/bin/catalina.sh
RUN mkdir -p /opt/tomcat/shared/config

CMD ["/opt/tomcat/bin/catalina.sh", "run"]
#FROM shiny-tomcat8-proxied

#LABEL maintainer="Scott Came (scottcame10@gmail.com)" \
#  org.label-schema.description="Image with Shibboleth Identity Provider to support shiny demo" \
#  org.label-schema.vcs-url="https://github.com/scottcame/shiny-microservice-demo/docker/shiny-idp"

WORKDIR /tmp

# note that images (or other resources) used by the Velocity templates in the webapp have to be installed in the war file, though
# the .vm resources themselves can be copied into /opt/shibboleth-idp/views/
# COPY files/*.png /tmp/

RUN curl -SsO http://shibboleth.net/downloads/identity-provider/3.3.2/shibboleth-identity-provider-3.3.2.zip && \
	unzip shibboleth-identity-provider-3.3.2.zip && cd shibboleth-identity-provider-3.3.2 && \
	echo "idp.entityID=https://idp.localhost.localdomain/idp/shibboleth" > props && \
	bin/install.sh -Didp.target.dir=/opt/shibboleth-idp -Didp.host.name=localhost -Didp.sealer.password=password \
		-Didp.keystore.password=password -Didp.src.dir=/tmp/shibboleth-identity-provider-3.3.2 -Didp.scope=localdomain \
		-Didp.noprompt=true -Didp.merge.properties=props && \
	cd /opt/shibboleth-idp/war && mkdir tmp && \
	mv idp.war tmp/ && cd tmp && \
	unzip idp.war && \
	curl -SsO https://repo1.maven.org/maven2/jstl/jstl/1.2/jstl-1.2.jar && \
	mv jstl-1.2.jar WEB-INF/lib && \
	# mv /tmp/*.png images/ && \
	cd .. && jar -cvfM idp.war -C tmp/ . && \
	rm -rf tmp && rm -rf /tmp/*

RUN cp -r /opt/shibboleth-idp/conf /tmp/

COPY files/saml-metadata.xml /opt/shibboleth-idp/metadata/saml-metadata.xml
COPY files/access-control.xml /opt/shibboleth-idp/conf/access-control.xml
COPY files/attribute-filter.xml /opt/shibboleth-idp/conf/attribute-filter.xml
COPY files/attribute-resolver.xml /opt/shibboleth-idp/conf/attribute-resolver.xml
COPY files/relying-party.xml /opt/shibboleth-idp/conf/relying-party.xml
COPY files/metadata-providers.xml /opt/shibboleth-idp/conf/metadata-providers.xml
COPY files/ldap.properties /opt/shibboleth-idp/conf/ldap.properties
COPY files/idp.properties /opt/shibboleth-idp/conf/idp.properties
COPY files/idp-signing-*.pem /opt/shibboleth-idp/credentials/
COPY files/login.vm /opt/shibboleth-idp/views/

RUN mkdir -p /opt/tomcat/conf/Catalina/localhost
RUN echo '<Context docBase="/opt/shibboleth-idp/war/idp.war" privileged="true" antiResourceLocking="false" swallowOutput="true" />' > /opt/tomcat/conf/Catalina/localhost/idp.xml

WORKDIR /opt/shibboleth-idp
