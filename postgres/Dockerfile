FROM debian:jessie
LABEL maintainer "Brett Vickers <github.com/beevik>"

# explicitly set user/group IDs
RUN set -x \
	&& groupadd -r postgres --gid=1000 \
	&& useradd -r -g postgres --uid=1000 postgres

# grab gosu for easy step-down from root
ENV GOSU_VERSION 1.7
RUN set -x \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends ca-certificates wget \
	&& rm -rf /var/lib/apt/lists/* \
	&& wget -q -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
	&& wget -q -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
	&& gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
	&& rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
	&& chmod +x /usr/local/bin/gosu \
	&& gosu nobody true \
	&& apt-get purge -y --auto-remove ca-certificates wget

# make the "en_US.UTF-8" locale so postgres will be utf-8 enabled by default
RUN set -x \
	&& apt-get update \
	&& apt-get install -y locales \
	&& rm -rf /var/lib/apt/lists/* \
	&& localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8

RUN mkdir /docker-entrypoint-initdb.d

RUN apt-key adv --keyserver ha.pool.sks-keyservers.net --recv-keys B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8

ENV PG_MAJOR 9.5

RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ jessie-pgdg main' $PG_MAJOR > /etc/apt/sources.list.d/pgdg.list

RUN set -x \
	&& apt-get update \
	&& apt-get install -y postgresql-common \
	&& sed -ri 's/#(create_main_cluster) .*$/\1 = false/' /etc/postgresql-common/createcluster.conf \
	&& apt-get install -y \
		postgresql-$PG_MAJOR \
		postgresql-contrib-$PG_MAJOR

# make the sample config easier to munge (and "correct by default")
RUN set -x \
	&& mv -v /usr/share/postgresql/$PG_MAJOR/postgresql.conf.sample /usr/share/postgresql/ \
	&& ln -sv ../postgresql.conf.sample /usr/share/postgresql/$PG_MAJOR/ \
	&& sed -ri "s@^#?(listen_addresses)\s*=\s*\S+.*@\1 = '*'@" /usr/share/postgresql/postgresql.conf.sample

RUN set -x \
	&& mkdir -p /var/run/postgresql \
	&& chown -R postgres /var/run/postgresql

ENV PATH /usr/lib/postgresql/$PG_MAJOR/bin:$PATH
ENV PGDATA /var/lib/postgresql/data

VOLUME /var/lib/postgresql/data

COPY files/docker-entrypoint.sh /docker-entrypoint.sh

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 5432
CMD ["postgres"]
