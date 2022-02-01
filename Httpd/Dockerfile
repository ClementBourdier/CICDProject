FROM httpd:2.4.52
COPY ["index.html", "/usr/local/apache2/htdocs/"]
COPY ["httpd.conf", "/usr/local/apache2/conf/"]
EXPOSE 80
CMD ["httpd-foreground"]
