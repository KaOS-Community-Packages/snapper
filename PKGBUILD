pkgname=snapper
pkgver=0.2.6
pkgrel=1
pkgdesc="A tool for managing BTRFS and LVM snapshots. It can create, diff and restore snapshots and provides timelined auto-snapping."
arch=("x86_64")
url="http://snapper.io/"
license=('GPL2')
depends=('btrfs-progs' 'libxml2' 'dbus' 'boost' 'pam')
makedepends=('lvm2' 'libxslt' 'docbook-xsl' 'git')
backup=('etc/conf.d/snapper')
source=("ftp://ftp.suse.com/pub/projects/$pkgname/$pkgname-$pkgver.tar.bz2")
install=snapper.install
sha512sums=('d49eef086d84f02106691872db6369b10cb55a6afe6f2cfb6c790330321397d66dc1aa0229d31d9e3c7b1efb6bd6c0fd9242dbb69a44fe3433233717df98a334')

prepare() {
  cd "$pkgname-$pkgver"

  ## Build fixes
  # boost fixlets - Arch doesn't use -mt suffix
  sed -e 's@lboost_thread-mt@lboost_thread@g' \
      -e 's@lboost_system-mt@lboost_system@g' \
      -i snapper/Makefile.am

  ## Location/naming fixes
  # fix pam plugin install location
  sed -i -e 's@shell echo /@shell echo /usr/@g' pam/Makefile.am
  # all in /usr/bin
  sed -i -e 's@/usr/sbin@/usr/bin@g' data/org.opensuse.Snapper.service
  # NTP drift file location
  sed -i -e 's@/var/lib/ntp/drift/ntp.drift@/var/lib/ntp/ntp.drift@' \
    data/base.txt
  # man pages sysconfig location
  sed -i -e 's@/etc/sysconfig@/etc/conf.d@g' doc/*

  # systemd timer stuff (>= systemd-212)
  sed ':a;N;$!ba;s/\[Timer\]\nOnCalendar=hourly\n\n/\[Timer\]\nOnCalendar=hourly\nPersistent=true\n\n/g' \
    -i data/timeline.timer
  sed '/cron./d' -i scripts/Makefile.am
  sed -e 's@noinst_PROGRAMS@libexec_PROGRAMS@g' -i client/Makefile.am
}

build() {
    cd "$pkgname-$pkgver"
    
  aclocal
  libtoolize --force --automake --copy
  autoheader
  automake --add-missing --copy
  autoconf
  ./configure --prefix=/usr \
              --sbindir=/usr/bin \
              --libexecdir=/usr/lib/snapper \
              --with-conf=/etc/conf.d \
              --disable-zypp \
              --disable-silent-rules \
              --disable-ext4 \
              --enable-xattrs
  make
}

package() {
    cd "$pkgname-$pkgver"
    make DESTDIR="${pkgdir}" install
  install -Dm644 data/sysconfig.snapper "${pkgdir}"/etc/conf.d/snapper

  # systemd timer units
  install -Dm644 data/cleanup.service "${pkgdir}"/usr/lib/systemd/system/snapper-cleanup.service
  install -Dm644 data/cleanup.timer "${pkgdir}"/usr/lib/systemd/system/snapper-cleanup.timer
  install -Dm644 data/timeline.service "${pkgdir}"/usr/lib/systemd/system/snapper-timeline.service
  install -Dm644 data/timeline.timer "${pkgdir}"/usr/lib/systemd/system/snapper-timeline.timer
}

# vim:set ts=2 sw=2 et:
