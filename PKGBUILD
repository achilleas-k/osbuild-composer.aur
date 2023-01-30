# Maintainer: Achilleas Koutsou <achilleas@koutsou.net>

pkgname=osbuild-composer
pkgdesc='An HTTP service for building bootable OS images'
pkgver=73
pkgrel=2
url="https://www.osbuild.org"
arch=(x86_64)
license=(Apache)
depends=('dnf' 'qemu' 'osbuild' 'systemd')
makedepends=('go' 'systemd')
optdepends=()
source=($pkgname-$pkgver.tar.gz::https://github.com/osbuild/osbuild-composer/archive/refs/tags/v${pkgver}.tar.gz)
sha256sums=('2ab90bb658cf95cf92171ae2f308b9df79a55eb0dcda785346f0ab9eb8111614')

prepare() {
  cd $pkgname-$pkgver
  mkdir -p build

  # Arch doesn't use /usr/libexec: edit service files
  sed -i 's,/usr/libexec,/usr/lib,g' distribution/osbuild-*.service

  # configure path to dnf-json
  cat > osbuild-composer.toml << EOF
dnf-json = "/usr/lib/osbuild-composer/dnf-json"
EOF
}

build() {
  cd $pkgname-$pkgver
  export LDFLAGS="-ldflags=-X=github.com/osbuild/osbuild-composer/internal/common.RpmVersion=${pkgname}-${pkgver}-${pkgrel}.${arch}"
  export GOFLAGS="-buildmode=pie -trimpath -mod=vendor -modcacherw ${LDFLAGS}"

  go build -o build/osbuild-composer ./cmd/osbuild-composer
  go build -o build/osbuild-worker ./cmd/osbuild-worker
}

package() {
  cd $pkgname-$pkgver

  # binaries
  install -Dm755 "build/osbuild-composer" "${pkgdir}/usr/lib/osbuild-composer/osbuild-composer"
  install -Dm755 "build/osbuild-worker" "${pkgdir}/usr/lib/osbuild-composer/osbuild-worker"
  install -Dm755 "dnf-json"               "${pkgdir}/usr/lib/osbuild-composer/dnf-json"

  # sysusers
  install -Dm644 distribution/osbuild-composer.conf "${pkgdir}/usr/lib/sysusers.d/osbuild-composer.conf"

  # systemd units
  mkdir -p "${pkgdir}/usr/lib/systemd/system"
  install -Dm644 distribution/*.service "${pkgdir}/usr/lib/systemd/system/"
  install -Dm644 distribution/*.socket "${pkgdir}/usr/lib/systemd/system/"

  # repositories
  mkdir -p "${pkgdir}/usr/share/osbuild-composer/repositories"
  install -Dm644 repositories/*.json "${pkgdir}/usr/share/osbuild-composer/repositories"

  # config file
  mkdir -p "${pkgdir}/etc/osbuild-composer/"
  install -Dm 644 osbuild-composer.toml "${pkgdir}/etc/osbuild-composer/osbuild-composer.toml"

  # license
  install -Dm644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
