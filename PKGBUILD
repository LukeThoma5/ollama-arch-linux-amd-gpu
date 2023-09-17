# Maintainer: Alexander F. Rødseth <xyproto@archlinux.org>
# Contributor: Matt Harrison <matt@harrison.us.com>

pkgname=ollama
pkgdesc='Create, run and share large language models (LLMs)'
pkgver=0.0.19
pkgrel=1
arch=(x86_64)
url='https://github.com/jmorganca/ollama'
license=(MIT)
makedepends=(cmake git go)
_ggmlcommit=9e232f0234073358e7031c1b8d7aa45020469a3b
_ggufcommit=53885d7256909ec3e2176cdc2477f3986c15ec69
_ollamacommit=45ac07cd025f9d1e84917db3f00e0f3e5651aede # tag: v0.0.19
source=(git+$url#commit=$_ollamacommit
        ggml::git+https://github.com/ggerganov/llama.cpp#commit=$_ggmlcommit
        gguf::git+https://github.com/ggerganov/llama.cpp#commit=$_ggufcommit)
b2sums=('SKIP'
        'SKIP'
        'SKIP')

prepare() {
  cd $pkgname

  rm -frv llm/llama.cpp/gg{ml,uf}

  # copy git submodule files instead of symlinking because the build process is sensitive to symlinks
  cp -r "$srcdir/ggml" llm/llama.cpp/ggml
  cp -r "$srcdir/gguf" llm/llama.cpp/gguf

  # do not git clone when "go generate" is being run
  sed -i 's|git submodule|true|g' llm/llama.cpp/generate.go
}

build() {
  cd $pkgname
  export CGO_CFLAGS="$CFLAGS" CGO_CPPFLAGS="$CPPFLAGS" CGO_CXXFLAGS="$CXXFLAGS" CGO_LDFLAGS="$LDFLAGS"
  go generate ./...
  go build -buildmode=pie -trimpath -ldflags=-linkmode=external -mod=readonly -modcacherw
}

check() {
  cd $pkgname
  go test ./...
}

package() {
  cd $pkgname
  install -Dm755 $pkgname "$pkgdir/usr/bin/$pkgname"
  install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
