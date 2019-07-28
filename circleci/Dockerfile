FROM circleci/buildpack-deps:buster
MAINTAINER Mossuru777 "mossuru777@gmail.com"

ENV DEBIAN_FRONTEND noninteractive

USER root

# Setup
RUN apt-get -y update \
    && apt-get -y upgrade \
    && apt-get -y install --no-install-recommends \
        apt-utils \
        nkf

# Install Perl 5.10.1 & App::cpanminus
# (ref: https://github.com/Perl/docker-perl/tree/9c264844428eaef70ec0c8eefc213cae53fdf12f/5.010.001-main)
COPY *.patch /usr/src/perl/
WORKDIR /usr/src/perl
RUN true \
    && curl -SL https://www.cpan.org/src/5.0/perl-5.10.1.tar.bz2 -o perl-5.10.1.tar.bz2 \
    && echo '9385f2c8c2ca8b1dc4a7c31903f1f8dc8f2ba867dc2a9e5c93012ed6b564e826 *perl-5.10.1.tar.bz2' | sha256sum -c - \
    && tar --strip-components=1 -xaf perl-5.10.1.tar.bz2 -C /usr/src/perl \
    && rm perl-5.10.1.tar.bz2 \
    && cat *.patch | patch -p1 \
    && gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
    && archBits="$(dpkg-architecture --query DEB_BUILD_ARCH_BITS)" \
    && archFlag="$([ "$archBits" = '64' ] && echo '-Duse64bitall' || echo '-Duse64bitint')" \
    && ./Configure -Darchname="$gnuArch" "$archFlag" -Duseshrplib -Dvendorprefix=/usr/local -A ccflags=-fwrapv -des \
    && make -j$(nproc) \
    && TEST_JOBS=$(nproc) make test_harness \
    && make install \
    && cd /usr/src \
    && curl -LO https://www.cpan.org/authors/id/M/MI/MIYAGAWA/App-cpanminus-1.7044.tar.gz \
    && echo '9b60767fe40752ef7a9d3f13f19060a63389a5c23acc3e9827e19b75500f81f3 *App-cpanminus-1.7044.tar.gz' | sha256sum -c - \
    && tar -xzf App-cpanminus-1.7044.tar.gz && cd App-cpanminus-1.7044 \
    && perl bin/cpanm . \
    && cd /root \
    && rm -fr ./cpanm /root/.cpanm /usr/src/perl /usr/src/App-cpanminus-1.7044* /tmp/** \
    && true

# Install ImageMagick & PerlMagick
WORKDIR /usr/src/imagemagick
RUN apt-get -y install --no-install-recommends \
        libgif7 \
        libgif-dev \
        libpng16-16 \
        libpng-dev \
    && apt-get -y clean \
    && curl -SL https://github.com/ImageMagick/ImageMagick6/archive/6.9.10-56.tar.gz -o imagemagick-6.9.10-56.tar.gz \
    && echo 'a80f448ea2d0abe52a9a91ae0ce29c90569f7aafd6143a2fe4cfe4a0c7893dbc *imagemagick-6.9.10-56.tar.gz' | sha256sum -c - \
    && tar --strip-components=1 -xaf imagemagick-6.9.10-56.tar.gz -C /usr/src/imagemagick \
    && rm imagemagick-6.9.10-56.tar.gz \
    && ./configure LDFLAGS="-L/usr/local/lib/perl5/5.10.1/x86_64-linux-gnu/CORE" --enable-shared --with-perl=/usr/local/bin/perl \
    && make -j$(nproc) \
    && make install \
    && cd /root \
    && rm -fr /usr/src/imagemagick /tmp/** \
    && true

WORKDIR /home/circleci
USER circleci

CMD /bin/sh
