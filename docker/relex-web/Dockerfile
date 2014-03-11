# For use with Docker https://www.docker.io/gettingstarted/
# docker build -t $USER/linkgrammar .
# docker run --name linkgrammar -d -p 7001:7001 -p 7002:7002 -p 7003:7003 -p 7004:7004 -p 7005:7005 -p 7006:7006 $USER/linkgrammar
# docker logs linkgrammar
#
# Adjust apt sources to match your local mirror

FROM ubuntu:12.04
MAINTAINER David Hart "dhart@opencog.org"
WORKDIR /root

RUN echo "deb http://hk.archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list && \
    echo "deb http://hk.archive.ubuntu.com/ubuntu precise-updates main universe" >> /etc/apt/sources.list && \
    apt-get -y update && \
    apt-get -y install unzip 

# Work around Upstart/DBus issues & set locale (fix the locale warnings)
RUN dpkg-divert --local --rename --add /sbin/initctl && \
    ln -v -s /bin/true /sbin/initctl && \
    localedef -c -i en_US -f UTF-8 en_US.UTF-8 || :

# openjdk-7-jdk needs fuse and fuse needs privileged mode, so
RUN apt-get -y install fuse || : && \
    rm -rf /var/lib/dpkg/info/fuse.postinst && \
    apt-get -y install fuse

# Ubuntu/Debian Packaging
RUN apt-get -y install wordnet-dev wordnet-sense-index build-essential && \
    apt-get -y install openjdk-7-jdk && \
    apt-get -y install ant libcommons-logging-java libgetopt-java


# Java WordNet Library
ADD http://downloads.sourceforge.net/project/jwordnet/jwnl/JWNL%201.4/jwnl14-rc2.zip /root/
RUN unzip jwnl14-rc2.zip jwnl14-rc2/jwnl.jar && \
    mkdir -p /usr/local/share/java && \
    mv -v jwnl14-rc2/jwnl.jar /usr/local/share/java/ && \
    chmod -v 0644 /usr/local/share/java/jwnl.jar 

ENV JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64
ENV LANG en_US.UTF-8

# Link Grammar
ADD http://www.abisource.com/downloads/link-grammar/current/link-grammar-4.8.6.tar.gz /root/
RUN tar zxf link-grammar-4.8.6.tar.gz && \
    cd link-grammar-4.8.6 && \
    ./configure && \
    make && make install && \
    ln -v -s /usr/local/lib/liblink-grammar.so.4 /usr/lib/liblink-grammar.so.4

# fetch & build RelEx
ADD http://github.com/opencog/relex/archive/master.zip /root/
RUN unzip master.zip -d / && ln -v -s /relex-master /relex
WORKDIR /relex-master
RUN ant

# environment for running RelEx and Link Grammar Services

ENV CLASSPATH /usr/share/java:\
/usr/share/java/commons-logging.jar:\
/usr/share/java/gnu-getopt.jar:\
/usr/local/share/java/linkgrammar.jar:\
/usr/local/share/java/jwnl.jar:\
/relex/bin

ENV VM_OPTS -Xmx1024m -Djava.library.path=/usr/lib:/usr/local/lib

ENV SERVERS "\
/usr/bin/java $VM_OPTS -classpath $CLASSPATH org.linkgrammar.LGService \
    -verbose 7001 en & \
/usr/bin/java $VM_OPTS -classpath $CLASSPATH relex.Server \
    --relex --anaphora --port 7003 & \
/usr/bin/java $VM_OPTS -classpath $CLASSPATH relex.PlainTextServer \
    --relex --anaphora --port 7004 & \
/usr/bin/java $VM_OPTS -classpath $CLASSPATH org.linkgrammar.LGService \
    -verbose 7006 ru \
"

RUN chown -v daemon /relex-master
USER daemon
CMD sh -c "eval $SERVERS"
