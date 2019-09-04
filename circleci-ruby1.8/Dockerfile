FROM circleci/buildpack-deps:stretch

USER root

RUN mkdir -p /usr/local/etc && { \
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

RUN set -ex \
    && buildDeps='\
         build-essential \
         autoconf \
         automake \
         libtool \
         bison \
         byacc \
         flex \
         gperf \
         libreadline-dev \
         libgdbm-dev \
         libssl1.0-dev \
         pkg-config \
       ' \
    && apt-get update \
    && apt-get install $buildDeps \
    && apt-get install libffi-dev libxml2-dev libxslt-dev \
    \
    && cd /usr/src \
    && wget -qO ruby.tar.gz https://github.com/ruby/ruby/archive/${RUBY_BRANCH}.tar.gz \
	  && echo "$RUBY_DOWNLOAD_SHA256 ruby.tar.gz" | sha256sum -c - \
    && mkdir /usr/src/ruby \
    && tar -xzf ruby.tar.gz -C /usr/src/ruby --strip-components=1 \
    && rm ruby.tar.gz \
    && cd ruby \
    && curl https://gist.githubusercontent.com/liff/0fbe245a3bcecf3a987db1d7cc59efd3/raw/ab0115752f011c94ca556ebe18ca863e06c71938/ruby-1.8.7-p375-openssl-1.0.patch | patch -p1 \
    && autoconf \
    && ./configure --disable-install-doc --enable-shared \
    && make -j"$(nproc)" \
    && make install \
    && cd .. \
    && rm -fr ruby \
    \
    && wget -qO rubygems.tar.gz https://rubygems.org/rubygems/rubygems-${RUBYGEMS_VERSION}.tgz \
	  && echo "$RUBYGEMS_DOWNLOAD_SHA256 rubygems.tar.gz" | sha256sum -c - \
    && mkdir /usr/src/rubygems \
    && tar -xzf rubygems.tar.gz -C /usr/src/rubygems --strip-components=1 \
    && rm rubygems.tar.gz \
    && cd rubygems \
    && ruby ./setup.rb \
    && cd .. \
    && rm -fr rubygems \
    && gem install bundler -v $BUNDLER_VERSION \
    \
    && apt-get purge -y --auto-remove $buildDeps \
    && rm -rf /var/lib/apt/lists/*

ENV GEM_HOME /usr/local/bundle
ENV BUNDLE_PATH="$GEM_HOME" \
    BUNDLE_SILENCE_ROOT_WARNING=1 \
    BUNDLE_APP_CONFIG="$GEM_HOME"
ENV PATH $GEM_HOME/bin:$BUNDLE_PATH/gems/bin:$PATH
RUN mkdir -p "$GEM_HOME" && chmod 777 "$GEM_HOME"

USER circleci

CMD ["/bin/sh"]
