# Maintainer: Alexander F. Rødseth <xyproto@archlinux.org>
# Contributor: Matt Harrison <matt@harrison.us.com>

pkgbase=ollama
pkgname=(ollama ollama-cuda)
pkgdesc='Create, run and share large language models (LLMs)'
pkgver=0.1.0
pkgrel=1
arch=(x86_64)
url='https://github.com/jmorganca/ollama'
license=(MIT)
makedepends=(cmake cuda git go)
_ollamacommit=5306b0269db6c6c4f716ff6c3c4514d5aba74c19 # tag: v0.1.0
# The git submodule commit hashes are here:
# https://github.com/jmorganca/ollama/tree/v0.1.0/llm/llama.cpp
_ggmlcommit=9e232f0234073358e7031c1b8d7aa45020469a3b
_ggufcommit=bc9d3e3971e5607a10ff4c24e39568ce1ac87271
source=(git+$url#commit=$_ollamacommit
        ggml::git+https://github.com/ggerganov/llama.cpp#commit=$_ggmlcommit
        gguf::git+https://github.com/ggerganov/llama.cpp#commit=$_ggufcommit)
b2sums=(SKIP SKIP SKIP)

prepare() {
  cd $pkgbase

  rm -frv llm/llama.cpp/gg{ml,uf}

  # Copy git submodule files instead of symlinking because the build process is sensitive to symlinks.
  cp -r "$srcdir/ggml" llm/llama.cpp/ggml
  cp -r "$srcdir/gguf" llm/llama.cpp/gguf

  # Do not git clone when "go generate" is being run.
  sed -i 's,git submodule,true,g' llm/llama.cpp/generate_linux.go

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
  go build -buildmode=pie -trimpath -ldflags=-linkmode=external -mod=readonly -modcacherw

  # with CUDA support (LLAMA_CUBLAS=on)
  cd ../${pkgbase}-cuda
  go generate ./...
  go build -buildmode=pie -trimpath -ldflags=-linkmode=external -mod=readonly -modcacherw
}

check() {
  (cd $pkgbase && go test ./...)
  (cd ${pkgbase}-cuda && go test ./...)
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
