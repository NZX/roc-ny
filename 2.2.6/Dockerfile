FROM nzxsed/centos-yarn:latest

MAINTAINER SED Team, NZX Limited

# skip installing gem documentation
RUN mkdir -p /usr/local/etc \
  && { \
    echo 'install: --no-document'; \
    echo 'update: --no-document'; \
  } >> /usr/local/etc/gemrc

ENV RUBY_MAJOR 2.2
ENV RUBY_VERSION 2.2.6
ENV RUBY_DOWNLOAD_SHA256 de8e192791cb157d610c48a9a9ff6e7f19d67ce86052feae62b82e3682cc675f
ENV RUBYGEMS_VERSION 2.2.4
ENV BUNDLER_VERSION 1.13.7

# install build dependencies
RUN yum -y install autoconf gcc make pax-utils wget readline-devel && \
    yum clean all

# LibreSSL Dynamic Library
RUN wget http://ftp.openbsd.org/pub/OpenBSD/LibreSSL/libressl-2.1.6.tar.gz && \
    tar -xzvf libressl-2.1.6.tar.gz && cd libressl-2.1.6 && \
    ./configure && make && make install && ldconfig && cd ../ && rm -rf libressl-2.1.6

# install ruby
RUN curl -fSL -o ruby.tar.gz "http://cache.ruby-lang.org/pub/ruby/$RUBY_MAJOR/ruby-$RUBY_VERSION.tar.gz" \
  && echo "$RUBY_DOWNLOAD_SHA256 *ruby.tar.gz" | sha256sum -c - \
  && mkdir -p /usr/src \
  && tar -xzf ruby.tar.gz -C /usr/src \
  && mv "/usr/src/ruby-$RUBY_VERSION" /usr/src/ruby \
  && rm ruby.tar.gz \
  && cd /usr/src/ruby \
  && { echo '#define ENABLE_PATH_CHECK 0'; echo; cat file.c; } > file.c.new && mv file.c.new file.c \
  && autoconf \
  # the configure script does not detect isnan/isinf as macros
  && ac_cv_func_isnan=yes ac_cv_func_isinf=yes \
    ./configure --disable-install-doc \
  && make -j"$(getconf _NPROCESSORS_ONLN)" \
  && make install \
  && runDeps="$( \
    scanelf --needed --nobanner --recursive /usr/local \
      | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
      | sort -u \
      | xargs -r apk info --installed \
      | sort -u \
  )" \
  && gem update --system $RUBYGEMS_VERSION \
  && rm -r /usr/src/ruby \
  && curl -fSL -o /usr/local/lib/ruby/site_ruby/2.2.0/rubygems/ssl_certs/GlobalSignRootCA.pem \
     "https://raw.githubusercontent.com/rubygems/rubygems/master/lib/rubygems/ssl_certs/index.rubygems.org/GlobalSignRootCA.pem"


RUN gem install bundler --version "$BUNDLER_VERSION"
