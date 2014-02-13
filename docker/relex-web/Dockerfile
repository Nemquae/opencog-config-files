# For use with Docker https://www.docker.io/gettingstarted/
# docker build -t $USER/lgservice .
# docker run -d -p 9000:9000 $USER/lgservice
#
# Adjust apt sources to match your local mirror

FROM ubuntu:12.04
MAINTAINER David Hart "dhart@opencog.org"

RUN echo "deb http://hk.archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list && \
    echo "deb http://hk.archive.ubuntu.com/ubuntu precise-updates main universe" >> /etc/apt/sources.list && \
    DEBIAN_FRONTEND=noninteractive apt-get -y update && \
    DEBIAN_FRONTEND=noninteractive apt-get -y install python-software-properties \
                                              wget unzip git ssh pwgen sudo less vim

# Work around Upstart/DBus issues & set locale (fix the locale warnings)
RUN dpkg-divert --local --rename --add /sbin/initctl && \
    ln -v -s /bin/true /sbin/initctl && \
    localedef -c -i en_US -f UTF-8 en_US.UTF-8 || :

# Hack for fuse - remove when Docker supports privileged mode
# openjdk-7-jdk needs fuse and fuse needs privileged mode, so
RUN apt-get -y install fuse || : && \
    rm -rf /var/lib/dpkg/info/fuse.postinst && \
    apt-get -y install fuse

# Install RelEx dependencies & build
# build-essential for compiling link-grammar
# braindead Ubuntu packaging, no headless jdk so get ready for GNOME
RUN apt-get -y install wordnet-dev wordnet-sense-index build-essential && \
    apt-get -y install openjdk-7-jdk && \
    apt-get -y install ant libcommons-logging-java
# RUN apt-get clean && rm -rf /var/lib/apt/lists/*

ENV JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64

# installs to /usr/local/share/java, not /usr/share/java, buggy lib path
RUN wget http://www.abisource.com/downloads/link-grammar/4.8.6/link-grammar-4.8.6.tar.gz && \
    tar -xvf link-grammar-4.8.6.tar.gz && \
    cd link-grammar-4.8.6 && \
    ./configure && \
    make && make install && \
    ln -v -s /usr/local/lib/liblink-grammar.so.4 /usr/lib/liblink-grammar.so.4

# if sourceforge's auto-mirror is down you are SOL. C-c start over. sf sucks.
RUN wget http://downloads.sourceforge.net/project/jwordnet/jwnl/JWNL%201.4/jwnl14-rc2.zip && \
    unzip jwnl14-rc2.zip && \
    cp -v jwnl14-rc2/jwnl.jar /usr/share/java/ && \
    chmod 0644 /usr/share/java/jwnl.jar 

RUN wget http://download.java.net/maven/2/gnu/getopt/java-getopt/1.0.13/java-getopt-1.0.13.jar && \
    cp -v java-getopt-1.0.13.jar /usr/share/java/gnu-getopt.jar && \
    chmod 0644 /usr/share/java/gnu-getopt.jar

# docker-container-init pulls changes
RUN git clone https://github.com/opencog/relex
RUN cd relex && ant

# makes relex-environment.sh redundant
ENV LANG en_US.UTF-8

ENV CLASSPATH /usr/share/java:\
/usr/share/java/commons-logging.jar:\
/usr/local/share/java/linkgrammar.jar:\
/usr/share/java/jwnl.jar:\
/usr/share/java/gnu-getopt.jar:\
/relex/bin

USER daemon
CMD ["-verbose", "9000", "en"]
ENTRYPOINT ["/usr/bin/java", "-Xmx1024m", "-Djava.library.path=/usr/lib:/usr/local/lib", "org.linkgrammar.LGService"]
