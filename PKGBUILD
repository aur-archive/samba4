# Maintainer: Michael Hansen <zrax0111 gmail com>
# Contributor: Marco A Rojas <marquicus at gmail.com>
# Contributor: Netanel Shine <netanel at archlinux.org.il >
# Contributor: ngoonee <ngoonee.talk@gmail.com>
# Contributor: Adam Russell <adamlr6+arch@gmail.com>
# Contributor: Dhananjay Sathe <dhananjaysathe@gmail.com>

# Hack for AUR
pkgname='samba4'
true && pkgname=('libwbclient' 'smbclient' 'samba')

pkgbase='samba4'
pkgver=4.0.4
# We use the 'A' to fake out pacman's version comparators.  Samba chooses
# to append 'a','b',etc to their subsequent releases, which pamcan
# misconstrues as alpha, beta, etc.  Bad samba!
_realver=4.0.4
pkgrel=2
arch=('i686' 'x86_64')
url="http://www.samba.org"
license=('GPL3')
makedepends=('python2' 'docbook-xsl' 'pkg-config' 'tdb' 'talloc' 'gnutls'
             'openldap' 'libbsd' 'libcups' 'ldb>=1.1.15' 'tevent' 'readline'
             'dnsutils')
options=(!makeflags)
source=(http://ftp.samba.org/pub/samba/stable/samba-${_realver}.tar.gz
        samba-4.0.3-fix_pidl_with_gcc48.patch)
sha256sums=('20a84280155543892ce939e70482243396a9a8bfa77dcb4bf58328f7029772c5'
            '5e119486bdff16626e7597ba7350e1de7fe7fba0070ef1c2abb51d34b7bc2f0c')

build() {
    # Use samba-pkg as a staging directory for the split packages
    # (This is so RPATHS and symlinks are generated correctly via
    # make install, but the otherwise unsplit pieces can be split)
    _pkgsrc=${srcdir}/samba-pkg

    rm -rf ${_pkgsrc}
    cd ${srcdir}/samba-${_realver}

    # Fix compilation with gcc 4.8
    patch -Np1 -i ${srcdir}/samba-4.0.3-fix_pidl_with_gcc48.patch

    # change to use python2
    SAVEIFS=${IFS}
    IFS=$(echo -en "\n\b")
    PYTHON_CALLERS="$(find ${srcdir}/samba-${_realver} -name '*.py')
$(find ${srcdir}/samba-${_realver} -name 'wscript*')
$(find ${srcdir}/samba-${_realver} -name 'configure.ac')
$(find ${srcdir}/samba-${_realver} -name 'upgrade_from_s3')
$(find ${srcdir}/samba-${_realver}/buildtools -type f)
$(find ${srcdir}/samba-${_realver}/source4/scripting -type f)"
    sed -i -e "s|/usr/bin/env python$|/usr/bin/env python2|" \
           -e "s|python-config|python2-config|" \
           -e "s|bin/python|bin/python2|" \
        ${PYTHON_CALLERS}
    IFS=${SAVEIFS}

    export PYTHON=/usr/bin/python2

    cd ${srcdir}/samba-${_realver}
    ./configure --enable-fhs \
                --prefix=/usr \
                --libdir=/usr/lib \
                --localstatedir=/var \
                --with-configdir=/etc/samba \
                --with-lockdir=/var/cache/samba \
                --with-sockets-dir=/var/run/samba \
                --with-piddir=/var/run \
                --with-ads \
                --with-ldap \
                --with-swat \
                --with-winbind \
                --with-acl-support \
                --enable-gnutls \
                --disable-rpath-install

                # Add this to the options once it's working...
                #--with-system-mitkrb5 /opt/heimdal
    make || return 1
    make DESTDIR="${_pkgsrc}/" install || return 1

    # This gets skipped somehow
    if [ ! -e ${_pkgsrc}/usr/bin/smbtar ]; then
        install -m755 ${srcdir}/samba-${_realver}/source3/script/smbtar ${_pkgsrc}/usr/bin/
    fi
}

package_libwbclient() {
pkgdesc="Samba winbind client library"
depends=('glibc' 'libbsd')

    # Use samba-pkg as a staging directory for the split packages
    # (This is so RPATHS and symlinks are generated correctly via
    # make install, but the otherwise unsplit pieces can be split)
    _pkgsrc=${srcdir}/samba-pkg

    install -d -m755 ${pkgdir}/usr/lib
    mv ${_pkgsrc}/usr/lib/libwbclient*.so* ${pkgdir}/usr/lib/

    install -d -m755 ${pkgdir}/usr/lib/samba
    mv ${_pkgsrc}/usr/lib/samba/libwinbind-client*.so* ${pkgdir}/usr/lib/samba/
    mv ${_pkgsrc}/usr/lib/samba/libreplace.so* ${pkgdir}/usr/lib/samba/

    install -d -m755 ${pkgdir}/usr/lib/pkgconfig
    mv ${_pkgsrc}/usr/lib/pkgconfig/wbclient.pc ${pkgdir}/usr/lib/pkgconfig/

    install -d -m755 ${pkgdir}/usr/include/samba-4.0
    mv ${_pkgsrc}/usr/include/samba-4.0/wbclient.h ${pkgdir}/usr/include/samba-4.0/
}


package_smbclient() {
pkgdesc="Tools to access a server's filespace and printers via SMB"
depends=('popt' 'cifs-utils' 'tdb' 'libwbclient>=4.0.2-1' 'ldb>=1.1.15'
         'tevent' 'talloc' 'readline' 'gnutls' 'openldap' 'libcups' 'dnsutils')

    _smbclient_bins=('smbclient' 'smbclient4' 'rpcclient' 'smbspool'
                     'smbtree' 'smbcacls' 'smbcquotas' 'smbget' 'net'
                     'nmblookup' 'nmblookup4' 'smbtar')

    # Use samba-pkg as a staging directory for the split packages
    # (This is so RPATHS and symlinks are generated correctly via
    # make install, but the otherwise unsplit pieces can be split)
    _pkgsrc=${srcdir}/samba-pkg

    install -d -m755 ${pkgdir}/usr/bin
    for bin in ${_smbclient_bins[@]}; do
        mv ${_pkgsrc}/usr/bin/${bin} ${pkgdir}/usr/bin/
    done

    # smbclient binaries link to the majority of the samba
    # libs, so this is a shortcut instead of resolving the
    # whole dependency tree by hand
    install -d -m755 ${pkgdir}/usr/lib
    for lib in ${_pkgsrc}/usr/lib/lib*.so*; do
        mv ${lib} ${pkgdir}/usr/lib/
    done

    install -d -m755 ${pkgdir}/usr/lib/samba
    for lib in ${_pkgsrc}/usr/lib/samba/lib*.so*; do
        mv ${lib} ${pkgdir}/usr/lib/samba/
    done

    install -d -m755 ${pkgdir}/usr/lib/pkgconfig
    mv ${_pkgsrc}/usr/lib/pkgconfig/smbclient.pc ${pkgdir}/usr/lib/pkgconfig/
    mv ${_pkgsrc}/usr/lib/pkgconfig/smbclient-raw.pc ${pkgdir}/usr/lib/pkgconfig/
    mv ${_pkgsrc}/usr/lib/pkgconfig/netapi.pc ${pkgdir}/usr/lib/pkgconfig/

    install -d -m755 ${pkgdir}/usr/share/man/man1
    install -d -m755 ${pkgdir}/usr/share/man/man7
    install -d -m755 ${pkgdir}/usr/share/man/man8
    for bin in ${_smbclient_bins[@]}; do
        if [ -e ${_pkgsrc}/usr/share/man/man1/${bin}.1 ]; then
            mv ${_pkgsrc}/usr/share/man/man1/${bin}.1 ${pkgdir}/usr/share/man/man1/
        fi
        if [ -e ${_pkgsrc}/usr/share/man/man8/${bin}.8 ]; then
            mv ${_pkgsrc}/usr/share/man/man8/${bin}.8 ${pkgdir}/usr/share/man/man8/
        fi
    done
    mv ${_pkgsrc}/usr/share/man/man7/libsmbclient.7 ${pkgdir}/usr/share/man/man7/

    install -d -m755 ${pkgdir}/usr/include/samba-4.0
    mv ${_pkgsrc}/usr/include/samba-4.0/libsmbclient.h ${pkgdir}/usr/include/samba-4.0/
    mv ${_pkgsrc}/usr/include/samba-4.0/netapi.h ${pkgdir}/usr/include/samba-4.0/

    mkdir -p ${pkgdir}/usr/lib/cups/backend
    ln -sf /usr/bin/smbspool ${pkgdir}/usr/lib/cups/backend/smb
}

package_samba() {
pkgdesc="SMB Fileserver and AD Domain server"
depends=('db>=4.7' 'popt' 'libcups' 'libcap>=2.16' 'gamin' 'gnutls>=2.4.1'
         'talloc' 'tdb' 'libgcrypt' 'python2' 'smbclient>=4.0.2-1')
replaces=('samba4')
conflicts=('samba4')
backup=('etc/samba/smb.conf')

    # Use samba-pkg as a staging directory for the split packages
    # (This is so RPATHS and symlinks are generated correctly via
    # make install, but the otherwise unsplit pieces can be split)
    _pkgsrc=${srcdir}/samba-pkg

    # Everything that libwbclient and smbclient didn't install goes
    # into the samba package...
    mv ${_pkgsrc}/* ${pkgdir}/
    rmdir ${_pkgsrc}

    _pyver=`python2 -c 'import sys; print(sys.version[:3])'`

    find ${pkgdir}/usr/lib/python${_pyver}/site-packages/ -name '*.py' | \
         xargs sed -i "s|#!/usr/bin/env python$|#!/usr/bin/env python2|"
    find ${pkgdir}/usr/bin ${pkgdir}/usr/sbin -type f -executable | \
         xargs sed -i "s|#!/usr/bin/env python$|#!/usr/bin/env python2|"

    # Make admin scripts look in the right place for the samba python module
    for script in sbin/samba_dnsupdate sbin/samba_kcc sbin/samba_spnupdate \
                  sbin/samba_upgradeprovision sbin/samba_upgradedns bin/samba-tool
    do
        sed -i "/^sys\.path\.insert/ a\
sys.path.insert(0, '/usr/lib/python${_pyver}/site-packages')" \
               ${pkgdir}/usr/${script}
    done

    install -d -m755 ${pkgdir}/usr/lib/systemd/system
    for sd in samba smb nmb winbind; do
        install -m644 ${srcdir}/samba-${_realver}/packaging/systemd/${sd}.service ${pkgdir}/usr/lib/systemd/system/${sd}.service
    done

    install -d -m755 ${pkgdir}/etc/samba
    install -m644 ${srcdir}/samba-${_realver}/packaging/LSB/smb.conf ${pkgdir}/etc/samba/smb.conf

    install -d -m755 ${pkgdir}/usr/lib/tmpfiles.d
    echo "d /var/run/samba 0700 root root" > ${pkgdir}/usr/lib/tmpfiles.d/samba.conf
}

# Hack for AUR
pkgdesc="Samba 4.0 File/Domain server and client utilities"
