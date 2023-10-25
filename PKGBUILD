# Maintainer: Alexander F. Rødseth <xyproto@archlinux.org>
# Contributor: Matt Harrison <matt@harrison.us.com>

pkgbase=ollama
pkgname=(ollama ollama-cuda)
pkgdesc='Create, run and share large language models (LLMs)'
pkgver=0.1.5
pkgrel=1
arch=(x86_64)
url='https://github.com/jmorganca/ollama'
license=(MIT)
makedepends=(cmake cuda git go setconf)
_ollamacommit=cecf83141e3813ad7e268521e6a95efce61cb146 # tag: v0.1.5
# The git submodule commit hashes are here:
# https://github.com/jmorganca/ollama/tree/v0.1.5/llm/llama.cpp
_ggmlcommit=9e232f0234073358e7031c1b8d7aa45020469a3b
_ggufcommit=9e70cc03229df19ca2d28ce23cc817198f897278
source=(git+$url#commit=$_ollamacommit
        ggml::git+https://github.com/ggerganov/llama.cpp#commit=$_ggmlcommit
        gguf::git+https://github.com/ggerganov/llama.cpp#commit=$_ggufcommit)
b2sums=('SKIP'
        'SKIP'
        'SKIP')

prepare() {
  cd $pkgbase

  rm -frv llm/llama.cpp/gg{ml,uf}

  # Copy git submodule files instead of symlinking because the build process is sensitive to symlinks.
  cp -r "$srcdir/ggml" llm/llama.cpp/ggml
  cp -r "$srcdir/gguf" llm/llama.cpp/gguf

  # Do not git clone when "go generate" is being run.
  sed -i 's,git submodule,true,g' llm/llama.cpp/generate_linux.go

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
