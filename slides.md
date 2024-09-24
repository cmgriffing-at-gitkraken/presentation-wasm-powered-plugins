---
# You can also start simply with 'default'
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: Welcome to Slidev
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
---

# Creating a WASM-Powered Plugin System For Your App

The Power of Extism

---

# Introduction

- Software Dev for 9 years
- Developer Advocate at GitKraken
- Twitch Programming Streamer
- Snowboarder
- cmgriffing on all socials
  - Twitch
  - YouTube
  - Twitter
  - LinkedIn
  - etc

---

# What is WASM?

WebAssembly: a low-level assembly-like language with a compact binary format that runs with near-native performance

<!-- 

according to MDN

-->

---

# The History of WASM

- 2011: Emscripten library started
- 2011: Google NaCL (Native Client)
- 2013: asm.js
- 2015: WASM Announced
- 2017: NaCL deprecated in favor of WASM
- 2017: Broad browser support
- 2019: WASM Threads enabled by default in Chrome
- 2022: WASM 2.0 draft

---

# asm.js Briefly

```ts {all|1|2|5|9-10|14-16|all}
function BasicMathModule(stdlib, foreign, heap) {
  "use asm";

  // Variable Declarations
  var sqrt = stdlib.Math.sqrt;

  // Function Declarations
  function square(x) {
      x = +x;
      return +(x*x);
  }

  function add(x, y) {
      x = x|0;
      y = y|0;
      return (x + y)|0;
  }

  return { square: square, add: add };
}
```

---

# The Birth and Death of JavaScript

A talk by Gary Bernhardt from PyCon 2014

https://www.destroyallsoftware.com/talks/the-birth-and-death-of-javascript

<img src="./images/birth-and-death-video.png">

<!--  

- His talk focuses on asm.js
- Pronounces JS as YavaScript
- Comedic but still informative
- Predicted where this stuff could go

-->


---

# Popular Usage

- Figma
- CapCut Web
- 1Password
- Google Earth
- Ublock Origin
- lichess

---

# Running WASM (Browsers)

- All Major Browsers
  - Chrome
  - Firefox
  - Opera
  - Safari
  - Edge

---

# WASI

WebAssembly System Interface

- Designed by Mozilla
- Provides POSIX-like features
  - Such as file I/O

---
layout: image-left
image: ./images/solomon-hykes.jpg
---

# WASM + WASI

> If WASM+WASI existed in 2008, we wouldn't have needed to create Docker. That's how important it is. WebAssembly on the server is the future of computing.

\- Solomon Hykes (2019), Creator of Docker

---

# Running WASM (outside browsers)

- Embeddable Runtimes
  - Extism
  - Wasmtime
  - Spin SDK
  - Wasmer.io

---

<div class="grid grid-cols-2 w-full h-full auto-rows-fr" frontmatter="[object Object]"><div class="slidev-layout default"><h1>Extism</h1><blockquote><p>a plug-in system for everyone</p></blockquote></div><div class="w-full h-auto" style="color: white; background-color: white; background-image: url(&quot;./images/extism-language-support.png&quot;); background-repeat: no-repeat; background-position: center center; background-size: contain;"></div></div>

---
layout: two-cols-header
---

# Extism Concepts: Hosts

## Host (SDKs)

::left::

- JS
  - Browsers
  - Node
- C/C++
- Elixir
- Go
- Haskell
- Java

::right::

- .Net
- OCaml
- Perl
- PHP
- Python
- Ruby
- Rust
- Zig

::bottom::

<div class="h-8 mt-8 mb-8"></div>

---

# Extism Concepts: Plugins

## Plug-in (PDKs)

- JS
- AssemblyScript
- C
- Go
- Haskell
- .Net
- Rust
- Zig

---

# Extism Concepts: 2-way Communication

- passing data from host to plugin by calling plugin functions
- passing data from plugin to host by calling host functions

## Memory

- Host/Plug-in seperately managed memory spaces
- Extism creates its own memory space for communication
- Communication is done via message passing
  - Objects must be serialized
    - JSON recommended

<!-- TODO: Maybe add image to the side? -->

---
layout: two-cols-header
---

# Extism CLI


::left::

## Scaffolding plugins

```bash
extism generate plugin -o new-plugin
Select a PDK language to use for your plugin:  
                                                
> 1. Rust                                      
  2. JavaScript                                
  3. Go                                        
  4. Zig                                       
  5. C#                                        
  6. F#                                        
  7. C                                         
  8. Haskell                                   
  9. AssemblyScript
```

::right::

## Testing plugins without a Host

```bash
extism call --input "this is a test" test/code.wasm count_vowels
{"count": 4}
```

::bottom::

<div class="h-8 mt-8 mb-8"></div>

---
layout: two-cols-header
---

# Why Extism?

Exploring a plugin system for GitKraken (unofficial and unplanned for now)

::left::

## GitKraken Platform:
- CLI: Go
- GKD: Electron JS/TS
- GitLens: TS
- JetBrains Plugin: Java/Gradle
  - (planned, not available yet)

::right::

## Reasoning:
- Consistency
  - Process
  - Documentation
- Compatibility

::bottom::

<div class="h-[10rem]"></div>

<!-- 

GitKraken wants to meet developers where they work.

Which is why we have all of the different surfaces of our platform (cli, desktop, vscode, web, etc)

By the same token, we would want people to be able to develop plugins in their language of choice (within reason). Sorry, COBOL developers.

-->

---

# What I did?

Basic functionality for:
  - GitKraken CLI
  - VSCode GitLens

Basic Process:
  - Install a Host SDK
    - CLI: Go
    - GitLens: TS
  - Write a plugin using a PDK
    - TS
    - Go

---


# Notes: HTTP

Must use Extism API instead of native APIs

<!-- Use Shiki magic move for code -->

````md magic-move {lines: true}

```go
import (
	"encoding/json"
	"fmt"
	"net/http"

	"github.com/extism/go-pdk"
)

//go:export greet
func Greet() int32 {
	name := pdk.InputString()
	pdk.OutputString("Hello, " + name)

	bodyReader := bytes.NewReader([]byte(fmt.Printf(`{"name": "%s"}`, )))

	req, err := http.NewRequest(http.MethodPost, "https://localhost/echo", bodyReader)

	return 0
}
```

```go
import (
	"encoding/json"
	"fmt"

	"github.com/extism/go-pdk"
)

//go:export greet
func Greet() int32 {
	name := pdk.InputString()
	pdk.OutputString("Hello, " + name)

	req := pdk.NewHTTPRequest(pdk.MethodPost, "https://localhost/echo")
	req.SetBody([]byte(fmt.Printf(`{"name": "%s"}`, )))
	req.Send()

	return 0
}
```
````

---

# Notes: OutputString

<!-- Use Shiki magic move for code -->

---

# Notes: Host Functions (JS only?)

<!-- Use Shiki magic move for code -->

---

# Notes: JS SDK Types

<!-- Use twoslash for highlighting the lack of a type -->

---

# TEMP: OUTLINE


- Quirks/Issues found along the way
  - http
  - JS SDK could use some stronger types
  - 

