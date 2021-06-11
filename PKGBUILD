# Maintainer: Levente Polyak <anthraxx[at]archlinux[dot]org>
# Maintainer: Guillaume ALAUX <guillaume@archlinux.org>

# TODO add test, see about packaging jtreg and using it here

pkgbase=java-openjdk16
pkgname=('jre-openjdk16-headless' 'jre-openjdk16' 'jdk-openjdk16' 'openjdk16-src' 'openjdk16-doc')
_majorver=16
_minorver=0
_securityver=1
_updatever=9
pkgrel=1
pkgver=${_majorver}.${_minorver}.${_securityver}.u${_updatever}
arch=('x86_64')
url='https://openjdk.java.net/'
license=('custom')
makedepends=('java-environment>=11' 'cpio' 'unzip' 'zip' 'libelf' 'libcups' 'libx11'
             'libxrender' 'libxtst' 'libxt' 'libxext' 'libxrandr' 'alsa-lib' 'pandoc'
             'graphviz' 'freetype2' 'libjpeg-turbo' 'giflib' 'libpng' 'lcms2'
             'libnet' 'bash' 'harfbuzz' 'gcc-libs' 'glibc')
source=(https://github.com/openjdk/jdk${_majorver}u/archive/refs/tags/jdk-${_majorver}.${_minorver}.${_securityver}+${_updatever}.tar.gz
        freedesktop-java.desktop
        freedesktop-jconsole.desktop
        freedesktop-jshell.desktop
        zListInline.patch)
sha256sums=('3b3aca61fe2834322e51636d99bcd09c4abba7d8a2cde231dbcbcbdedf829baf'
            '11150daae08fff25309dcc060c07cd056181644d2b563c22668b98e306e8f2db'
            'c0673d680cc728af88bec89275f0a8e2199d52e48b5c5b0aa2d921bedbc2e12f'
            '0cdf3dc10a64c5e589e655276189422b60a6b3294539164a6f29123d6b2046c7'
            '297cd5926d1b9f89c588f4bdc1bfd9554a0e1a3502bd04a7a2061f027794ede6')

case "${CARCH}" in
  x86_64) _JARCH='x86_64';;
  i686)   _JARCH='x86';;
esac

_jvmdir=/usr/lib/jvm/java-${_majorver}-openjdk
_jdkdir=jdk${_majorver}u-jdk-${_majorver}.${_minorver}.${_securityver}-${_updatever}
_imgdir=${_jdkdir}/build/linux-${_JARCH}-server-release/images

_nonheadless=(lib/libawt_xawt.{so,debuginfo}
              lib/libjawt.{so,debuginfo}
              lib/libjsound.{so,debuginfo}
              lib/libsplashscreen.{so,debuginfo})

prepare() {
  cd ${_jdkdir}
  patch --forward --strip=1 --input=${srcdir}/zListInline.patch
}

build() {
  cd ${_jdkdir}

  NUM_PROC_OPT=''
  MAKEFLAG_J=$(echo ${MAKEFLAGS} | sed -En 's/.*-j([0-9]+).*/\1/p')
  if [ -n "${MAKEFLAG_J}" ]; then
    # http://hg.openjdk.java.net/jdk10/jdk10/file/85e6cb013b98/make/InitSupport.gmk#l105
    echo "Removing '-j${MAKEFLAG_J}' from MAKEFLAGS to prevent build fail. Passing it directly to ./configure."
    export MAKEFLAGS=${MAKEFLAGS/-j${MAKEFLAG_J}/}
    NUM_PROC_OPT="--with-num-cores=${MAKEFLAG_J}"
  fi

  # Avoid optimization of HotSpot to be lowered from O3 to O2
  local _CFLAGS="${CFLAGS//-O2/-O3} ${CPPFLAGS} -fcommon"
  local _CXXFLAGS="${CXXFLAGS//-O2/-O3} ${CPPFLAGS} -fcommon"
  local _LDFLAGS=${LDFLAGS}
  if [[ ${CARCH} = i686 ]]; then
    echo "Removing '-fno-plt' from CFLAGS and CXXFLAGS to prevent build fail with this architecture"
    _CFLAGS=${CFLAGS/-fno-plt/}
    _CXXFLAGS=${CXXFLAGS/-fno-plt/}
  fi

  # CFLAGS, CXXFLAGS and LDFLAGS are ignored as shown by a warning
  # in the output of ./configure unless used like such:
  #  --with-extra-cflags="${CFLAGS}"
  #  --with-extra-cxxflags="${CXXFLAGS}"
  #  --with-extra-ldflags="${LDFLAGS}"
  # See also paragraph "Configure Control Variables from "jdk${_majorver}-${_hg_tag}/common/doc/building.md
  unset CFLAGS
  unset CXXFLAGS
  unset LDFLAGS

  bash configure \
    --with-version-build="${_updatever}" \
    --with-version-pre="" \
    --with-version-opt="" \
    --with-stdc++lib=dynamic \
    --with-extra-cflags="${_CFLAGS}" \
    --with-extra-cxxflags="${_CXXFLAGS}" \
    --with-extra-ldflags="${_LDFLAGS}" \
    --with-libjpeg=system \
    --with-giflib=system \
    --with-libpng=system \
    --with-lcms=system \
    --with-zlib=system \
    --with-jvm-features=zgc \
    --enable-unlimited-crypto \
    --disable-warnings-as-errors \
    ${NUM_PROC_OPT}
    #--disable-javac-server

  make images legacy-jre-image docs

  # https://bugs.openjdk.java.net/browse/JDK-8173610
  find "../${_imgdir}" -iname '*.so' -exec chmod +x {} \;
}

check() {
  cd ${_jdkdir}
  # TODO package jtreg
  # make -k check
}

package_jre-openjdk16-headless() {
  pkgdesc="OpenJDK Java ${_majorver} headless runtime environment"
  depends=('java-runtime-common>=3' 'ca-certificates-utils' 'nss' 'libjpeg-turbo' 'libjpeg.so'
           'lcms2' 'liblcms2.so' 'libnet' 'freetype2' 'libfreetype.so' 'harfbuzz' 'libharfbuzz.so=0'
           'glibc' 'gcc-libs')
  optdepends=('java-rhino: for some JavaScript support')
  provides=("java-runtime-headless=${_majorver}" "java-runtime-headless-openjdk=${_majorver}" "jre${_majorver}-openjdk-headless=${pkgver}-${pkgrel}")
  backup=(etc/${pkgbase}/logging.properties
          etc/${pkgbase}/management/jmxremote.access
          etc/${pkgbase}/management/jmxremote.password.template
          etc/${pkgbase}/management/management.properties
          etc/${pkgbase}/net.properties
          etc/${pkgbase}/security/java.policy
          etc/${pkgbase}/security/java.security
          etc/${pkgbase}/security/policy/README.txt
          etc/${pkgbase}/security/policy/limited/default_US_export.policy
          etc/${pkgbase}/security/policy/limited/default_local.policy
          etc/${pkgbase}/security/policy/limited/exempt_local.policy
          etc/${pkgbase}/security/policy/unlimited/default_US_export.policy
          etc/${pkgbase}/security/policy/unlimited/default_local.policy
          etc/${pkgbase}/sound.properties)
  install=install_jre-openjdk-headless.sh

  cd ${_imgdir}/jre

  install -dm 755 "${pkgdir}${_jvmdir}"

  cp -a bin lib \
    "${pkgdir}${_jvmdir}"

  for f in "${_nonheadless[@]}"; do
    rm "${pkgdir}${_jvmdir}/${f}"
  done

  cp ../jdk/release "${pkgdir}${_jvmdir}"
  cp ../jdk/lib/modules "${pkgdir}${_jvmdir}/lib"

  # Conf
  install -dm 755 "${pkgdir}/etc"
  cp -r conf "${pkgdir}/etc/${pkgbase}"
  ln -s /etc/${pkgbase} "${pkgdir}/${_jvmdir}/conf"

  # Legal
  install -dm 755 "${pkgdir}/usr/share/licenses"
  cp -r legal "${pkgdir}/usr/share/licenses/${pkgbase}"
  ln -s ${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"
  ln -s /usr/share/licenses/${pkgbase} "${pkgdir}/${_jvmdir}/legal"

  # Man pages
  for f in bin/*; do
    f=$(basename "${f}")
    _man=../jdk/man/man1/"${f}.1"
    test -f "${_man}" && install -Dm 644 "${_man}" "${pkgdir}/usr/share/man/man1/${f}-openjdk${_majorver}.1"
  done
  ln -s /usr/share/man "${pkgdir}/${_jvmdir}/man"

  # Link JKS keystore from ca-certificates-utils
  rm -f "${pkgdir}${_jvmdir}/lib/security/cacerts"
  ln -sf /etc/ssl/certs/java/cacerts "${pkgdir}${_jvmdir}/lib/security/cacerts"
}

package_jre-openjdk16() {
  pkgdesc="OpenJDK Java ${_majorver} full runtime environment"
  depends=("jre${_majorver}-openjdk-headless=${pkgver}-${pkgrel}" 'giflib' 'libgif.so'
           'glibc' 'gcc-libs' 'libpng')
  optdepends=('alsa-lib: for basic sound support'
              'gtk2: for the Gtk+ 2 look and feel - desktop usage'
              'gtk3: for the Gtk+ 3 look and feel - desktop usage')
  provides=("java-runtime=${_majorver}" "java-runtime-openjdk=${_majorver}" "jre${_majorver}-openjdk=${pkgver}-${pkgrel}")
  install=install_jre-openjdk.sh

  cd ${_imgdir}/jre

  install -dm 755 "${pkgdir}${_jvmdir}"

  for f in "${_nonheadless[@]}"; do
    install -Dm 644 ${f} "${pkgdir}${_jvmdir}/${f}"
  done

  # Licenses
  install -dm 755 "${pkgdir}/usr/share/licenses"
  ln -s ${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"
}

package_jdk-openjdk16() {
  pkgdesc="OpenJDK Java ${_majorver} development kit"
  depends=("jre${_majorver}-openjdk=${pkgver}-${pkgrel}" 'java-environment-common=3'
           'hicolor-icon-theme' 'libelf' 'glibc' 'gcc-libs')
  provides=("java-environment=${_majorver}" "java-environment-openjdk=${_majorver}" "jdk${_majorver}-openjdk=${pkgver}-${pkgrel}")
  install=install_jdk-openjdk.sh

  cd ${_imgdir}/jdk

  install -dm 755 "${pkgdir}${_jvmdir}"

  cp -a bin demo include jmods lib \
    "${pkgdir}${_jvmdir}"

  rm "${pkgdir}${_jvmdir}/lib/src.zip"

  # Remove files held by JRE
  pushd ../jre
  for d in bin lib; do
    find ${d} ! -type d -exec rm "${pkgdir}${_jvmdir}/{}" \;
  done
  popd
  find "${pkgdir}${_jvmdir}/lib" -type d -empty -delete

  # Conf files all belong to JRE

  # Legal
  install -dm 755 "${pkgdir}/usr/share/licenses"
  cp -r legal "${pkgdir}/usr/share/licenses/${pkgbase}"
  pushd ../jre/legal
  find . ! -type d -exec rm "${pkgdir}/usr/share/licenses/${pkgbase}/{}" \;
  popd
  find "${pkgdir}/usr/share/licenses" -type d -empty -delete
  ln -s ${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"

  # Man pages
  for f in bin/*; do
    f=$(basename "${f}")
    _man=man/man1/"${f}.1"
    test -f "../jre/bin/${f}" && continue
    test -f "${_man}" && install -Dm 644 "${_man}" "${pkgdir}/usr/share/man/man1/${f}-openjdk${_majorver}.1"
  done

  # Icons
  for s in 16 24 32 48; do
    install -Dm 644 \
      "${srcdir}/${_jdkdir}/src/java.desktop/unix/classes/sun/awt/X11/java-icon${s}.png" \
      "${pkgdir}/usr/share/icons/hicolor/${s}x${s}/apps/${pkgbase}.png"
  done

  # Desktop files
  for f in jconsole java jshell; do
    install -Dm 644 \
      "${srcdir}/freedesktop-${f}.desktop" \
      "${pkgdir}/usr/share/applications/${f}-${pkgbase}.desktop"
  done
}

package_openjdk16-src() {
  pkgdesc="OpenJDK Java ${_majorver} sources"
  # Depends on JDK to get license files
  depends=("jdk${_majorver}-openjdk=${pkgver}-${pkgrel}")
  provides=("openjdk${_majorver}-src=${pkgver}-${pkgrel}")

  install -Dm 644 -t "${pkgdir}${_jvmdir}/lib" ${_imgdir}/jdk/lib/src.zip

  install -dm 755 "${pkgdir}/usr/share/licenses"
  ln -s ${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"
}

package_openjdk16-doc() {
  pkgdesc="OpenJDK Java ${_majorver} documentation"
  # Depends on JDK to get license files
  depends=("jdk${_majorver}-openjdk=${pkgver}-${pkgrel}")
  provides=("openjdk${_majorver}-doc=${pkgver}-${pkgrel}")

  install -dm 755 "${pkgdir}/usr/share/doc"
  cp -r ${_imgdir}/docs "${pkgdir}/usr/share/doc/${pkgbase}"

  install -dm 755 "${pkgdir}/usr/share/licenses"
  ln -s ${pkgbase} "${pkgdir}/usr/share/licenses/${pkgname}"
}

# vim: ts=2 sw=2 et:
