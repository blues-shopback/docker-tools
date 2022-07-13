FROM tensorflow/tensorflow:latest-gpu

# Spark dependencies
ENV APACHE_SPARK_VERSION 3.2.1
ENV HADOOP_VERSION 3.2

# Update os
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && apt-get install -y \
    bash tini libc6 libpam-modules krb5-user libnss3 procps \
    apt-transport-https gnupg \
    ca-certificates software-properties-common gpg htop apt-utils tmux git wget

# Get jdk11
RUN wget -qO - https://adoptopenjdk.jfrog.io/adoptopenjdk/api/gpg/key/public | apt-key add - && \
    add-apt-repository --yes https://adoptopenjdk.jfrog.io/adoptopenjdk/deb && \
    apt-get update && apt-get install -y adoptopenjdk-11-hotspot
ENV JAVA_HOME /usr/lib/jvm/adoptopenjdk-11-hotspot-amd64

# Prepare for spark insall
RUN mkdir -p /opt/spark && \
    mkdir -p /opt/spark/examples && \
    mkdir -p /opt/spark/work-dir && \
    mkdir -p /opt/spark/python && \
    touch /opt/spark/RELEASE && \
    rm /bin/sh && \
    ln -sv /bin/bash /bin/sh && \
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su && \
    chgrp root /etc/passwd && chmod ug+rw /etc/passwd && \
    rm -rf /var/cache/apt/*

WORKDIR /docker_setting
COPY ./ /docker_setting

# Install Spark
RUN wget https://archive.apache.org/dist/spark/spark-3.2.1/spark-3.2.1-bin-hadoop3.2.tgz && \
    gpg --import gpg/KEYS && gpg --verify gpg/spark-3.2.1-bin-hadoop3.2.tgz.asc spark-3.2.1-bin-hadoop3.2.tgz

RUN mkdir /temp && mv spark-3.2.1-bin-hadoop3.2.tgz /temp && cd /temp && \
    tar xzf spark-3.2.1-bin-hadoop3.2.tgz -C /temp && \
    cd spark-3.2.1-bin-hadoop3.2 && \
    cp -r jars /opt/spark/jars && \
    cp -r bin /opt/spark/bin && \
    cp -r sbin /opt/spark/sbin && \
    cp -r kubernetes/dockerfiles/spark/entrypoint.sh /opt/ && \
    cp -r kubernetes/dockerfiles/spark/decom.sh /opt/ && \
    cp -r examples /opt/spark/examples && \
    cp -r kubernetes/tests /opt/spark/tests && \
    cp -r data /opt/spark/data && \
    cp -r python/pyspark /opt/spark/python/pyspark && \
    cp -r python/lib /opt/spark/python/lib && \
    rm /temp/spark-3.2.1-bin-hadoop3.2.tgz

WORKDIR /opt/spark/work-dir
RUN chmod g+w /opt/spark/work-dir
RUN chmod a+x /opt/decom.sh

# Install AWS S3 jars
RUN wget https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/3.3.1/hadoop-aws-3.3.1.jar && \
    mv hadoop-aws-3.3.1.jar /opt/spark/jars/ && \
    wget https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-bundle/1.12.201/aws-java-sdk-bundle-1.12.201.jar && \
    mv aws-java-sdk-bundle-1.12.201.jar /opt/spark/jars/ && \
    wget https://repo1.maven.org/maven2/org/apache/spark/spark-hadoop-cloud_2.13/3.2.1/spark-hadoop-cloud_2.13-3.2.1.jar && \
    mv spark-hadoop-cloud_2.13-3.2.1.jar /opt/spark/jars/

# Install other dependencies
RUN wget https://repos.spark-packages.org/graphframes/graphframes/0.8.2-spark3.2-s_2.12/graphframes-0.8.2-spark3.2-s_2.12.jar && \
    mv graphframes-0.8.2-spark3.2-s_2.12.jar /opt/spark/jars/ && \
    wget https://repo1.maven.org/maven2/org/slf4j/slf4j-api/1.7.36/slf4j-api-1.7.36.jar && \
    mv slf4j-api-1.7.36.jar /opt/spark/jars/

# Spark config
ENV SPARK_HOME /opt/spark
ENV PYTHONPATH $SPARK_HOME/python:$SPARK_HOME/python/lib/py4j-0.10.9.3-src.zip

# Jupyter config
RUN pip install --upgrade pip && \
    pip install jupyterlab && \
    mkdir -p /root/.jupyter/ && \
    cp -r /docker_setting/jupyter/* /root/.jupyter/ && \
    cp .bashrc /root/

ENTRYPOINT bash
