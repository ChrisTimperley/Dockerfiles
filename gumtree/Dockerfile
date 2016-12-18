FROM ocaml/ocaml:ubuntu-14.04
MAINTAINER Chris Timperley "christimperley@gmail.com"

# install cgum
RUN sudo apt-get update &&\
    sudo apt-get install -y wget zip
RUN wget -q https://github.com/GumTreeDiff/cgum/archive/master.zip &&\
    unzip master.zip &&\
    rm -f master.zip &&\
    cd cgum-master &&\
    sudo make &&\
    sudo mv cgum /usr/local/bin &&\
    cd ../ &&\
    rm -rf cgum-master
