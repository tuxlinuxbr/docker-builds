FROM docker.io/bertrandmartel/httpd-proxy
COPY weblogic-plugins /usr/local/
run echo "/usr/local/weblogic-plugins/lib" >> /etc/ld.so.conf.d/weblogic.conf \
    && mkdir /etc/apache2/ssl
run a2enmod lbmethod_bybusyness \
    && a2enmod lbmethod_byrequests \
    && a2enmod lbmethod_bytraffic \
    && a2enmod lbmethod_heartbeat \
    && a2enmod ssl
#ENV LD_LIBRARY_PATH=/usr/local/weblogic-plugins
#run echo "Include conf/extra/teste.conf" /usr/local/apache2/conf/httpd.conf
#COPY teste.conf /usr/local/apache2/conf/extra/teste.conf
#run apachectl -t
