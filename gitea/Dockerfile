FROM debian:jessie
LABEL maintainer "Brett Vickers <github.com/beevik>"

# ----------------
# Install Postgres
# ----------------

# explicitly set user/group IDs
RUN set -x \
	&& groupadd -r gitea --gid=1000 \
	&& adduser --disabled-password --gecos 'Gitea' --uid 1000 --ingroup gitea gitea

# grab gosu for easy step-down from root
ENV GOSU_VERSION 1.7
RUN set -x \
	&& sed -i 's/main/main contrib non-free/g' /etc/apt/sources.list \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends ca-certificates git wget locales sudo git openssh-server \
	&& wget -q -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
	&& wget -q -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
	&& gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
	&& rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
	&& chmod +x /usr/local/bin/gosu \
	&& gosu nobody true \
	&& rm -rf /var/lib/apt/lists/*

# make the "en_US.UTF-8" locale so postgres will be utf-8 enabled by default
RUN set -x \
	&& localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8

RUN apt-key adv --keyserver ha.pool.sks-keyservers.net --recv-keys B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8

ENV PG_MAJOR 9.5

RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ jessie-pgdg main' $PG_MAJOR > /etc/apt/sources.list.d/pgdg.list

RUN set -x \
	&& apt-get update \
	&& apt-get install -y postgresql-common \
	&& sed -ri 's/#(create_main_cluster) .*$/\1 = false/' /etc/postgresql-common/createcluster.conf \
	&& apt-get install -y \
		postgresql-$PG_MAJOR \
		postgresql-contrib-$PG_MAJOR \
	&& rm -rf /var/lib/apt/lists/*

# make the sample config easier to munge (and "correct by default")
RUN set -x \
	&& mv -v /usr/share/postgresql/$PG_MAJOR/postgresql.conf.sample /usr/share/postgresql/ \
	&& ln -sv ../postgresql.conf.sample /usr/share/postgresql/$PG_MAJOR/ \
	&& sed -ri "s@^#?(listen_addresses)\s*=\s*\S+.*@#\1 = '*'@" /usr/share/postgresql/postgresql.conf.sample

RUN set -x \
	&& mkdir -p /var/run/postgresql \
	&& chown -R gitea:gitea /var/run/postgresql

ENV PATH /usr/lib/postgresql/$PG_MAJOR/bin:$PATH
ENV PGDATA /var/lib/postgresql/data


# ------------
# Install sshd
# ------------

RUN set -x \
	&& mkdir -p /var/run/sshd \
	&& chmod 700 /var/run/sshd \
	&& sed -i 's/^#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config \
	&& sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd


# -------------
# Install gitea
# -------------

ENV GITEA_VERSION 1.13.2

RUN set -x \
    && cd /tmp \
    && wget -q -O gitea https://github.com/go-gitea/gitea/releases/download/v$GITEA_VERSION/gitea-$GITEA_VERSION-linux-amd64 \
	&& mkdir -p /usr/local/gitea /var/lib/gitea /var/log/gitea \
	&& mv /tmp/gitea /usr/local/gitea \
	&& chown -R gitea:gitea /usr/local/gitea /var/lib/gitea /var/log/gitea \
	&& chmod 770 /usr/local/gitea/gitea

COPY files/launch.sh /usr/local/gitea/launch.sh


# ----------------
# Complete install
# ----------------

COPY files/docker-entrypoint.sh /docker-entrypoint.sh

RUN set -x \
	&& chmod 700 /docker-entrypoint.sh \
	&& chown gitea:gitea /usr/local/gitea/launch.sh \
	&& chmod 775 /usr/local/gitea/launch.sh

VOLUME /var/lib/postgresql/data
VOLUME /var/lib/gitea
VOLUME /var/log/gitea
VOLUME /home/gitea

EXPOSE 22 3000

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["start"]
