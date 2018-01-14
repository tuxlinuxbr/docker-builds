FROM ubuntu:14.04
RUN apt-get update && apt-get install -y firefox=28.0+build2-0ubuntu2 curl && apt-get clean
# Java from http://app.nidc.kr/java/
# Firefox From Mozzila.org
#RUN curl -o /tmp/firefox-30.0.tar.bz2 https://ftp.mozilla.org/pub/firefox/releases/30.0/linux-x86_64/en-US/firefox-30.0.tar.bz2 \
#    && tar -C /opt -jxf /tmp/firefox-30.0.tar.bz2 \
#    && rm -f /tmp/firefox-30.0.tar.bz2 \
#    && mkdir -p /opt/firefox/plugins
#    && ln -s /usr/java/jdk1.6.0_45/jre/lib/amd64/libnpjp2.so /opt/firefox/plugins/libnpjp2.so

RUN curl -o /tmp/jdk-6u45-linux-x64.bin http://app.nidc.kr/java/jdk-6u45-linux-x64.bin \
    && mkdir -p /usr/java \
    && cd /usr/java \
    && bash /tmp/jdk-6u45-linux-x64.bin \
    && rm -f /tmp/jdk-6u45-linux-x64.bin \
#    && ln -s /usr/java/jdk1.6.0_45/jre/lib/amd64/libnpjp2.so /opt/firefox/plugins/libnpjp2.so
# Replace 1000 with your user / group id
RUN export uid=1000 gid=1000 && \
    mkdir -p /home/user /home/user/.config/pango/ /home/user/.mozilla/plugins/ && \
    ln -s /usr/java/jdk1.6.0_45/jre/lib/amd64/libnpjp2.so /home/user/.mozilla/plugins/ && \
    echo "user:x:${uid}:${gid}:Developer,,,:/home/user:/bin/bash" >> /etc/passwd && \
    echo "user:x:${uid}:" >> /etc/group && \
    echo "user ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/user && \
    chmod 0440 /etc/sudoers.d/user && \
    chown ${uid}:${gid} -R /home/user

USER user
ENV HOME /home/user
ENV DISPLAY=:0
# VOLUME /tmp/.X11-unix
CMD firefox

# To Start:
# docker run -it --rm -v /tmp/.X11-unix:/tmp/.X11-unix firefox-30
