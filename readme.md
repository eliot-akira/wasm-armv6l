# WASM runtime on ARM 32bit architecture

Here are some notes on what it took to get a WASM runtime working on ARM 32bit architecture (`armv6l`), such as Raspberry Pi Zero. A newer model such as Zero 2 is on 64bit, which is more widely supported. The following is an example of steps, adjust as needed.

This article was a great help: [Modern websites in a Raspberry Pi Zero with WebAssembly](https://wasmlabs.dev/articles/modern-websites-pi-zero/).

### Git

Install Git

```sh
sudo apt update
sudo apt install git
```

If that doesn't work, as in my case, build Git from source. First, install dependencies. Some of these were found through trial and error.

```sh
mkdir ~/src
cd ~/src
sudo apt install make libssl-dev libghc-zlib-dev libexpat1-dev gettext openssl-dev libssl-dev libz-dev libcurl4-openssl-dev
curl -L https://github.com/git/git/archive/refs/tags/v2.49.0.tar.gz | tar xz
cd git-*
sudo make prefix=/usr/local all
sudo make prefix=/usr/local install
```

### Go

Download the lastest version of Go for `armv6l`. This is what made it all possible, the fact that Go can run and build on this CPU architecture.

```sh
cd ~/src
curl -L https://go.dev/dl/go1.24.2.linux-armv6l.tar.gz | tar xz
cd ~/bin
ln -s ~/src/go/bin/go
```

Maybe next time try [`tinygo`](https://tinygo.org/).

### Wazero

The only WASM runtime I found that had a chance of running on ARM 32bit is [Wazero](https://wazero.io). Wasmtime, Wasmer, WAMR (WASM Micro Runtime) and others do not support 32bit at this time.

```sh
cd ~/src
git clone --recursive --depth 1 --single-branch --branch main https://github.com/tetratelabs/wazero
cd wazero
go build
go build cmd/wazero/wazero.go

cd ~/bin
ln -s ~/src/wazero
```

### Example

Create `hello.go`.

```go
package main

import "fmt"

func main() {
  fmt.Println("Hello World!")
}
```

Compile to WASM. Note the OS `wasip1`, WebAssembly System Interface (WASI) Preview 1.

```sh
GOOS=wasip1 GOARCH=wasm go build -o hello.wasm hello.go
```

Run it.

```sh
wazero run hello.wasm
Hello World!
```
