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
optdepends=('texlive-most: for pdf creation')
options=(strip staticlibs !makeflags !distcc !emptydirs)
source=("$pkgname"::"git+https://github.com/jgm/gitit.git#tag=$pkgver")
#install=$pkgname.install
sha256sums=('SKIP')
#_cabal_verbose=--verbose
_cabal_dosandbox=0
_cabal_sandboxdir=$HOME/tmp/.cabal_sandbox_$pkgname
# in case your /tmp has no executable rights...
export TMPDIR=$HOME/tmp/$pkgname

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
  local _builddir=$srcdir/build
  local _libdir=$_builddir/usr/lib
  mkdir -p $srcdir/{build,$pkgname-$pkgver}
  while read _hpkg; do
    echo "entry [$_hpkg]"
    [ -d "$_libdir" ] && continue
  done <(cabal install --flags="embed_data_files" --only-dependencies --dry-run )
}

build() {
  cd "$srcdir/$pkgname"
  [ -n "$TMPDIR" ] && mkdir -p "$TMPDIR"
  local sandboxdir=""
  if [ $_cabal_dosandbox ]; then
    [ -n "$_cabal_sandboxdir" ] && sandboxdir="--sandbox=$_cabal_sandboxdir"
    cabal sandbox $_cabal_verbose $sandboxdir init
  cabal update $_cabal_verbose
  cabal install $_cabal_verbose \
    --flags="embed_data_files" \
    --only-dependencies
  cabal configure $_cabal_verbose \
    --flags="embed_data_files plugins" \
    --prefix=/usr \
    --datadir=\$prefix/share/\$pkgid \
    --datasubdir= \
    --docdir=\$prefix/share/doc/\$pkgid \
    --libsubdir=\$compiler/\$pkgid
#    --enable-split-objs \
#    --enable-shared \
  cabal build $_cabal_verbose
  cabal haddock $_cabal_verbose
  cabal register $_cabal_verbose --gen-script
}

package() {
  cd "$srcdir/$pkgname"
  cabal copy $_cabal_verbose --destdir=$pkgdir
  msg2 "Copying license..."
  install -Dm544 register.sh ${pkgdir}/usr/share/$pkgname-$pkgver/register.sh
  install -Dm444 $pkgdir/usr/share/doc/$pkgname-$pkgver/LICENSE ${pkgdir}/usr/share/licenses/$pkgname/LICENSE
  rm -f $pkgdir/usr/share/doc/$pkgname-$pkgver/LICENSE
}

# vim: set ft=sh syn=sh ts=2 sw=2 et:
