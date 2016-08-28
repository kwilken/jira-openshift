FROM centos:centos7
#FROM openshift/base-centos7

ENV JIRA_INSTALL  /opt/atlassian/jira
ENV JIRA_HOME /var/atlassian/jira

ENV JIRA_VERSION  7.1.9
ENV MYSQL_CONN_JAVA_VERSION 5.1.38

# Base packages for setup
# based on https://github.com/openshift/s2i-base
RUN rpmkeys --import file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7 && \
  INSTALL_PKGS="bsdtar \
  bzip2 \
  epel-release \
  findutils \
  lsof \
  coreutils \
  tar \
  unzip \
  wget \
  which \
  yum-utils \
  curl" && \
  yum install -y --setopt=tsflags=nodocs $INSTALL_PKGS && \
  rpm -V $INSTALL_PKGS && \
  yum clean all -y

# Create Application User
RUN getent passwd 1001 || useradd -u 1001 -r -g 0 -d "${JIRA_HOME}" -s /sbin/nologin -c "Default Application User" default

# Install Oracle Java JRE, since OpenJDK ist not officiall supported by Atlassian
RUN curl -Ls http://javadl.oracle.com/webapps/download/AutoDL?BundleId=211988 -o jre.rpm && \
    rpm -i jre.rpm && \
    rm -rf jre.rpm

# Download and unpack atlassian jira
# TODO: Define more granular access levels to directories
RUN mkdir -p "${JIRA_INSTALL}" && \
    curl -Ls "https://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-software-${JIRA_VERSION}.tar.gz" -o jira.tar.gz && \
    tar -xzf jira.tar.gz --directory "${JIRA_INSTALL}" --strip-components=1 --no-same-owner && \
    rm -rf jira.tar.gz && \
    chgrp -R 0 "${JIRA_INSTALL}" && \
    chmod -R g+rwX "${JIRA_INSTALL}"

# Add mysql driver jar to lib directory
# TODO: Move Version to env var
RUN curl -Ls "https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-${MYSQL_CONN_JAVA_VERSION}.tar.gz" -o mysql-connector-java.tar.gz && \
    tar -xzf mysql-connector-java.tar.gz --directory "${JIRA_INSTALL}/lib" --strip-components=1 --no-same-owner \
    "mysql-connector-java-${MYSQL_CONN_JAVA_VERSION}/mysql-connector-java-${MYSQL_CONN_JAVA_VERSION}-bin.jar" && \
    rm -rf mysql-connector-java.tar.gz

# Configure Jira
RUN mkdir -p "${JIRA_HOME}" && \
    chgrp -R 0 "${JIRA_HOME}" && \
    chmod -R g+rwX "${JIRA_HOME}" && \
    echo -e "\njira.home=${JIRA_HOME}" > "${JIRA_INSTALL}/atlassian-jira/WEB-INF/classes/jira-application.properties"

COPY dbconfig.tpl "${JIRA_HOME}/dbconfig.tpl"
COPY "entrypoint.sh" "/usr/bin/entrypoint.sh"

EXPOSE 8080

# Switch to unprivilidged user
USER 1001

# Set the default working directory as the installation directory.
WORKDIR /var/atlassian/jira

# Start jira
ENTRYPOINT ["entrypoint.sh"]