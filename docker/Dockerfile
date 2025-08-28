FROM alpine:3.20

# Metadata params
ARG VCS_REF
ARG BUILD_DATE

# https://networkupstools.org/download.html
ENV NUT_VERSION_RANGE="2.8"
#ENV NUT_VERSION="2.8.4"
ENV NUT_VERSION="2.8.3"
#ENV NUT_SHA256="a2fe55bc2d90b4a848d6ff8bac361e6d1c97f899a545219cad707d17a27ff127"
ENV NUT_SHA256="d6ca17f0b39003bac7649eb17ab4a713e4d5fcaa8fd1aedca28357d59df095ed"

ENV UPS_NAME="ups"
ENV UPS_DESC="UPS"
ENV UPS_DRIVER="usbhid-ups"
ENV UPS_PORT="auto"

ENV API_PASSWORD=""
ENV ADMIN_PASSWORD=""

ENV SHUTDOWN_CMD="echo 'System shutdown not configured!'"

# Metadata
LABEL org.label-schema.vendor="HB9UQS" \
    org.label-schema.url="https://github.com/hb9uqs/nut-upsd#readme" \
    org.label-schema.name="gh-deployer container" \
    org.label-schema.description="UPS daemon" \
    org.label-schema.version=${NUT_VERSION} \
    org.label-schema.vcs-url="https://github.com/hb9uqs/nut-upsd.git" \
    org.label-schema.vcs-ref=${VCS_REF} \
    org.label-schema.build-date=${BUILD_DATE} \
    org.label-schema.docker.schema-version="1.0"

# Find similar config in: https://git.alpinelinux.org/aports/tree/testing/nut/APKBUILD?id=17c662d69ebf24fbff2fa065d7544863ada956a5
RUN set -ex; \
	# run dependencies
	apk add --update  --no-cache \
		openssh-client \
		libusb-compat \
        nss \
	; \
	# build dependencies
	apk add --update --no-cache --virtual .build-deps \
		libusb-compat-dev \
        libsnmp40 \
        libsnmp-dev \
        net-snmp-dev \
        openssl-dev \
        nss-dev \
        neon-dev \
		build-base \
	; \
	# download and extract
	cd /tmp; \
    printf "%s  nut-${NUT_VERSION}.tar.gz\n" "$NUT_SHA256" > nut-$NUT_VERSION.tar.gz.sha256; \
	wget https://www.networkupstools.org/source/$NUT_VERSION_RANGE/nut-$NUT_VERSION.tar.gz; \
	tar xfz nut-$NUT_VERSION.tar.gz; \
    sha256sum -c -w nut-$NUT_VERSION.tar.gz.sha256; \
	cd nut-$NUT_VERSION \
	; \
	# build
	./configure \
		--prefix=/usr \
		--sysconfdir=/etc/nut \
		--disable-dependency-tracking \
		--enable-strip \
		--disable-static \
		--with-all=no \
		--with-usb=yes \
		--datadir=/usr/share/nut \
        --with-nss \
 		--with-openssl \
        --with-neon \
        --with-snmp \
		--with-drvpath=/usr/share/nut \
		--with-statepath=/var/run/nut \
		--with-user=nut \
		--with-group=nut \
	; \
	# install
	make install \
	; \
	# create nut user
	adduser -D -h /var/run/nut nut; \
	chgrp -R nut /etc/nut; \
	chmod -R o-rwx /etc/nut; \
	install -d -m 750 -o nut -g nut /var/run/nut \
	; \
	# cleanup
	rm -r /tmp/nut-$NUT_VERSION.tar.gz /tmp/nut-$NUT_VERSION.tar.gz.sha256 /tmp/nut-$NUT_VERSION; \
	rm -r /usr/share/man/man5 /usr/share/man/man8; \
	apk del .build-deps

COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]

WORKDIR /var/run/nut

EXPOSE 3493
