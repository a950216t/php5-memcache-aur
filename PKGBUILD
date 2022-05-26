# Maintainer: tjbp (archlinux@tjbp.net)
# Contributor: Thore Bödecker <me [at] foxxx0 [dot] de>
# Contributor: Evangelos Foutras <evangelos@foutrelis.com>

pkgname=php5-memcache
_pkgbase="${pkgname#php5-}"
pkgver=3.0.8
pkgrel=5
_commit=fdbd46bbc6f53ed6e024521895e142cbfc9b3340
pkgdesc="Memcache module for php5"
arch=('i686' 'x86_64')
url="https://pecl.php.net/package/memcache"
license=('PHP')
provides=("php-memcache=${pkgver}-${pkgrel}")
depends=('php5>=5.6.17-3')
makedepends=('git')
checkdepends=('memcached')
backup=('etc/php5/conf.d/memcache.ini')
#source=(https://pecl.php.net/get/memcache-${pkgver}.tgz)
source=("git+https://github.com/websupport-sk/pecl-memcache.git#commit=$_commit")
sha512sums=('SKIP')

CFLAGS+=' -std=gnu89'

prepare() {
  cd "${srcdir}/pecl-memcache"

  # Disable UDP tests
  sed -i "s|^\(\$udpPort2\? =\) .*|\1 0;|" tests/connect.inc

  # Remove flaky tests
  #   [tests/040.phpt] memcache->increment()/decrement() with multiple keys
  #   [tests/042.phpt] memcache->set() with multiple values
  #   strange keys [tests/005.phpt]
  #   ini_set('session.save_handler') [tests/036.phpt]
  #   ini_set('memcache.session_redundancy') [tests/044.phpt]
  #   ini_set('session.save_handler') with unix domain socket [tests/053.phpt]
  #   session locking [tests/057.phpt]
  rm tests/{005,036,040,042,044,053,057,100b}.phpt
}

build() {
    cd "${srcdir}/pecl-memcache"

    phpize5
    ./configure \
        --config-cache \
        --sysconfdir=/etc/php5 \
        --with-php-config=/usr/bin/php-config5 \
        --localstatedir=/var
    make
}

check() {
    cd "${srcdir}/pecl-memcache"
    sed -i "s|^\(\$domainsocket =\) .*|\1 'unix://$PWD/memcached.sock';|" \
      tests/connect.inc

    memcached_pids=()
    memcached -p 11211 -U 11211 >/dev/null        & memcached_pids+=($!)
    memcached -p 11212 -U 11212 >/dev/null        & memcached_pids+=($!)
    memcached -s "$PWD/memcached.sock" >/dev/null & memcached_pids+=($!)

    local ret=0
    make test NO_INTERACTION=1 REPORT_EXIT_STATUS=1 || ret=1

    kill ${memcached_pids[@]}

    return $ret
}

package() {
    cd "${srcdir}/pecl-memcache"

    make INSTALL_ROOT="$pkgdir" install
    echo ';extension=memcache.so' >memcache.ini
    install -Dm644 memcache.ini "$pkgdir/etc/php5/conf.d/memcache.ini"
}
