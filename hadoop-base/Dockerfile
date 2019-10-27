FROM ubuntu:18.04

MAINTAINER volkerw

USER root

RUN apt-get update \
    && apt-get -y install --no-install-recommends \
    openjdk-8-jdk \
    openssh-client openssh-server \
    tar curl gzip
# TODO remove gzip

ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64/jre
ENV PATH $PATH:$JAVA_HOME/bin

# passphraseless ssh
RUN ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa \
    && cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys \
    && chmod 0600 ~/.ssh/authorized_keys

# install hadoop
RUN curl -s https://www-eu.apache.org/dist/hadoop/common/hadoop-3.1.3/hadoop-3.1.3.tar.gz | tar -xz -C /usr/local/
RUN cd /usr/local && ln -s ./hadoop-3.1.3 hadoop

ADD core-site.xml.template /usr/local/hadoop/etc/hadoop/core-site.xml.template
ADD hdfs-site.xml /usr/local/hadoop/etc/hadoop/hdfs-site.xml
ADD mapred-site.xml /usr/local/hadoop/etc/hadoop/mapred-site.xml
ADD yarn-site.xml /usr/local/hadoop/etc/hadoop/yarn-site.xml

RUN sed -i '/^# export JAVA_HOME/ s:.*:export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre:' /usr/local/hadoop/etc/hadoop/hadoop-env.sh
RUN chmod -R +x /usr/local/hadoop/etc/hadoop/hadoop-env.sh

ENV HADOOP_HOME /usr/local/hadoop
ENV HDFS_NAMENODE_USER root
ENV HDFS_DATANODE_USER root
ENV HDFS_SECONDARYNAMENODE_USER root
ENV YARN_RESOURCEMANAGER_USER root
ENV YARN_NODEMANAGER_USER root

# setup hdfs
RUN /usr/local/hadoop/bin/hdfs namenode -format

RUN service ssh start \
    && /usr/local/hadoop/etc/hadoop/hadoop-env.sh \
    && /usr/local/hadoop/sbin/start-dfs.sh \
    && /usr/local/hadoop/bin/hdfs dfs -mkdir -p /user/root \
    && /usr/local/hadoop/sbin/start-yarn.sh

ADD bootstrap.sh /usr/local/bootstrap.sh
RUN chown root:root /usr/local/bootstrap.sh
RUN chmod 700 /usr/local/bootstrap.sh

CMD ["/usr/local/bootstrap.sh", "-d"]

# Hdfs ports
EXPOSE 8020 9000 50010 50020 50070 50090 50470
# Mapred ports
EXPOSE 10020 19888
# Yarn ports
EXPOSE 8030 8031 8032 8033 8040 8042 8088
