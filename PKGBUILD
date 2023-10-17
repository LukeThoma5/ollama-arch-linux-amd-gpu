# Maintainer: Alexander F. Rødseth <xyproto@archlinux.org>
# Contributor: Matt Harrison <matt@harrison.us.com>

pkgbase=ollama
pkgname=(ollama ollama-cuda)
pkgdesc='Create, run and share large language models (LLMs)'
pkgver=0.1.3
pkgrel=1
arch=(x86_64)
url='https://github.com/jmorganca/ollama'
license=(MIT)
makedepends=(cmake cuda git go setconf)
_ollamacommit=832b4db9d4baf22497145dde55f334b292ed665f # tag: v0.1.3
# The git submodule commit hashes are here:
# https://github.com/jmorganca/ollama/tree/v0.1.1/llm/llama.cpp
_ggmlcommit=9e232f0234073358e7031c1b8d7aa45020469a3b
_ggufcommit=bc9d3e3971e5607a10ff4c24e39568ce1ac87271
source=(git+$url#commit=$_ollamacommit
        ggml::git+https://github.com/ggerganov/llama.cpp#commit=$_ggmlcommit
        gguf::git+https://github.com/ggerganov/llama.cpp#commit=$_ggufcommit
        gotest.patch::https://github.com/xyproto/ollama/commit/fddb303f23e68ef80a028257554979b22addb438.patch)
b2sums=('SKIP'
        'SKIP'
        'SKIP'
        '75cb9609585e5bdf7ba1714ae55ee77a769fa46973c3e06bec4e2bd3f965a1547f82aef963a0e37efaaaae8ea3e5cc4cc734b03fb9b056300e29e3cb715c8df3')

prepare() {
  cd $pkgbase

  rm -frv llm/llama.cpp/gg{ml,uf}

  # Copy git submodule files instead of symlinking because the build process is sensitive to symlinks.
  cp -r "$srcdir/ggml" llm/llama.cpp/ggml
  cp -r "$srcdir/gguf" llm/llama.cpp/gguf

  # Do not git clone when "go generate" is being run.
  sed -i 's,git submodule,true,g' llm/llama.cpp/generate_linux.go

  # Fix go test ./...
  patch -p1 -i ../gotest.patch

  # Set the correct version number
  setconf version/version.go 'var Version string' "\"$pkgver\""

  cd ..

  # Copy the sources to an ollama-cuda directory.
  cp -r $pkgbase ${pkgbase}-cuda

  # Disable CUDA for the regular ollama package (it is enabled by default).
  sed -i 's,DLLAMA_CUBLAS=on,DLLAMA_CUBLAS=off,g' $pkgbase/llm/llama.cpp/generate_linux.go
}

build() {
  export CGO_CFLAGS="$CFLAGS" CGO_CPPFLAGS="$CPPFLAGS" CGO_CXXFLAGS="$CXXFLAGS" CGO_LDFLAGS="$LDFLAGS"

  # without CUDA support (LLAMA_CUBLAS=off)
  cd $pkgbase
  go generate ./...
  go build -buildmode=pie -trimpath -mod=readonly -modcacherw -ldflags=-linkmode=external -ldflags=-buildid=''

  # with CUDA support (LLAMA_CUBLAS=on)
  cd ../${pkgbase}-cuda
  go generate ./...
  go build -buildmode=pie -trimpath -mod=readonly -modcacherw -ldflags=-linkmode=external -ldflags=-buildid=''
}

check() {
  cd $pkgbase
  go test ./...
}

package_ollama() {
  cd $pkgname
  install -Dm755 $pkgbase "$pkgdir/usr/bin/$pkgbase"
  install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}

package_ollama-cuda() {
  depends=(cuda)
  provides=(ollama)
  conflicts=(ollama)
  cd $pkgname
  install -Dm755 $pkgbase "$pkgdir/usr/bin/$pkgbase"
  install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
