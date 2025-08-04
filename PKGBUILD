# Maintainer: Yun Dou <dixyes@gmail.com>

_name=swow
_upstream=swow/swow
pkgbase=php-swow
pkgname=('php-swow' 'php-legacy-swow')
pkgver=1.6.0
pkgrel=0
pkgdesc="Swow coroutine IO extension for PHP"
arch=('x86_64' 'arm64')
url="https://github.com/swow/swow"
license=('APACHE')
makedepends=('php' 'php-legacy' 'postgresql-libs')
depends=('glibc')
source=(
    "$_name-$pkgver.tar.gz::https://github.com/${_upstream}/archive/refs/tags/v${pkgver}.tar.gz"
    "0001-Fix_PQclosePrepared_weak_dep.patch::https://github.com/swow/swow/commit/bb348b9da8da1cd501c9d916be62806950092009.patch"
)
sha512sums=('f92578675425fbf8fa15716e7ce858f5a61f03ee8fe7274e59078f05bc7a3842852267ae061e02c943d5776e04aff605c1dc8f404881e51661e96207fcfbbe8a'
            'd5e66be35877c1b76cdeaf4f2a5603c5a48de994952d9e9f6933014b025a24457800a4737003a7a509f963336903e08c3c74f42f9f6b61b54c2c55ac9198f698')

prepare() {
    mv -v "${_name}-${pkgver}" "$pkgbase-$pkgver"

    echo -e "; the Swow extension requires curl to be activated first\n; and it's conflict with swoole\nextension=${_name}" > "$pkgbase-$pkgver/ext/${_name}.ini"

    cp -av "$pkgbase-$pkgver" "${pkgname[1]}-$pkgver"
    for p in $(ls ${srcdir}/*.patch); do
        patch --directory="$pkgbase-$pkgver" --forward --strip=1 --input="${p}" ;
    done ;

    (
        cd "$pkgbase-$pkgver/ext"
        phpize
    )

    (
        cd "${pkgname[1]}-$pkgver/ext"
        phpize-legacy
    )
}

build() {
    (
        cd "$pkgbase-$pkgver/ext"
        ./configure \
            --prefix=/usr \
            --enable-swow-ssl \
            --enable-swow-curl
        make
    )

    (
        cd "${pkgname[1]}-$pkgver/ext"
        ./configure \
            --prefix=/usr \
            --enable-swow-ssl \
            --enable-swow-curl
        make
    )
}

check() {
    local EXTRA_PHPT_ARGS="-n --show-diff"
    (
        /usr/bin/php -d extension=${srcdir}/${pkgbase}-${pkgver}/ext/modules/swow.so --ri swow
        export TEST_PHP_EXECUTABLE=/usr/bin/php
        local TEST_PHP_ARGS="${EXTRA_PHPT_ARGS} -d extension=${srcdir}/${pkgbase}-${pkgver}/ext/modules/swow.so"
        cd "$pkgbase-$pkgver/ext"
        $TEST_PHP_EXECUTABLE run-tests.php . $TEST_PHP_ARGS
    )

    (
        /usr/bin/php-legacy -d extension=${srcdir}/${pkgname[1]}-${pkgver}/ext/modules/swow.so --ri swow
        export TEST_PHP_EXECUTABLE=/usr/bin/php-legacy
        local TEST_PHP_ARGS="${EXTRA_PHPT_ARGS} -d extension=${srcdir}/${pkgname[1]}-${pkgver}/ext/modules/swow.so"
        cd "${pkgname[1]}-$pkgver/ext"
        $TEST_PHP_EXECUTABLE run-tests.php . $TEST_PHP_ARGS
    )
}

package_php-swow() {
    backup=("etc/php/conf.d/${_name}.ini")
    depends+=('php')
    provides+=('php-swow')
    conflicts+=('php-swow')

    cd "$pkgbase-$pkgver/ext"
    make INSTALL_ROOT="$pkgdir/" install
    install -vDm 644 "${_name}.ini" -t "${pkgdir}/etc/php/conf.d/"
}

package_php-legacy-swow() {
    backup=("etc/php-legacy/conf.d/${_name}.ini")
    depends+=('php-legacy')
    provides+=('php-legacy-swow')
    conflicts+=('php-legacy-swow')

    cd "${pkgname[1]}-$pkgver/ext"
    make INSTALL_ROOT="$pkgdir/" install
    install -vDm 644 "${_name}.ini" -t "${pkgdir}/etc/php-legacy/conf.d/"
}
