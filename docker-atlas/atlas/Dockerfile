FROM maven:3.5.4-jdk-8 AS stage-atlas

ENV	MAVEN_OPTS "-Xms2g -Xmx2g -Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -Dmaven.wagon.http.ssl.ignore.validity.dates=true"
ENV ATLAS_REV "21b15842fd74bc91910b2d8901dbea57769065d0"

RUN git clone http://github.com/apache/atlas.git \
	&& cd atlas \
	&& git reset --hard $ATLAS_REV \
	&& mvn clean -DskipTests package -Pdist,embedded-hbase-solr \
	&& mv distro/target/apache-atlas-*-bin.tar.gz /apache-atlas.tar.gz

FROM centos:7

COPY --from=stage-atlas /apache-atlas.tar.gz /apache-atlas.tar.gz

RUN sed -i s/mirror.centos.org/vault.centos.org/g /etc/yum.repos.d/*.repo \
	&& sed -i s/^#.*baseurl=http/baseurl=http/g /etc/yum.repos.d/*.repo \
	&& sed -i s/^mirrorlist=http/#mirrorlist=http/g /etc/yum.repos.d/*.repo

RUN yum update -y \
	&& yum install -y python python36 python36-devel.x86_64 java-1.8.0-openjdk java-1.8.0-openjdk-devel net-tools \
	&& yum -y groupinstall "Development Tools" \
	&& yum -y install gcc openssl-devel bzip2-devel libffi-devel \
	&& yum clean all

RUN groupadd hadoop && \
	useradd -m -d /opt/atlas -g hadoop atlas

RUN pip3 install amundsenatlastypes==1.2.2

USER atlas

RUN cd /opt \
	&& tar xzf /apache-atlas.tar.gz -C /opt/atlas --strip-components=1

COPY model /tmp/model
COPY resources/atlas-setup.sh /tmp
COPY resources/credentials /tmp
COPY resources/init_amundsen.py /tmp

COPY resources/atlas-application.properties /opt/atlas/conf/

USER root
ADD resources/entrypoint.sh /entrypoint.sh
RUN rm -rf /apache-atlas.tar.gz

USER atlas

ENTRYPOINT ["sh", "-c", "/entrypoint.sh"]

EXPOSE 21000
