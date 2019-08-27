FROM debian:testing AS builder

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get -y update
RUN apt-get -y upgrade
RUN apt-get -y autoremove

# tools
RUN apt-get install -y \
	ca-certificates \
	git-core \
	wget \
	--no-install-recommends

# common depends
RUN apt-get install -y \
	autoconf \
	automake \
	build-essential \
	intltool \
	libglib2.0-0 \
	libglib2.0-dev \
	libtool \
	--no-install-recommends
	
# compile dir

ENV PREFIX=/usr/local
ENV PATH=$PREFIX/bin:$PATH
ENV PKG_CONFIG_PATH=$PREFIX/lib/pkgconfig:$PREFIX/share/pkgconfig


ENV SRCDIR=/usr/src/gimp-git
RUN mkdir -p $SRCDIR

WORKDIR $SRCDIR
RUN git clone https://gitlab.gnome.org/GNOME/babl.git
RUN git clone https://gitlab.gnome.org/GNOME/gegl.git
RUN git clone https://github.com/mypaint/libmypaint.git
RUN git clone https://gitlab.gnome.org/GNOME/gimp.git

RUN DEBIAN_FRONTEND=noninteractive apt-get install -y \
	gobject-introspection \
	meson \
	--no-install-recommends

WORKDIR $SRCDIR/babl
RUN meson --prefix=$PREFIX build
WORKDIR $SRCDIR/babl/build
RUN ninja -j`nproc`
RUN ninja install


# TODO: enable more
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y \
	gtk-doc-tools \
	libgexiv2-2 \
	libgexiv2-dev \
	libgs9 \
	libgs-dev \
	libjson-glib-1.0-0 \
	libjson-glib-dev \
	libopenexr23 \
	libopenexr-dev \
	librsvg2-2 \
	librsvg2-dev \
	python \
	python-dev \
	python-gtk2 \
	python-gtk2-dev \
	python-cairo \
	python-cairo-dev \
	ruby \
	--no-install-recommends

WORKDIR $SRCDIR/gegl
RUN ./autogen.sh --prefix=$PREFIX
RUN make -j`nproc`
RUN make install


RUN DEBIAN_FRONTEND=noninteractive apt-get install -y \
	libjson-c-dev \
	libjson-c4 \
	--no-install-recommends

WORKDIR $SRCDIR/libmypaint

RUN git checkout libmypaint-v1
RUN git config --global user.email "auto@example.com"
RUN git config --global user.name "Automatic"
# backport for automake 1.16
RUN git cherry-pick -x 40d9077a80be13942476f164bddfabe842ab2a45

RUN ./autogen.sh --prefix=$PREFIX
RUN ./configure --disable-gegl
RUN make -j`nproc`
RUN make install


RUN apt-get install -y \
	autopoint \
	gettext \
	gtk-3-examples \
	libaa1 \
	libaa1-dev \
	libappstream-glib-dev \
	libasound2 \
	libasound2-dev \
	libbz2-1.0 \
	libbz2-dev \
	libexif12 \
	libexif-dev \
	libgtk2.0-0 \
	libgtk2.0-bin \
	libgtk2.0-dev \
	libgtk-3-dev \
	liblcms2-2 \
	liblcms2-dev \
	libmng1 \
	libmng-dev \
	libpoppler-glib8 \
	libpoppler-glib-dev \
	libtiff5 \
	libtiff5-dev \
	libwebp6 \
	libwebp-dev \
	libwmf0.2-7 \
	libwmf-dev \
	libxpm4 \
	libxpm-dev \
	mypaint-brushes \
	openexr \
	valac \
	xsltproc \
	--no-install-recommends

WORKDIR $SRCDIR/gimp
RUN ./autogen.sh --prefix=$PREFIX --disable-gtk-doc --without-webkit
RUN make -j`nproc`
RUN make install

RUN ldconfig

# final binary

# ln -s `ls /usr/local/bin/gimp-?.* | head -n2` /usr/local/bin/gimp



FROM debian:testing
MAINTAINER Jaroslaw Filiochowski <jarfil@gmail.com>

# copy
COPY --from=builder /usr/local/ /usr/local/

# entry point
WORKDIR /
ENTRYPOINT /usr/local/bin/gimp
