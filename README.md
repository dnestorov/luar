# Luar: Lua reflection bindings for Go

Luar is designed to make using Lua from Go more convenient. Go structs, slices
and maps can be automatically converted to Lua tables and vice-versa. The
resulting conversion can either be a copy or a proxy. In the latter case, any change
made to the result will reflect on the source.

Any Go function can be made available to Lua scripts, without having to write
C-style wrappers.

Luar support cyclic structures (`map[string]interface{}`, lists, etc.).

User-defined types can be made available to Lua as well: their exported methods
can be called and usual operations such as indexing or arithmetic can be
performed.

See the [documentation](http://godoc.org/github.com/stevedonovan/luar) for usage
instructions and examples.

# Installation

Install with

    go get <repo>/luar

The original Luar uses Alessandro Arzilli's [golua](https://github.com/aarzilli/golua).
This fork of Luar uses D.Nestorov's [golua](https://github.com/dnestorov/golua).
See golua's homepage for further installation details.

# Usage

This is a fork of https://github.com/stevedonovan/luar that runs with Lua 5.3.3 and supports FFI (https://github.com/dnestorov/luaffifb).
The FFI interface is a fork from Facebook's luaffifb (https://github.com/facebook/luaffifb). The only difference is that the former builds a static library with Premake.

The final results is:
```go
package main

import (
	"fmt"
	"log"

	"github.com/dnestorov/luar"
)

var lcode = `
local ffi = require("ffi")
ffi.cdef[[
	int printf(const char *fmt, ...);
]]
ffi.C.printf("Hello %s from FFI!\n", "world")

print("Hello world from Lua!")

Print("Hello world from Go!")
`

func main() {
	l := luar.Init()
	defer l.Close()

	l.OpenFFI()

	luar.Register(l, "", luar.Map{
		// Go functions may be registered directly.
		"Print": fmt.Println,
	})

	err := l.DoString(lcode)
	if err != nil {
		log.Fatal(err)
	}
}
```

# REPL

An example REPL is available in the `cmd` folder.

# Issues

The `GoToLua` and `LuaToGo` functions take a `reflect.Type` parameter, which is
bad design. Sadly changing this would break backward compatibility.
