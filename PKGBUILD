# Contributor: Joop Kiefte <ikojba@gmail.com>
# Maintainer: Marcel Huber <`echo "moc tknup liamg tä oofrebuhlecram" | rev`>

pkgname=gitit
pkgver=0.10.4
pkgrel=1
pkgdesc="Wiki using happstack, git or darcs, and pandoc."
url="https://github.com/jgm/gitit"
license=('GPL')
arch=('i686' 'x86_64')
# Needed for pandoc-citeproc
depends=('icu>=52' 'icu<=54')
makedepends=('ghc' 'sh' 'cabal-install' 'alex' 'happy')
#optdepends=('texlive-most: for pdf creation')
options=(strip staticlibs !makeflags !distcc !emptydirs)
source=("$pkgname"::"git+https://github.com/jgm/gitit.git#tag=$pkgver")
install=${pkgname}.install
sha256sums=('SKIP')
#_my_verbose_=--verbose

#pkgver() {
#  cd "$srcdir/$pkgname"
#  if GITTAG="$(git describe --abbrev=0 --tags 2>/dev/null)"; then
#    local _revs_ahead_tag=$(git rev-list --count ${GITTAG}..)
#    local _commit_id_short=$(git log -1 --format=%h)
#    echo $(sed -e s/^${pkgname%%-git}// -e 's/^[-_/a-zA-Z]\+//' -e 's/[-_+]/./g' <<< ${GITTAG}).${_revs_ahead_tag}.g${_commit_id_short}
#  else
#    echo 0.$(git rev-list --count master).g$(git log -1 --format=%h)
#  fi
#}

prepare() {
  cd "$srcdir/$pkgname"
}

build() {
  cd "$srcdir/$pkgname"
  cabal sandbox $_my_verbose_ init
  cabal update $_my_verbose_
  cabal install $_my_verbose_ --only-dependencies --flags="embed_data_files" hsb2hs .
  cabal configure $_my_verbose_ \
    --flags="embed_data_files" \
    --prefix=/usr \
    --datadir=/usr/share/$pkgname/data \
    --datasubdir= \
    --docdir=/usr/share/doc/$pkgname \
    --libsubdir=$pkgname
  cabal build $_my_verbose_
  cabal register $_my_verbose_ --inplace
}

package() {
  cd "$srcdir/$pkgname"
  cabal copy $_my_verbose_ --destdir=$pkgdir
# For some reason the library is installed anyway
# Remove all files and !emptydirs takes care of the rest
  msg2 "Removing lib files..."
  find ${pkgdir} -iname lib -print0 | xargs -0 rm -vf
#  msg2 "Adjusting license and doc dirs..."
#  mv $pkgdir/usr/share/doc/
#  install -D -m744 register.sh   ${pkgdir}/usr/share/haskell/${pkgname}/register.sh
#  install    -m744 unregister.sh ${pkgdir}/usr/share/haskell/${pkgname}/unregister.sh
#  install -d -m755 ${pkgdir}/usr/share/doc/ghc/html/libraries
#  ln -s /usr/share/doc/${pkgname}/html ${pkgdir}/usr/share/doc/ghc/html/libraries/${_hkgname}
#  runhaskell Setup copy --destdir=${pkgdir}
}

#package() {
#  cd "$srcdir/$pkgname"
#  make DESTDIR="$pkgdir" install
#  mkdir "$pkgdir/usr/lib/systemd/user"
#  cd "$pkgdir/usr/lib/systemd/user"
#  ln -s "$pkdir/usr/lib/systemd/system/envoy@.service" "$pkdir/usr/lib/systemd/system/envoy@.socket" .
#}

# vim: set ft=sh syn=sh ts=2 sw=2 et:
