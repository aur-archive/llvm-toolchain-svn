# Maintainer: Martin Schulze <mail dot martin dot schulze at gmx dot net>
pkgname=llvm-toolchain-svn
pkgver=3.6.0svn.r225437
pkgrel=1
pkgdesc="LLVM, compiler-rt, lldb, clang/-tools-extra from the latest SVN sources"
url="http://llvm.org/"
arch=('i686' 'x86_64')
license=('UIUC')
depends=('ocaml' 'swig' 'libedit')
makedepends=('subversion' 'cmake' 'python2' 'libffi' 'ocaml-ctypes' 'ocaml-ounit')
optdepends=('libc++-svn') 
provides=('llvm' 'compiler-rt' 'clang' 'llvm-ocaml' 'clang-analyzer' 'lldb')
# replaces=('llvm' 'clang' 'llvm-ocaml' 'clang-analyzer')
conflicts=('llvm' 'clang' 'llvm-ocaml' 'clang-analyzer')
changelog=".Changelog"
options=('staticlibs' 'emptydirs')

source=("$pkgname"::'svn+http://llvm.org/svn/llvm-project/llvm/trunk'
	"$pkgname/tools/clang"::'svn+http://llvm.org/svn/llvm-project/cfe/trunk'
	"$pkgname/tools/clang/tools/extra"::'svn+http://llvm.org/svn/llvm-project/clang-tools-extra/trunk'
	"$pkgname/projects/compiler-rt"::'svn+http://llvm.org/svn/llvm-project/compiler-rt/trunk'
	"$pkgname/tools/lldb"::'svn+http://llvm.org/svn/llvm-project/lldb/trunk'
)
md5sums=('SKIP'
	'SKIP'
	'SKIP'
	'SKIP'
	'SKIP'
)

if [ ${CARCH} == x86_64 ]; then
    _target=$CARCH
else
    _target=x86
fi

pkgver(){
	cd "$srcdir/$pkgname"
	local ver=`./configure --version | grep "LLVM configure" | cut -d ' ' -f 3`
	local rev="$(svnversion)"
	printf "%s.r%s" "$ver" "${rev//[[:alpha:]]}"
}

build(){
    cd "$srcdir"
    mkdir -p build
    cd build
    msg "Entered `pwd`"
    _python2bin="$srcdir/python2bin"
    msg "Building Python2 bin '$_python2bin'"
    mkdir -p "$_python2bin"
    cd "$_python2bin"
    ln -fs /usr/bin/python2 python
    ln -fs /usr/bin/python2-config python-config
    export PATH="$_python2bin/:${PATH}"
    cd -

    # Include location of libffi headers in CPPFLAGS
    export CPPFLAGS="$CPPFLAGS $(pkg-config --cflags libffi)"

# remove lldb from build as long as compiler isn't clang.
#    if [ "${CXX}" == "clang++" ]; then
#	msg "OK compiler is Clang, building lldb"
#	    else	
#	 msg "lldb requires clang to build, removing it from build"
#	 rm -rf "$srcdir/$pkgname/tools/lldb"
#    fi
    
# remove "sample" from projects... it does not do anything useful
    msg "removing 'sample' project"
    rm -rf "$srcdir/$pkgname/projects/sample"
    
# several undocumented configure options that were used by someone else to build LLVM/Clang with clang and c++...
# those that fail are commented out again.
#build only enables host target.
    msg "Configuring build"
    ../$pkgname/configure \
    --prefix=/usr \
    --libdir=/usr/lib/llvm \
    --enable-cxx11 \
    --with-python="$_python2bin/python" \
    --sysconfdir=/etc \
    --enable-shared \
    --enable-pic \
    --enable-polly \
    --enable-jit \
    --enable-libffi \
    --disable-libcpp \
    --enable-targets=host \
    --with-cxx-include-arch=$_target \
    --enable-ltdl-install \
    --disable-expensive-checks \
    --disable-debug-runtime \
    --disable-assertions \
    --enable-optimized \
    --enable-threads \
    --disable-multilib \
    --with-optimize-option=-O2 \
    --with-binutils-include=/usr/include

#    --with-clang \
#    --with-built-clang \
#    --disable-bootstrap \
#    --no-recursion \
#    --no-create \
#    --with-c-include-dirs=
# due to linking errors with libc++, I have currently explicitly disabled it.
# to test compiling with LLVM libc++. change --disable-libcpp to --enable-libcpp.

#fixing packaging problems with libc++ makefile
#essentially the same functions can be performed by toolchain.install
# sed -i 's/chown -R root:wheel $(HEADER_DIR)/#placeholder: chown -R broke packaging/b' $srcdir/llvm/projects/libcxx/Makefile 
    msg "Building with path '${PATH}'"

    if [ ${MAKE_PARALLEL} ]; then
	make -j ${MAKE_PARALLEL}  REQUIRES_RTTI=1 || return 1
    else
	make REQUIRES_RTTI=1 || return 1
    fi
}

package() { 
    cd "$srcdir/build"
    make DESTDIR=${pkgdir} install || return 1 
    #getting some ld paths OK
    mkdir -p "$pkgdir/etc/ld.so.conf.d"
    echo "/usr/lib/llvm" > "$pkgdir/etc/ld.so.conf.d/llvm.conf"
    echo "/usr/lib/ocaml" > "$pkgdir/etc/ld.so.conf.d/llvm-ocaml.conf"

}
