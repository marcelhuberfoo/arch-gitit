# Contributor: Joop Kiefte <ikojba@gmail.com>
# Maintainer: Marcel Huber <`echo "moc tknup liamg tä oofrebuhlecram" | rev`>

pkgname=gitit
pkgver=0.10.4
pkgrel=1
pkgdesc="Wiki using happstack, git or darcs, and pandoc."
url="https://github.com/jgm/gitit"
license=('GPL')
arch=('i686' 'x86_64')
depends=('sh' 'ghc' 'cabal-install>=1.20')
makedepends=('parallel' 'chrpath')
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
_cabalsandboxdir=
_cabalsandboxbindir=
_cabaldir=
_packageconfdir=
_dependantpackagesfile=
_packagestobuild=("$pkgname" "embed_data_files plugins")

_setupLocalEnvVars() {
  _builddir=$srcdir
  _confdir=$_builddir/package.conf
  _tmpdir=$_builddir/tmp
  _pkgwithver=$pkgname-$pkgver
  _tmppackages=$_builddir/pkg
  _dependantpackagesfile=$_builddir/${_packagestobuild[0]}.depends
  # use another subdir for sandbox to not disturb regular build
  _cabalsandboxdir=$_builddir/sandbox/.cabal-sandbox-$pkgname
  _cabalsandboxbindir=$_cabalsandboxdir/bin
  _cabaldir=$_builddir/.cabal
  _packageconfdir=$pkgdir/usr/lib/$_pkgwithver/package.conf.d
  mkdir -p $_confdir $_tmpdir $_tmppackages $_cabalsandboxdir
  # in case your /tmp directory has no execute perms (needed for package installs)
  export TMPDIR=$_tmpdir;
  [ -n "$TMPDIR" -a ! -d "$TMPDIR" ] && mkdir -p "$TMPDIR"
  # use our own local file storage for cabal instead of real $HOME/.cabal
  export HOME=$_builddir;
  export PATH=$_cabalsandboxbindir:$PATH
}

# $@ contains list of packages to get dependencies for
_getcabaldepends() {
  pushd "$srcdir/$pkgname" >/dev/null
  local _installoptions="$1"
  shift
  local _packiges=$@
  [ "$1" = "$pkgname" ] &&  _packiges=.
  cabal install --dry-run $_installoptions $_packiges 2>/dev/null | grep "\-[0-9]\+" | cut -d' ' -f1
  popd >/dev/null
}

_installBuildHelpers() {
  msg2 "Installing build helpers"
  local _sandboxbase=$(dirname $_cabalsandboxdir)
  pushd "$_sandboxbase" >/dev/null
  cabal sandbox --sandbox=$_cabalsandboxdir init
  cabal update >/dev/null
  cabal install \
    --flags="embed_data_files" \
    happy alex
  popd >/dev/null
}

prepare() {
  _setupLocalEnvVars
  _installBuildHelpers
  cabal update $_cabal_verbose
  local _firstpackage=$(echo ${_packagestobuild[0]} | cut -d' ' -f1)
  local _installopts=""
  [ "$_firstpackage" = "$pkgname" ] && _installopts="--dependencies-only"
  if [ ! -f "$_dependantpackagesfile" ]; then
    echo ${_packagestobuild[1]} >$_dependantpackagesfile
    _getcabaldepends "$_installopts" ${_packagestobuild[0]} >>$_dependantpackagesfile
  fi
  msg2 "Downloading/Extracting packages"
  sed -n '2,$ p' $_dependantpackagesfile | parallel --no-notice --no-run-if-empty --bar "cd $_builddir && cabal fetch {}>/dev/null; find $_cabaldir -name {}.tar.gz -exec tar xzf \{\} \;"
  [ "$_firstpackage" = "$pkgname" -a "$(sed -n '$ p' $_dependantpackagesfile | tr -d '\n')" != "$pkgname" ] && echo "$pkgname" >>$_dependantpackagesfile
}

# arg1: configure options
# arg2: package
_buildPackageWithOpts() {
  local _flags="$1"
  local _hpkg=$2
  if [ ! -d "$_builddir/$_hpkg" ]; then echo "Package $_hpkg not found, skipping"; return; fi
  pushd $_builddir/$_hpkg >/dev/null
  cabal configure \
    --flags="$_flags" \
    --prefix=/usr/lib/$_pkgwithver \
    --datadir=/usr/share/$_pkgwithver \
    --docdir=\$datadir/doc/\$pkgid \
    --datasubdir=data/\$pkgid \
    --libsubdir=\$pkgid
  cabal build; #>/dev/null 2>&1;
  cabal register --gen-pkg-config >/dev/null
  if [ -f "$_hpkg.conf" ]; then
    cp -fp $_hpkg.conf $_confdir
  fi
  cabal register --inplace >/dev/null
  cabal copy --destdir=$_tmppackages
  popd >/dev/null
}

build() {
  local _flags="$(sed -n '1 p' $_dependantpackagesfile | tr -d '\n')"
  export -f _buildPackageWithOpts
  export _builddir _pkgwithver _tmppackages _confdir
  sed -n '2,$ p' $_dependantpackagesfile | parallel --jobs 1 --no-notice --no-run-if-empty --bar "_buildPackageWithOpts \"$(sed -n '1 p' $_dependantpackagesfile)\" {}"
#  sed -n '2,$ p' $_dependantpackagesfile | parallel --no-notice --no-run-if-empty --bar "pushd $_builddir/{} >/dev/null; cabal clean; popd >/dev/null"
}

# arg1: path and name of script
_createWrapperScript() {
  cat <<EOF >$1
#!/bin/sh

GHC_PACKAGE_PATH=\$(/usr/bin/ghc --print-global-package-db):$(echo $_packageconfdir | sed "s|$pkgdir||g" )
LD_LIBRARY_PATH=$2:\$LD_LIBRARY_PATH
export GHC_PACKAGE_PATH LD_LIBRARY_PATH
exec /usr/lib/$_pkgwithver/bin/\$(basename \$0) "\$@"

EOF
  chmod 0755 $1
}

package() {
  _setupLocalEnvVars
  local _licensedstdir=$pkgdir/usr/share/licenses/$_pkgwithver
  local _licensesrcdir=$_tmppackages/usr/share/$_pkgwithver/doc
  mkdir -p $_licensedstdir $_packageconfdir
  msg2 "Moving licenses..."
  ( cd $_licensesrcdir && find . -maxdepth 2 -name 'LICENSE' | parallel --no-run-if-empty --no-notice --bar "install -Dm444 $_licensesrcdir/{} $_licensedstdir/{}" )
  for d in usr/share/$_pkgwithver/{doc,data,man} usr/lib/$_pkgwithver/lib; do
    msg2 "Copying $(basename $d)..."
    ( cd $_tmppackages && tar cf - --exclude='*/LICENSE' $d ) | ( cd $pkgdir && tar xf - )
  done
  msg2 "Registering packages in package.conf.d..."
  find $_confdir -name '*.conf' -exec ghc-pkg update --force --package-db=$_packageconfdir {} >/dev/null 2>&1 \;
  msg2 "Creating scripts in /usr/bin..."
  local _wrapperScriptLocation=/usr/lib/$_pkgwithver/bin/gitit_wrapper.sh
  mkdir -p $pkgdir/usr/bin
  for binname in gitit expireGititCache; do
    local _binrel=usr/lib/$_pkgwithver/bin/$binname
    local _fullbinpath=$pkgdir/$_binrel
    install -Dm555 $_tmppackages/$_binrel $_fullbinpath
    [ -f "$_fullbinpath" ] && ln -s $_wrapperScriptLocation $pkgdir/usr/bin/$binname
  done
  # prepare ld-library-path from rpath entries and delete rpath entries
  find $pkgdir/usr/lib/$_pkgwithver/ -name '*.so' | parallel --no-notice --no-run-if-empty --bar "chrpath --list {} 2>/dev/null && chrpath --delete {} >/dev/null 2>&1" | sed -n 's|.*RPATH=||g p' | tr ':' '\n' | sort | uniq | sed -r -e "s|^.*/([^/]*)/dist/build|/usr/lib/$_pkgwithver/lib/\1|" >$srcdir/ld.path
  _createWrapperScript "$_wrapperScriptLocation" "$(cat $srcdir/ld.path | tr '\n' ':')"
}

# vim: set ft=sh syn=sh ts=2 sw=2 et:
