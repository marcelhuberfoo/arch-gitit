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
depends=('sh' 'icu>=52' 'icu<=54' 'ghc' 'cabal-install>=1.20')
makedepends=('parallel')
optdepends=('texlive-most: for pdf creation')
options=(strip staticlibs !makeflags !distcc !emptydirs)
source=("$pkgname"::"git+https://github.com/jgm/gitit.git#tag=$pkgver")
#install=$pkgname.install
sha256sums=('SKIP')

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

_cabal_verbose=--verbose
_builddir=
_confdir=
_tmpdir=
_pkgwithver=
_tmppackages=
_tmpbindir=
_cabaldir=
_depends=()

_setupDirs() {
  _builddir=$srcdir/build
  _confdir=$_builddir/conf
  _tmpdir=$_builddir/tmp
  _pkgwithver=$pkgname-$pkgver
  _tmppackages=$_builddir/pkg
  _tmpbindir=$_tmppackages/usr/lib/$_pkgwithver/bin
  _cabaldir=$_builddir/.cabal
  mkdir -p $_confdir $_tmpdir $_tmppackages
}

_setupEnv() {
  HOME=$_builddir; TMPDIR=$_tmpdir; PATH=$_tmpbindir:$PATH
}

# $@ contains list of packages to get dependencies for
getdepends() {
  pushd "$srcdir/$pkgname" >/dev/null
  cabal install --dry-run $@ 2>/dev/null | grep "\-[0-9]\+" | cut -d' ' -f1 | tr '\n' ':'
  popd >/dev/null
}

prepare() {
  _setupDirs
  _setupEnv
  cabal update $_cabal_verbose
  _depends[0]=$(getdepends alex happy):"embed_data_files"
  _depends[1]=$(getdepends .):"embed_data_files plugins"
  for _hspackages in "${_depends[@]}"; do
    local _flags="${_hspackages##*:}"
    local _packages=$(echo ${_hspackages%:*} | tr ':' ' ')
    msg2 "Extracting packages"
    cabal fetch $_packages
    echo $_packages | tr ' ' '\n' | parallel --no-notice --no-run-if-empty --bar "cd $_builddir && find $_cabaldir -name {}.tar.gz -exec tar xzf \{\} \;"
  done
}

build() {
  for _hspackages in "${_depends[@]}"; do
    local _flags="${_hspackages##*:}"
    local _packages=$(echo ${_hspackages%:*} | tr ':' ' ')
    for _hpkg in $_packages; do
        if [ ! -d "$_builddir/$_hpkg" ]; then echo "Package $_hpkg not found, skipping"; continue; fi
        pushd $_builddir/$_hpkg >/dev/null
        msg2 "Building $_hpkg $_nameonly"
        cabal configure \
          --flags="$_flags" \
          --prefix=/usr/lib/$_pkgwithver \
          --datadir=/usr/share/$_pkgwithver \
          --docdir=\$datadir/doc/\$pkgid \
          --datasubdir=data/\$pkgid \
          --libsubdir=\$pkgid
        cabal build >/dev/null;
        cabal register --gen-pkg-config >/dev/null
        [ -r "$_hpkg.conf" ] && cp $_hpkg.conf $_confdir;
        cabal register --inplace >/dev/null
        cabal copy --destdir=$_tmppackages
        popd >/dev/null
    done
  done
}

package() {
  _setupDirs
  local _licensedstdir=$pkgdir/usr/share/licenses/$_pkgwithver
  local _licensesrcdir=$_tmppackages/usr/share/$_pkgwithver/doc
  mkdir -p $_licensedstdir
  msg2 "Copying licenses..."
  ( cd $_licensesrcdir && find . -maxdepth 2 -name 'LICENSE' | parallel --no-run-if-empty --no-notice --bar "install -Dm444 $_licensesrcdir/{} $_licensedstdir/{}" )
  msg2 "Copying register configs..."
  ( cd $_confdir && find . -name '*.conf' | parallel --no-run-if-empty --no-notice --bar "install -Dm444 $_confdir/{} $pkgdir/usr/lib/$_pkgwithver/conf/{}" )
  msg2 "Copying docs..."
  ( cd $_tmppackages && tar cf - --exclude='*/LICENSE' usr/share/$_pkgwithver/doc ) | ( cd $pkgdir && tar xf - )
  msg2 "Copying libs..."
  ( cd $_tmppackages && tar cf - --exclude='*/LICENSE' usr/lib/$_pkgwithver/lib ) | ( cd $pkgdir && tar xf - )
  msg2 "Copying bins..."
  ( cd $_tmppackages && tar cf - --exclude='*/LICENSE' usr/lib/$_pkgwithver/bin ) | ( cd $pkgdir && tar xf - )
}

# vim: set ft=sh syn=sh ts=2 sw=2 et:
