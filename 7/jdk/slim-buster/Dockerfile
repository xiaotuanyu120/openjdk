FROM debian:buster-slim

RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
# utilities for keeping Debian and OpenJDK CA certificates in sync
		ca-certificates p11-kit \
	; \
	rm -rf /var/lib/apt/lists/*

# Default to UTF-8 file.encoding
ENV LANG C.UTF-8

ENV JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64
ENV PATH $JAVA_HOME/bin:$PATH

# backwards compatibility shim
RUN { echo '#/bin/sh'; echo 'echo "$JAVA_HOME"'; } > /usr/local/bin/docker-java-home && chmod +x /usr/local/bin/docker-java-home && [ "$JAVA_HOME" = "$(docker-java-home)" ]

ENV JAVA_VERSION 7u261

RUN set -eux; \
	\
    echo "deb http://security.debian.org/debian-security jessie/updates main" >> /etc/apt/sources.list; \
    echo "deb http://ftp.de.debian.org/debian buster main" >> /etc/apt/sources.list; \
    echo "deb http://ftp.de.debian.org/debian jessie main" >> /etc/apt/sources.list; \
	apt-get update; \
	apt-get install -y --allow-downgrades --no-install-recommends \
        tzdata=2019c-0+deb8u1 \
    ; \
# https://github.com/geerlingguy/ansible-role-java/issues/64#issuecomment-393299088
    mkdir -p /usr/share/man/man1 ;\
    apt-get install -y --no-install-recommends \
        openjdk-7-jre-headless \
        libasound2 \
        libatk1.0-0 \
        libcairo2 \
        libfontconfig1 \
        libgdk-pixbuf2.0-0 libgif4 \
        libgtk2.0-0 \
        libpango-1.0-0 \
        libpangocairo-1.0-0 \
        libpangoft2-1.0-0 \
        libx11-6 \
        libxcomposite1 \
        libxext6 \
        libxi6 \
        libxrender1 \
        libxtst6 \
        libxrandr2 \
        libxinerama1 \
        libgl1-mesa-glx \
        libgl1 \
        libatk-wrapper-java-jni \
        tzdata-java \
	; \
    apt-get install -y --no-install-recommends \
        openjdk-7-jre \
        openjdk-7-jdk \
    ; \
	rm -rf /var/lib/apt/lists/*; \
	\
    update-alternatives --remove java /usr/lib/jvm/java-11-openjdk-amd64/bin/java; \
    rm -rf /usr/lib/jvm/java-11-openjdk-amd64 ; \
	\
    savedAptMark="$(apt-mark showmanual)"; \
# TODO strip "demo" and "man" folders?
	\
	apt-mark auto '.*' > /dev/null; \
	[ -z "$savedAptMark" ] || apt-mark manual $savedAptMark > /dev/null; \
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	\
# update "cacerts" bundle to use Debian's CA certificates (and make sure it stays up-to-date with changes to Debian's store)
# see https://github.com/docker-library/openjdk/issues/327
#     http://rabexc.org/posts/certificates-not-working-java#comment-4099504075
#     https://salsa.debian.org/java-team/ca-certificates-java/blob/3e51a84e9104823319abeb31f880580e46f45a98/debian/jks-keystore.hook.in
#     https://git.alpinelinux.org/aports/tree/community/java-cacerts/APKBUILD?id=761af65f38b4570093461e6546dcf6b179d2b624#n29
	{ \
		echo '#!/usr/bin/env bash'; \
		echo 'set -Eeuo pipefail'; \
		echo 'if ! [ -d "$JAVA_HOME" ]; then echo >&2 "error: missing JAVA_HOME environment variable"; exit 1; fi'; \
# 8-jdk uses "$JAVA_HOME/jre/lib/security/cacerts" and 8-jre and 11+ uses "$JAVA_HOME/lib/security/cacerts" directly (no "jre" directory)
		echo 'cacertsFile=; for f in "$JAVA_HOME/lib/security/cacerts" "$JAVA_HOME/jre/lib/security/cacerts"; do if [ -e "$f" ]; then cacertsFile="$f"; break; fi; done'; \
		echo 'if [ -z "$cacertsFile" ] || ! [ -f "$cacertsFile" ]; then echo >&2 "error: failed to find cacerts file in $JAVA_HOME"; exit 1; fi'; \
		echo 'trust extract --overwrite --format=java-cacerts --filter=ca-anchors --purpose=server-auth "$cacertsFile"'; \
	} > /etc/ca-certificates/update.d/docker-openjdk; \
	chmod +x /etc/ca-certificates/update.d/docker-openjdk; \
	/etc/ca-certificates/update.d/docker-openjdk; \
	\
# https://github.com/docker-library/openjdk/issues/331#issuecomment-498834472
	find "$JAVA_HOME/lib" -name '*.so' -exec dirname '{}' ';' | sort -u > /etc/ld.so.conf.d/docker-openjdk.conf; \
	ldconfig; \
	\
# basic smoke test
	javac -version; \
	java -version
