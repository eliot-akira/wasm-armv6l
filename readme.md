# WASM runtime on 32-bit ARM

Here are some notes on what it took to get a WASM runtime working on 32-bit ARM architecture (`armv6l`), such as Raspberry Pi Zero. A newer model like Zero 2 is on 64 bit, which is more widely supported. The following is an example of steps, adjust as needed.

## Install experimental build of Node.js targeting 32-bit ARM

This method is based on a helpful Gist and its comments: [Steps to install nodejs v6.2 on Raspberry Pi B+ (armv6)](https://gist.github.com/davps/6c6e0ba59d023a9e3963cea4ad0fb516).

Confirmed working for Node.js v21.7.3 on Pi Zero W (`armv6l`).

This seems to be the last version that can run on 32-bit ARM. See [nodejs/unofficial-builds](https://github.com/nodejs/unofficial-builds).

```sh
curl -L https://unofficial-builds.nodejs.org/download/release/v21.7.3/node-v21.7.3-linux-armv6l.tar.xz | tar xJ
cd node-*
rm CHANGELOG.md LICENSE README.md
cp -R * ~/.local
```

It's installed for the current user (`~/.local`) instead of all users (`/usr/local`). This way both `node` and `npm` commands work without `sudo`. Add `~/.local/bin` to the `PATH`, such as in `.bashrc`.

```
export PATH="~/.local/bin:$PATH"
```

### Running WASM binary with Node.js

Node.js cannot run WASM binary directly, it needs a JavaScript wrapper. See: [Node.js with WebAssembly](https://nodejs.org/en/learn/getting-started/nodejs-with-webassembly).

In addition, if the WASM module needs to interact with the OS, such as the file system, it needs [WASI](https://nodejs.org/api/wasi.html) (WebAssembly System Interface) or compiled with Emscripten ([`STANDALONE_WASM`](https://emscripten.org/docs/tools_reference/settings_reference.html#standalone-wasm)).

## Compile Wazero with Go targeting 32-bit ARM

This article had useful info: [Modern websites in a Raspberry Pi Zero with WebAssembly](https://wasmlabs.dev/articles/modern-websites-pi-zero/).

### Git

Install Git

```sh
sudo apt update
sudo apt install git
```

If that doesn't work, as in my case, build Git (see [Tags](https://github.com/git/git/tags)) from source. First, install dependencies. Some of these were found through trial and error.

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

Download the lastest version of Go (see [Releases](https://go.dev/dl/#stable)) for `armv6l`. This is what made it all possible, the fact that Go can run and build on this CPU architecture.

```sh
cd ~/src
curl -L https://go.dev/dl/go1.24.2.linux-armv6l.tar.gz | tar xz
cd ~/bin
ln -s ~/src/go/bin/go
```

Maybe next time try [`tinygo`](https://tinygo.org/).

### Wazero

The only WASM runtime I found that had a chance of running on 32-bit ARM is [Wazero](https://github.com/tetratelabs/wazero), written in pure Go. None of the others supported it: [Wasmtime](https://github.com/bytecodealliance/wasmtime/issues/3721), [Wasmer](https://github.com/wasmerio/wasmer/issues/1652), WAMR (WASM Micro Runtime).

```sh
cd ~/src
git clone --recursive --depth 1 --single-branch --branch main https://github.com/tetratelabs/wazero
cd wazero
go build
go build cmd/wazero/wazero.go

cd ~/bin
ln -s ~/src/wazero
```

The built binary is included in this repo `wazero-armv6l` for reference.

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
