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
makedepends=()
optdepends=('texlive-most: for pdf creation')
options=(strip staticlibs !makeflags !distcc !emptydirs)
source=("$pkgname"::"git+https://github.com/jgm/gitit.git#tag=$pkgver")
#install=$pkgname.install
sha256sums=('SKIP')
#_cabal_verbose=--verbose
_cabal_dosandbox=0
_cabal_sandboxdir=$HOME/tmp/.cabal_sandbox_$pkgname

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

_cabal_cmd="HOME=\$srcdir/build; cabal"
_depends=()

# $@ contains list of packages to get dependencies for
getdepends() {
  local _cmd="$_cabal_cmd install --dry-run $@ | grep \"\-[0-9]\+\" | cut -d' ' -f1 | tr '\n' ':'"
  local _dependencies="$(cd $srcdir/$pkgname && eval $_cmd 2>/dev/null)"
  echo "$_dependencies"
}

prepare() {
  eval "$_cabal_cmd update"
  _depends[0]=$(getdepends alex happy):"embed_data_files"
#  _depends[1]=$(getdepends .):"embed_data_files plugins"
  for _hspackages in "${_depends[@]}"; do
    local _flags="${_hspackages##*:}"
    local _packages=$(echo ${_hspackages%:*} | tr ':' ' ')
    eval "$_cabal_cmd fetch $_packages"
    echo $_packages | tr ' ' '\n' | parallel --no-notice --no-run-if-empty --bar "cd $srcdir/build && find $_cabal_buildlocal -name {}.tar.gz -exec tar xzf \{\} \;"
  done
}

build() {
  local _builddir=$srcdir/build
  local _confdir=$_builddir/conf
  local _tmpdir=$_builddir/tmp
  mkdir -p $_confdir $_tmpdir
  for _hspackages in "${_depends[@]}"; do
    local _flags="${_hspackages##*:}"
    local _packages=$(echo ${_hspackages%:*} | tr ':' ' ')
    echo "pkg [$_packages]"
    local _pkgwithver=$pkgname-$pkgver
    local _cmd="echo $_packages | tr ' ' '\n' | parallel --no-notice --no-run-if-empty --jobs=1 --bar \"cd $_builddir/{} && HOME=$_builddir; TMPDIR=$_tmpdir; cabal configure --verbose --flags=\\\"$_flags\\\" --prefix=/usr/lib/$_pkgwithver --datadir=/usr/share/$_pkgwithver --docdir=\\\$datadir/doc/\\\$pkgid --datasubdir=data/\\\$pkgid --libsubdir=\\\$compiler/\\\$pkgid;\"" 
#    local _cmd="cd $_builddir/{} && $_cabal_cmd configure --verbose --flags=\"$_flags\" --prefix=/usr/lib/$_pkgwithver --datadir=/usr/share/$_pkgwithver --docdir=\\\$datadir/doc/\\\$pkgid --datasubdir=data/\\\$pkgid --libsubdir=\\\$compiler/\\\$pkgid ;" 
#   "$_cabal_cmd build ; $_cabal_cmd register --gen-pkg-config; cp {}.conf $_confdir; $_cabal_cmd register --inplace;"
#   "$_cabal_cmd copy --destdir=$pkgdir"
    echo "cmd [$_cmd]"
    eval "$_cmd"
#    echo $_packages | tr ' ' '\n' | parallel --no-notice --no-run-if-empty --jobs=1 --bar $_cmd
  done
  false
}

prepold() {
  cd "$srcdir"
  local _builddir=$srcdir/
  local _prefixdir=$srcdir/build
  local _libdir=$_prefixdir/usr/lib
  local _pkgsrc="$srcdir/$pkgname"
  mkdir -p $_prefixdir
  while read _hpkg; do
    echo "entry [$_hpkg]"
    [ -d "$_libdir/$_hpkg" ] && continue
    local _curpkgdir=$srcdir/$_hpkg
    pushd $_curpkgdir >/dev/null
    msg2 "Fetching $_hpkg"
    case $_hpkg in
      $pkgname-$pkgver)
        HOME=$_builddir cabal configure \
          --flags="embed_data_files plugins" \
          --prefix=/usr \
          --libdir=$_libddir \
          --verbose
        HOME=$_builddir cabal build
        HOME=$_builddir cabal register --inplace
      ;;

      *)
        HOME=$_builddir \
        cabal install --prefix=$_builddir/usr --flags="embed_data_files"
      ;;
    esac
    popd >/dev/null
  done < <(cd "$srcdir/$pkgname" && cabal install --dependencies-only --dry-run 2>/dev/null | grep "\-[0-9]\+" | cut -d' ' -f1 )
}

buildY() {
  cd "$srcdir/$pkgname"
  local _builddir=$srcdir/build
  local _libdir=$_builddir/usr/lib
  local _pkgsrc="$srcdir/$pkgname"
  while read _hpkg; do
    echo "entry [$_hpkg]"
    continue
    [ -d "$_libdir/$_hpkg" ] && continue
    pushd $_pkgsrc/$_hkpkg >/dev/null
    msg2 "Building $_hkpkg"
    case $_hkpkg in
      $pkgname-$pkgver)
        HOME=$_pkgsrc cabal configure --prefix=/usr --libdir=$_libddir --verbose
        HOME=$_pkgsrc cabal build
        HOME=$_pkgsrc cabal register --inplace
      ;;

      *)
        HOME=$_pkgsrc \
        cabal install --prefix=$_builddir/usr --flags="embed_data_files"
      ;;
    esac
    popd >/dev/null
  done < <(cabal install --flags="embed_data_files" --only-dependencies --dry-run 2>/dev/null | grep "\-[0-9]\+" )
}

buildX() {
  cd "$srcdir/$pkgname"
  [ -n "$TMPDIR" ] && mkdir -p "$TMPDIR"
  local sandboxdir=""
  if [ $_cabal_dosandbox ]; then
    [ -n "$_cabal_sandboxdir" ] && sandboxdir="--sandbox=$_cabal_sandboxdir"
    cabal sandbox $_cabal_verbose $sandboxdir init
  fi
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
#  msg2 "Copying license..."
#  install -Dm444 $pkgdir/usr/share/doc/$pkgname-$pkgver/LICENSE ${pkgdir}/usr/share/licenses/$pkgname/LICENSE
#  rm -f $pkgdir/usr/share/doc/$pkgname-$pkgver/LICENSE
}

# vim: set ft=sh syn=sh ts=2 sw=2 et:
