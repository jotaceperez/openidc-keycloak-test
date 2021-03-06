
# https://github.com/docker-library/repo-info/blob/master/repos/httpd/tag-details.md
FROM httpd:2.4.25@sha256:4612fba4347bd87eaeecd5c522d844f26cc4065b45eef9291277497946b7a86c

ENV OPENIDC_VERSION v2.1.6
ENV OPENIDC_VERSION_TGZ_URL https://github.com/pingidentity/mod_auth_openidc/archive/$OPENIDC_VERSION.tar.gz
ENV OPENIDC_VERSION_TGZ_SHA1 5f45c6a6b5111f007fbfd52efd9ade654bfee905

ENV CJOSE_VERSION 0.4.1
ENV CJOSE_DEB_URL https://github.com/pingidentity/mod_auth_openidc/releases/download/v2.1.0/libcjose_$CJOSE_VERSION-1_amd64.deb
ENV CJOSE_DEB_SHA1 21a2aa687c7e151a4f72cae096e91b857da5294f

ENV DEBIAN_FRONTEND noninteractive

RUN depsRuntime=" \
		libcurl3 ca-certificates \
		libpcre3 \
		libjansson4 \
		libhiredis0.10 \
	" \
  && depsBuild=" \
		curl \
		gcc \
		make \
		libpcre3-dev \
		libssl-dev=$OPENSSL_VERSION \
		libcurl4-openssl-dev \
		libjansson-dev \
		pkg-config \
		dh-autoreconf \
		libhiredis-dev \
	" \
	set -x \
	&& apt-get update \
	&& apt-get install -y --no-install-recommends $depsRuntime \
	&& apt-get install -y --no-install-recommends $depsBuild \
	&& rm -r /var/lib/apt/lists/* \
	&& curl -SL "$CJOSE_DEB_URL" -o libcjose.deb \
	&& echo "$CJOSE_DEB_SHA1 libcjose.deb" | sha1sum -c - \
	&& dpkg -i libcjose.deb \
	&& rm libcjose.deb \
	&& curl -SL "$OPENIDC_VERSION_TGZ_URL" -o mod_auth_openidc-$OPENIDC_VERSION.tar.gz \
	&& echo "$OPENIDC_VERSION_TGZ_SHA1 mod_auth_openidc-$OPENIDC_VERSION.tar.gz" | sha1sum -c - \
	&& mkdir -p src/mod_auth_openidc \
	&& tar -xvf mod_auth_openidc-$OPENIDC_VERSION.tar.gz -C src/mod_auth_openidc --strip-components=1 \
	&& rm mod_auth_openidc-$OPENIDC_VERSION.tar.gz* \
	&& cd src/mod_auth_openidc \
	&& ./autogen.sh \
	&& ./configure --with-apxs2=/usr/local/apache2/bin/apxs \
	&& make -j"$(nproc)" \
	&& make install \
	&& /sbin/ldconfig \
	&& cd ../../ \
	&& rm -r src/mod_auth_openidc \
	&& apt-get purge -y --auto-remove $depsBuild

RUN sed -i 's|LoadModule rewrite_module modules/mod_rewrite.so|LoadModule rewrite_module modules/mod_rewrite.so\nLoadModule auth_openidc_module modules/mod_auth_openidc.so|' conf/httpd.conf
