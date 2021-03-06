[![Build Status](https://travis-ci.org/andrewcooke/AutoTypeParameters.jl.svg?branch=master)](https://travis-ci.org/andrewcooke/AutoTypeParameters.jl)
[![Coverage Status](https://coveralls.io/repos/andrewcooke/AutoTypeParameters.jl/badge.svg)](https://coveralls.io/r/andrewcooke/AutoTypeParameters.jl)

[![AutoTypeParameters](http://pkg.julialang.org/badges/AutoTypeParameters_0.3.svg)](http://pkg.julialang.org/?pkg=AutoTypeParameters&ver=0.3)
[![AutoTypeParameters](http://pkg.julialang.org/badges/AutoTypeParameters_0.4.svg)](http://pkg.julialang.org/?pkg=AutoTypeParameters&ver=0.4)
[![AutoTypeParameters](http://pkg.julialang.org/badges/AutoTypeParameters_0.5.svg)](http://pkg.julialang.org/?pkg=AutoTypeParameters&ver=0.5)


# AutoTypeParameters

A Julia library to reversibly encode "any" value so that it can be used as a
type parameter.

## Do I Need This?

You need this if you have an error like:

```
ERROR: TypeError: apply_type: in Val, expected Type{T}, got XXX
```

when you try to create a "complicated" dependent type.

That error is (partly) explained
[here](https://groups.google.com/forum/#!topic/julia-users/Ihl50vgSQxw) - it
occurs because the kinds of things that can be types in Julia is limited.
Partly for sensible reasons (you don't want a mutable type), but also because
of arbitrary implementation details.

## What Does It Do?

This package has two functions - `freeze()` and `thaw()` - that translate
arbitrary values back and forth into a form that *is* accepted by Julia.

## How Does It Work?

You can choose between two encondings.

### Using showall()

By default, `freeze()` takes the output from `showall()` and converts it into
a Symbol, while `thaw()` uses `eval()` to convert it back into a "real" value:

```julia
julia> using AutoTypeParameters

julia> freeze("a string")
symbol("ATP \"a string\"")

julia> thaw(eval, symbol("ATP \"a string\""))
"a string"
```

The advantage of the default approach is that the type parameter is readable.
The disadvantage, of course, is that it requires `showall()` to generate
output that `eval()` can handle.

### Using serialize()

Alternatively (eg for values which do not have a useful `showall()` function),
the base64 encoded output from `serialize()` can be used:

```julia
julia> using AutoTypeParameters

julia> freeze("a string"; format=:serialize)
symbol("ATP=JhWGYSBzdHJpbmc=")

julia> thaw(eval, symbol("ATP=JhWGYSBzdHJpbmc="))
"a string"
```

## Warnings

Because this package uses `eval()` it should not be passed arbitrary values
rom an untrusted user.

## Example

```julia
julia> using AutoTypeParameters

julia> type MyType{N}
           x
       end

julia> MyType{"strings not allowed"}(42)
ERROR: TypeError: apply_type: in MyType, expected Type{T}, got ASCIIString

julia> MyType(N, x) = MyType{freeze(N)}(x)
MyType{N}

julia> MyType("strings not allowed", 42)
MyType{symbol("ATP \"strings not allowed\"")}(42)

julia> extract_type{N}(x::MyType{N}) = thaw(eval, N)
extract_type (generic function with 1 method)

julia> extract_type(MyType("strings not allowed", 42))
"strings not allowed"
```
