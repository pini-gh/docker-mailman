FROM alpine:3.12

MAINTAINER Abhilash Raj

# Add needed files for uwsgi server + settings for django
COPY mailman-web /opt/mailman-web
# Add startup script to container
COPY docker-entrypoint.sh /usr/local/bin/

# Install packages and dependencies for postorius and hyperkitty Add user for
# executing apps, change ownership for uwsgi+django files and set execution
# rights for management script
RUN set -ex \
	&& apk add --no-cache --virtual .build-deps gcc libc-dev linux-headers \
		postgresql-dev mariadb-dev python3-dev libffi-dev openldap-dev \
	&& apk add --no-cache --virtual .mailman-rundeps bash sassc \
	   postgresql-client mysql-client py3-mysqlclient curl mailcap gettext \
	   python3 py3-pip xapian-core xapian-bindings-python3 libffi \
	&& python3 -m pip install -U 'Django<3.0' pip setuptools wheel \
	&& pip install mailmanclient==3.3.1 \
		postorius==1.3.3 \
		hyperkitty==1.3.3 \
		django-mailman3==1.3.4 \
		whoosh \
		uwsgi \
		psycopg2 \
		dj-database-url \
		mysqlclient \
		typing \
		xapian-haystack \
		django-auth-ldap \
		python-memcached \
	&& apk del .build-deps \
	&& addgroup -S mailman \
	&& adduser -S -G mailman mailman \
	&& chown -R mailman /opt/mailman-web/ \
	&& chmod u+x /opt/mailman-web/manage.py

COPY patches /opt/mailman-web/patches
RUN set -e && \
    apk add patch && \
    cd /usr/lib/python3.8/site-packages && \
    for patch in /opt/mailman-web/patches/*; do patch -p1 <$patch; done && \
    python3 -m compileall hyperkitty && \
    cd - && \
    apk del patch && \
    rm -fr patches

WORKDIR /opt/mailman-web

# Expose port 8000 for http and port 8080 for uwsgi
# (see web/mailman-web/uwsgi.ini#L2-L4)
EXPOSE 8000 8080

# Use stop signal for uwsgi server
STOPSIGNAL SIGINT

ENTRYPOINT ["docker-entrypoint.sh"]

CMD ["uwsgi", "--ini", "/opt/mailman-web/uwsgi.ini"]
