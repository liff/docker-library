FROM debian:stretch-slim

RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		bzip2 \
		ca-certificates \
		libffi-dev \
		libgmp-dev \
		libssl1.0-dev \
		libyaml-dev \
		procps \
		zlib1g-dev \
	; \
	rm -rf /var/lib/apt/lists/*

# skip installing gem documentation
RUN set -eux; \
	mkdir -p /usr/local/etc; \
	{ \
		echo 'install: --no-document'; \
		echo 'update: --no-document'; \
	} >> /usr/local/etc/gemrc

ENV RUBY_MAJOR 1.8
ENV RUBY_VERSION 1.8.7-p376
ENV RUBY_BRANCH ruby_1_8_7
ENV RUBY_DOWNLOAD_SHA256 45cc90308e27b276621ac9c5139944a14ff194b802f2e775bc4a9adbe6ba657a

ENV RUBYGEMS_VERSION 1.6.2
ENV RUBYGEMS_DOWNLOAD_SHA256 cb5261818b931b5ea2cb54bc1d583c47823543fcf9682f0d6298849091c1cea7

ENV BUNDLER_VERSION 1.17.3

RUN set -eux; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		autoconf \
		bison \
		dpkg-dev \
		gcc \
		libbz2-dev \
		libgdbm-dev \
		libglib2.0-dev \
		libncurses-dev \
		libreadline-dev \
		libxml2-dev \
		libxslt-dev \
		make \
		wget \
		xz-utils \
		curl \
	; \
	rm -rf /var/lib/apt/lists/*; \
	\
	wget -O ruby.tar.gz https://github.com/ruby/ruby/archive/${RUBY_BRANCH}.tar.gz; \
	echo "$RUBY_DOWNLOAD_SHA256 ruby.tar.gz" | sha256sum --check --strict; \
	\
	mkdir -p /usr/src/ruby; \
	tar -xzf ruby.tar.gz -C /usr/src/ruby --strip-components=1; \
	rm ruby.tar.gz; \
	\
	cd /usr/src/ruby; \
	\
# hack in "ENABLE_PATH_CHECK" disabling to suppress:
#   warning: Insecure world writable dir
	{ \
		echo '#define ENABLE_PATH_CHECK 0'; \
		echo; \
		cat file.c; \
	} > file.c.new; \
	mv file.c.new file.c; \
	\
	curl https://gist.githubusercontent.com/liff/0fbe245a3bcecf3a987db1d7cc59efd3/raw/ab0115752f011c94ca556ebe18ca863e06c71938/ruby-1.8.7-p375-openssl-1.0.patch | patch -p1; \
	autoconf; \
	gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"; \
	./configure \
		--build="$gnuArch" \
		--disable-install-doc \
		--enable-shared \
	; \
	make -j "$(nproc)"; \
	make install; \
	cd /; \
	rm -r /usr/src/ruby; \
	\
	wget -qO rubygems.tar.gz https://rubygems.org/rubygems/rubygems-${RUBYGEMS_VERSION}.tgz; \
	echo "$RUBYGEMS_DOWNLOAD_SHA256 rubygems.tar.gz" | sha256sum -c -; \
	mkdir /usr/src/rubygems; \
	tar -xzf rubygems.tar.gz -C /usr/src/rubygems --strip-components=1; \
	rm rubygems.tar.gz; \
	cd /usr/src/rubygems; \
	ruby ./setup.rb; \
	cd /; \
	rm -r /usr/src/rubygems; \
	\
	gem install bundler -v $BUNDLER_VERSION; \
	\
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark > /dev/null; \
	find /usr/local -type f -executable -not \( -name '*tkinter*' \) -exec ldd '{}' ';' \
		| awk '/=>/ { print $(NF-1) }' \
		| sort -u \
		| xargs -r dpkg-query --search \
		| cut -d: -f1 \
		| sort -u \
		| xargs -r apt-mark manual \
	; \
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	\
# verify we have no "ruby" packages installed
	! dpkg -l | grep -i ruby; \
	[ "$(command -v ruby)" = '/usr/local/bin/ruby' ]; \
# rough smoke test
	ruby --version; \
	gem --version; \
	bundle --version

# install things globally, for great justice
# and don't create ".bundle" in all our apps
ENV GEM_HOME /usr/local/bundle
ENV BUNDLE_PATH="$GEM_HOME" \
	BUNDLE_SILENCE_ROOT_WARNING=1 \
	BUNDLE_APP_CONFIG="$GEM_HOME"
# path recommendation: https://github.com/bundler/bundler/pull/6469#issuecomment-383235438
ENV PATH $GEM_HOME/bin:$BUNDLE_PATH/gems/bin:$PATH
# adjust permissions of a few directories for running "gem install" as an arbitrary user
RUN mkdir -p "$GEM_HOME" && chmod 777 "$GEM_HOME"
# (BUNDLE_PATH = GEM_HOME, no need to mkdir/chown both)

CMD [ "irb" ]
