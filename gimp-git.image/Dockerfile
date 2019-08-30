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
	apt-utils \
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
	gobject-introspection \
	meson \
	--no-install-recommends
	
# compile dir

ENV PREFIX=/usr/local
ENV PATH=$PREFIX/bin:$PATH
ENV PKG_CONFIG_PATH=$PREFIX/lib/pkgconfig:$PREFIX/share/pkgconfig:/usr/lib/pkgconfig:/usr/share/pkgconfig

# sources dir

ENV SRCDIR=/usr/src/gimp-git
RUN mkdir -p $SRCDIR

WORKDIR $SRCDIR

RUN ldconfig


# ====

FROM builder AS build-babl

RUN git clone https://gitlab.gnome.org/GNOME/babl.git

RUN DEBIAN_FRONTEND=noninteractive apt-get install -y \
	liblcms2-dev \
	--no-install-recommends

RUN ldconfig

WORKDIR $SRCDIR/babl
RUN meson --prefix=$PREFIX build
WORKDIR $SRCDIR/babl/build
RUN ninja -j`nproc`
RUN ninja install


# ====

FROM builder AS build-gegl

RUN git clone https://gitlab.gnome.org/GNOME/gegl.git

# TODO: enable more
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y \
	gtk-doc-tools \
	libgexiv2-dev \
	libgs-dev \
	libjson-glib-dev \
	libopenexr-dev \
	librsvg2-dev \
	python \
	python-dev \
	python-gtk2-dev \
	python-cairo-dev \
	ruby \
	libgirepository1.0-dev \
	--no-install-recommends

COPY --from=build-babl /usr/local /usr/local
COPY --from=build-babl /usr/local/share/gir-1.0 /usr/share/gir-1.0

RUN ldconfig

WORKDIR $SRCDIR/gegl
RUN meson --prefix=$PREFIX build
WORKDIR $SRCDIR/gegl/build
RUN ninja -j`nproc`
RUN ninja install


# ====

FROM builder AS build-libmypaint

RUN git clone https://github.com/mypaint/libmypaint.git

RUN DEBIAN_FRONTEND=noninteractive apt-get install -y \
	libjson-c-dev \
	libjson-c4 \
	python \
	--no-install-recommends

WORKDIR $SRCDIR/libmypaint

RUN git checkout libmypaint-v1

RUN ./autogen.sh --prefix=$PREFIX
RUN ./configure --disable-gegl
RUN make -j`nproc`
RUN make install


# ====

FROM builder AS build-gimp

RUN git clone https://gitlab.gnome.org/GNOME/gimp.git

RUN apt-get install -y \
	autopoint \
	gettext \
	gtk-3-examples \
	libaa1-dev \
	libappstream-glib-dev \
	libasound2-dev \
	libbz2-dev \
	libexif-dev \
	libgexiv2-dev \
	libgtk2.0-dev \
	libgtk-3-dev \
	libjson-glib-dev \
	liblcms2-dev \
	libmng-dev \
	libpoppler-glib-dev \
	libtiff5-dev \
	libwebp-dev \
	libwmf-dev \
	libxpm-dev \
	mypaint-brushes \
	openexr \
	poppler-data \
	valac \
	xsltproc \
	libgirepository1.0-dev \
	gjs \
	luajit \
	python-gobject-2-dev \
	libopenexr-dev \
	libheif-dev \
	libopenjp2-7-dev \
	python3-gi \
	libjson-c-dev \
	librsvg2-dev \
	--no-install-recommends

COPY --from=build-babl /usr/local /usr/local
COPY --from=build-gegl /usr/local /usr/local
COPY --from=build-libmypaint /usr/local /usr/local

RUN ldconfig

WORKDIR $SRCDIR/gimp
RUN ./autogen.sh --prefix=$PREFIX --disable-gtk-doc --without-webkit
RUN make -j`nproc`
RUN make install


# ====

FROM debian:testing
MAINTAINER Jaroslaw Filiochowski <jarfil@gmail.com>

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get -y update
RUN apt-get -y upgrade
RUN apt-get -y autoremove

ENV PREFIX=/usr/local
ENV PATH=$PREFIX/bin:$PATH

# copy
COPY --from=build-gimp /usr/local/ /usr/local/

# depends
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y \
	gjs \
	libaa1 \
	libappstream-glib8 \
	libasound2 \
	libbz2-1.0 \
	libexif12 \
	libgexiv2-2 \
	libgs9 \
	libgtk2.0-0 \
	libgtk2.0-bin \
	libgtk-3-0 \
	libgtk-3-bin \
	libheif1 \
	libjson-c4 \
	libjson-glib-1.0-0 \
	liblcms2-2 \
	libmng1 \
	libopenexr23 \
	libopenjp2-7 \
	libpoppler-glib8 \
	librsvg2-2 \
	libtiff5 \
	libwebp6 \
	libwmf0.2-7 \
	libxpm4 \
	luajit \
	mypaint-brushes \
	openexr \
	poppler-data \
	python \
	python-cairo \
	python-gobject-2 \
	python-gtk2 \
	ruby \
	valac \
	xsltproc \
	--no-install-recommends

RUN ldconfig

# final binary
RUN ln -s `ls /usr/local/bin/gimp-?.* | head -n2` /usr/local/bin/gimp

# entry point
WORKDIR /
ENTRYPOINT ["/usr/local/bin/gimp"]
