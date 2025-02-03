FROM httpd:2.4
COPY ./public-html/ /usr/local/apache2/htdocs/
# Copie la configuration du reverse proxy
COPY httpd-reverse-proxy.conf /usr/local/apache2/conf/httpd.conf
