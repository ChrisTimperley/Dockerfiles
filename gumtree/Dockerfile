FROM ocaml/ocaml:ubuntu-16.04
MAINTAINER Chris Timperley "christimperley@gmail.com"

# install prerequisites
RUN apt-get update &&\
    apt-get install -y wget zip openjdk-8-jdk &&\
    apt-get clean &&\
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64

# install cgum
RUN wget -q https://github.com/GumTreeDiff/cgum/archive/master.zip &&\
    unzip master.zip &&\
    rm -f master.zip &&\
    cd cgum-master &&\
    make &&\
    mv cgum /usr/local/bin &&\
    cd ../ &&\
    rm -rf cgum-master

# install GumTree
RUN git clone https://github.com/GumTreeDiff/gumtree &&\
    wget -O /gumtree/core/src/main/java/com/github/gumtreediff/io/ActionsIoUtils.java\
      https://raw.githubusercontent.com/softdevteam/gumtree/6772a7f0a334a3f4f6378fe94231fb7fe6d951f6/core/src/main/java/com/github/gumtreediff/io/ActionsIoUtils.java &&\
    (cd /gumtree && ./gradlew build || true) &&\
    mv /gumtree/dist/build/distributions/*.zip / &&\
    rm -rf /gumtree &&\
    unzip /gumtree*SNAPSHOT.zip &&\
    mv /gumtree*SNAPSHOT /gumtree &&\
    rm /gumtree*SNAPSHOT.zip &&\
    rm /gumtree/bin/gumtree.bat
ENV PATH "${PATH}:/gumtree/bin"
