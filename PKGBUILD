# Maintainer: Alexander F. Rødseth <xyproto@archlinux.org>
# Contributor: Matt Harrison <matt@harrison.us.com>
# Contributor: Kainoa Kanter <kainoa@t1c.dev>

pkgname=ollama
pkgdesc='Create, run and share large language models (LLMs)'
pkgver=0.1.22
pkgrel=2
arch=(x86_64)
url='https://github.com/jmorganca/ollama'
license=(MIT)
_ollamacommit=b538dc3858014f94b099730a592751a5454cab0a # tag: v0.1.22
# The llama.cpp git submodule commit hash can be found here:
# https://github.com/jmorganca/ollama/tree/v0.1.22/llm
_llama_cpp_commit=d2f650cb5b04ee2726663e79b47da5efe196ce00
makedepends=(cmake git go rocm-hip-sdk rocm-opencl-sdk clblast)
source=(git+$url#commit=$_ollamacommit
        llama.cpp::git+https://github.com/ggerganov/llama.cpp#commit=$_llama_cpp_commit
        sysusers.conf
        tmpfiles.d
        ollama.service)
b2sums=('SKIP'
        'SKIP'
        '3aabf135c4f18e1ad745ae8800db782b25b15305dfeaaa031b4501408ab7e7d01f66e8ebb5be59fc813cfbff6788d08d2e48dcf24ecc480a40ec9db8dbce9fec'
        'e8f2b19e2474f30a4f984b45787950012668bf0acb5ad1ebb25cd9776925ab4a6aa927f8131ed53e35b1c71b32c504c700fe5b5145ecd25c7a8284373bb951ed'
        'a773bbf16cf5ccc2ee505ad77c3f9275346ddf412be283cfeaee7c2e4c41b8637a31aaff8766ed769524ebddc0c03cf924724452639b62208e578d98b9176124')

prepare() {
  cd $pkgname

  rm -frv llm/llama.cpp

  # Copy git submodule files instead of symlinking because the build process is sensitive to symlinks
  cp -r "$srcdir/llama.cpp" llm/llama.cpp

  # Turn LTO on and set the build type to Release
  # sed -i 's,T_CODE=on,T_CODE=on -D LLAMA_LTO=on -D CMAKE_BUILD_TYPE=Release,g' llm/generate/gen_linux.sh

  # Display a more helpful error message
  sed -i "s|could not connect to ollama server, run 'ollama serve' to start it|ollama is not running, try 'systemctl start ollama'|g" cmd/cmd.go
}

build() {
  cd $pkgname
  # export CGO_CFLAGS="$CFLAGS" CGO_CPPFLAGS="$CPPFLAGS" CGO_CXXFLAGS="$CXXFLAGS" CGO_LDFLAGS="$LDFLAGS"
  export ROCM_PATH=/opt/rocm CLBlast_DIR=/usr/lib/cmake/CLBlast
  go generate -tags rocm ./...
  go build -tags rocm
}


check() {
  cd $pkgname
  go test ./api ./format
  ./ollama --version >/dev/null
}

package() {
  install -Dm755 $pkgname/$pkgname "$pkgdir/usr/bin/$pkgname"
  install -dm755 "$pkgdir/var/lib/ollama"
  install -Dm644 ollama.service "$pkgdir/usr/lib/systemd/system/ollama.service"
  install -Dm644 sysusers.conf "$pkgdir/usr/lib/sysusers.d/ollama.conf"
  install -Dm644 tmpfiles.d "$pkgdir/usr/lib/tmpfiles.d/ollama.conf"
  install -Dm644 $pkgname/LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
