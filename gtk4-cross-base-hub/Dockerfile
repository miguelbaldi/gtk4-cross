FROM fedora:40
WORKDIR /root
RUN dnf install -y git cmake gcc-c++ boost-devel
RUN git clone https://github.com/gsauthof/pe-util
WORKDIR /root/pe-util
RUN git submodule update --init
RUN mkdir build
WORKDIR /root/pe-util/build
RUN cmake .. -DCMAKE_BUILD_TYPE=Release
RUN make

FROM fedora:40
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y
# RUN . ~/.cargo/env 
# Set the rust linker to gcc
# RUN echo "[target.x86_64-pc-windows-gnu]\nlinker = \"x86_64-w64-mingw32-gcc\"\nar = \"x86_64-w64-mingw32-gcc-ar\"" > /home/rust/.cargo/config
RUN dnf install -y gcc
RUN . ~/.cargo/env && cargo install --git https://github.com/MGlolenstine/peldd_dependency_scanner.git

FROM fedora:40
COPY --from=0 /root/pe-util/build/peldd /usr/bin/peldd
COPY --from=1 /root/.cargo/bin/peldd_dependency_scanner /usr/bin/pds
RUN dnf install git glslc mingw64-vulkan-headers mingw64-vulkan-tools mingw64-vulkan-loader mingw64-openssl-static mingw64-gcc-c++ zstd cyrus-sasl-devel perl mingw64-binutils mingw64-gcc meson mingw64-pango mingw64-gdk-pixbuf mingw64-libepoxy mingw64-winpthreads-static glib2-devel gobject-introspection-devel mingw64-gstreamer1-plugins-bad-free vala wine cmake gcc zip boost boost-system dos2unix sassc mingw64-librsvg2 intltool itstool gtk-update-icon-cache adwaita-icon-theme mingw64-adwaita-icon-theme mingw64-gettext mingw64-libyaml mingw64-xz mingw64-xz-libs mingw64-zstd gperf -y && dnf clean all -y

# Clone the gtk4 repository (Move after the mingw setup for efficiency later)
WORKDIR /tmp
RUN git clone --depth 1 --branch %GTKTAG% https://gitlab.gnome.org/GNOME/gtk.git

# Clone the libadwaita repository (Move after the mingw setup for efficiency later)
WORKDIR /tmp
RUN git clone --depth 1 --branch %ADWTAG% https://gitlab.gnome.org/GNOME/libadwaita.git

# Make the meson use mingw
ADD x86_64-meson /usr/bin
RUN dos2unix /usr/bin/x86_64-meson
RUN chmod +x /usr/bin/x86_64-meson

# Add mingw-env
ADD mingw-env /usr/bin
RUN dos2unix /usr/bin/mingw-env
RUN chmod +x /usr/bin/mingw-env

# Add package.sh
ADD package.sh /usr/bin/package
RUN dos2unix /usr/bin/package
RUN chmod +x /usr/bin/package

# Update with the new toolchain file
ADD toolchain-mingw64.meson /usr/share/mingw
RUN dos2unix /usr/share/mingw/toolchain-mingw64.meson

# Hacks for GTK4 to compile
#ENV PKG_CONFIG_PATH /usr/x86_64-w64-mingw32/lib/pkgconfig
RUN cp /usr/x86_64-w64-mingw32/sys-root/mingw/include/windows.h /usr/x86_64-w64-mingw32/sys-root/mingw/include/Windows.h

RUN if [[ -e /tmp/gtk/gdk/loaders/gdkjpeg.c ]] ; then echo -e "#include <stdlib.h>\n$(cat /tmp/gtk/gdk/loaders/gdkjpeg.c)" > /tmp/gtk/gdk/loaders/gdkjpeg.c ; fi

# Enable Iphlpapi (on windows, they're case-insensitive, on linux they're not.)
RUN cp /usr/x86_64-w64-mingw32/sys-root/mingw/lib/libiphlpapi.a /usr/x86_64-w64-mingw32/sys-root/mingw/lib/libIphlpapi.a

# Hacks for libadwaita
WORKDIR /tmp
RUN dnf install pacman -y
RUN rm -fv /etc/pacman.conf
RUN rm -fv /etc/pacman.d/mirrorlist
RUN git clone --depth 1 https://github.com/msys2/MSYS2-packages.git

RUN perl -i -pe 's/^SigLevel[\s]*=[\s]*Required/#SigLevel = Required/' MSYS2-packages/pacman/pacman.conf
RUN perl -i -pe 's/^#SigLevel[\s]*=[\s]*Never/SigLevel = Never/' MSYS2-packages/pacman/pacman.conf

RUN cp MSYS2-packages/pacman/pacman.conf /etc/
RUN cp MSYS2-packages/pacman-mirrors/mirrorlist.msys /etc/pacman.d/
RUN cp MSYS2-packages/pacman-mirrors/mirrorlist.mingw /etc/pacman.d/

RUN pacman -Sy --noconfirm mingw-w64-x86_64-appstream

# Installing appstream/xmlb
RUN cp -rfv /mingw64/lib/pkgconfig/appstream.pc /usr/x86_64-w64-mingw32/sys-root/mingw/lib/pkgconfig/
RUN perl -i -pe 's/mingw64/usr\/x86_64-w64-mingw32\/sys-root\/mingw/' /usr/x86_64-w64-mingw32/sys-root/mingw/lib/pkgconfig/appstream.pc
RUN cp -rfv /mingw64/share/appstream /usr/x86_64-w64-mingw32/sys-root/mingw/share/
RUN cp -rfv /mingw64/share/metainfo/org.freedesktop.appstream.cli.metainfo.xml /usr/x86_64-w64-mingw32/sys-root/mingw/share/metainfo/
RUN cp -rfv /mingw64/include/appstream /usr/x86_64-w64-mingw32/sys-root/mingw/include/
RUN cp -rfv /mingw64/bin/appstreamcli.exe /usr/x86_64-w64-mingw32/sys-root/mingw/bin/
RUN cp -rfv /mingw64/lib/libappstream.dll.a /usr/x86_64-w64-mingw32/sys-root/mingw/lib/
RUN cp -rfv /mingw64/bin/libappstream-5.dll /usr/x86_64-w64-mingw32/sys-root/mingw/bin/

RUN cp -rfv /mingw64/lib/pkgconfig/xmlb.pc /usr/x86_64-w64-mingw32/sys-root/mingw/lib/pkgconfig/
RUN perl -i -pe 's/mingw64/usr\/x86_64-w64-mingw32\/sys-root\/mingw/' /usr/x86_64-w64-mingw32/sys-root/mingw/lib/pkgconfig/xmlb.pc
RUN cp -rfv /mingw64/include/libxmlb-2/ /usr/x86_64-w64-mingw32/sys-root/mingw/include/
RUN cp -rfv /mingw64/lib/libxmlb.dll.a /usr/x86_64-w64-mingw32/sys-root/mingw/lib/
RUN cp -rfv /mingw64/bin/libxmlb-2.dll /usr/x86_64-w64-mingw32/sys-root/mingw/bin/

# Installing gettext

RUN /bin/cp -fv /mingw64/bin/envsubst.exe /usr/x86_64-w64-mingw32/sys-root/mingw/bin/
RUN /bin/cp -fv /mingw64/bin/gettext.exe /usr/x86_64-w64-mingw32/sys-root/mingw/bin/
RUN /bin/cp -fv /mingw64/bin/gettext.sh /usr/x86_64-w64-mingw32/sys-root/mingw/bin/
RUN /bin/cp -fv /mingw64/bin/libasprintf-0.dll /usr/x86_64-w64-mingw32/sys-root/mingw/bin/
RUN /bin/cp -fv /mingw64/bin/libintl-8.dll /usr/x86_64-w64-mingw32/sys-root/mingw/bin/
RUN /bin/cp -fv /mingw64/bin/ngettext.exe /usr/x86_64-w64-mingw32/sys-root/mingw/bin/

RUN /bin/cp -fv /mingw64/include/autosprintf.h /usr/x86_64-w64-mingw32/sys-root/mingw/include/
RUN /bin/cp -fv /mingw64/include/libintl.h /usr/x86_64-w64-mingw32/sys-root/mingw/include/

RUN /bin/cp -fv /mingw64/lib/libasprintf.a /usr/x86_64-w64-mingw32/sys-root/mingw/lib/
RUN /bin/cp -fv /mingw64/lib/libasprintf.dll.a /usr/x86_64-w64-mingw32/sys-root/mingw/lib/
RUN /bin/cp -fv /mingw64/lib/libintl.a /usr/x86_64-w64-mingw32/sys-root/mingw/lib/
RUN /bin/cp -fv /mingw64/lib/libintl.dll.a /usr/x86_64-w64-mingw32/sys-root/mingw/lib/

# Installing iconv

RUN /bin/cp -fv /mingw64/bin/libcharset-1.dll /usr/x86_64-w64-mingw32/sys-root/mingw/bin/
RUN /bin/cp -fv /mingw64/bin/libiconv-2.dll /usr/x86_64-w64-mingw32/sys-root/mingw/bin/

RUN /bin/cp -fv /mingw64/include/iconv.h /usr/x86_64-w64-mingw32/sys-root/mingw/include/
RUN /bin/cp -fv /mingw64/include/libcharset.h /usr/x86_64-w64-mingw32/sys-root/mingw/include/
RUN /bin/cp -fv /mingw64/include/localcharset.h /usr/x86_64-w64-mingw32/sys-root/mingw/include/

RUN /bin/cp -fv /mingw64/lib/libcharset.a /usr/x86_64-w64-mingw32/sys-root/mingw/lib/
RUN /bin/cp -fv /mingw64/lib/libcharset.dll.a /usr/x86_64-w64-mingw32/sys-root/mingw/lib/
RUN /bin/cp -fv /mingw64/lib/libiconv.a /usr/x86_64-w64-mingw32/sys-root/mingw/lib/
RUN /bin/cp -fv /mingw64/lib/libiconv.dll.a /usr/x86_64-w64-mingw32/sys-root/mingw/lib/

RUN /bin/cp -rfv /mingw64/share/locale/ /usr/x86_64-w64-mingw32/sys-root/mingw/share/

# Add compiler paths
ENV PKG_CONFIG_ALLOW_CROSS=1
ENV PKG_CONFIG_PATH=/usr/x86_64-w64-mingw32/sys-root/mingw/lib/pkgconfig/:/usr/x86_64-w64-mingw32/lib/pkgconfig/
ENV GTK_INSTALL_PATH=/usr/x86_64-w64-mingw32/sys-root/mingw
ENV MINGW_PREFIX=/usr/x86_64-w64-mingw32/sys-root/mingw

# Fix paths in the pixbuf loaders.cache
    # Required for gdk-pixbuf-query-loaders.exe and other tools to work
ENV LANG="en_GB.utf8"
RUN wine --version
RUN wine $MINGW_PREFIX/bin/gdk-pixbuf-query-loaders.exe --update-cache

# Build and install GTK4
WORKDIR /tmp/gtk
RUN x86_64-meson build -Dintrospection=disabled --wrap-mode=default
WORKDIR /tmp/gtk/build
RUN meson configure -Db_lto=false -Dc_link_args="['-lssp']" -Dc_args=-DG_ENABLE_DEBUG -Dtracker=disabled -Dvulkan=disabled
RUN ninja all && ninja install

# Build and install libadwaita
WORKDIR /tmp/libadwaita
RUN x86_64-meson -Dintrospection=disabled -Dexamples=false -Dvapi=false build --wrap-mode=default
WORKDIR /tmp/libadwaita/build
RUN meson configure -Db_lto=false -Dc_link_args="['-lssp']"
RUN ninja all && ninja install

# Return to the current working directory
WORKDIR /mnt

CMD ["/bin/bash"]
