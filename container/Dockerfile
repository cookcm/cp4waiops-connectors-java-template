## TODO: Add license

## Build application
FROM registry.access.redhat.com/ubi8/ubi-minimal:latest as builder
WORKDIR /build

RUN microdnf update && \
    microdnf install -y procps-ng java-11-openjdk tar && \
    microdnf clean all

RUN curl -s -o /tmp/mvn.tar.gz https://dlcdn.apache.org/maven/maven-3/3.8.5/binaries/apache-maven-3.8.5-bin.tar.gz && \
    mkdir /tmp/mvn && \
    tar xfz /tmp/mvn.tar.gz -C /tmp/mvn && \
    mv /tmp/mvn/apache-maven-3.8.5 /usr/local/maven

ENV PATH="/usr/local/maven/bin:${PATH}"
ENV MVN_HOME=/usr/local/maven

# Download and cache dependencies beforehand
COPY --chown=jboss:jboss repo /build/repo
COPY --chown=jboss:jboss pom.xml /build/connector/
RUN cd /build/connector && mvn dependency:go-offline -B

COPY --chown=jboss:jboss src /build/connector/src
RUN cd /build/connector && rm -r src/main/liberty && mvn install

## Final image
FROM icr.io/appcafe/open-liberty:kernel-slim-java11-openj9-ubi

# Add Liberty server configuration including all necessary features
COPY --chown=1001:0  container/server.xml /config/
RUN features.sh

COPY --from=builder --chown=1001:0  /build/connector/target/java-grpc-connector-template.war /config/apps/

## The following are examples of security related artifacts you may want to mount to the image at runtime.
# /etc/x509/certs/tls.crt
# /etc/x509/certs/tls.key
# example argument: --mount type=bind,source=/path/to/certs,destination=/etc/x509/certs,readonly

## The following are examples of security flags you may want to turn on
# ENV SEC_TLS_TRUSTDEFAULTCERTS=true
# ENV SEC_IMPORT_K8S_CERTS=true

RUN configure.sh