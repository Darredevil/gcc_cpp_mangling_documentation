This document gathers my findings about gcc c++ name mangling.

It is to be considered as supplementaty materials to the [Itanium C++ ABI's mangling section](https://mentorembedded.github.io/cxx-abi/abi.html#mangling), especially it explores what accounts as a symbol to be substituted in the case of regular functions, templates and abbreviations.

### Preambule
```
void foo()
```
`foo` is mangled as `_Z3foov`
- `_Z` preambule is always here, it starts the mangled name, for OSX it would be `__Z`.
- `3foo` function name, length encoded.
- `v` no parameter is encoded as a `void` parameter.

Note: the return type is not encoded here (although there are cases where it is encoded: function pointers and funtion template instances)

### Encoding parameters

Basic types are encoded using a single letter.
eg. `void foo(int)` is encoded `_Z3fooi`

No parameter is encoded as if a single void parameter were passed.
eg. `void foo()` is encoded `_Z3foov`

Parameters are encoded one after the other.
eg. `void foo(char, int, short)` is encoded `_Z3foocis`

### Substitutions

To save space a compression scheme is used where symbols that appears multiple times are then substituted by an item from the sequence : `S_`, `S0_`, `S1_`, `S2_`, etc ...

eg.
```
void foo(void*, void*)
```
`foo` would be encoded as `_Z3fooPvS_`. To be decomposed as 
- `_Z`
- `3foo`
- `Pv` stands for "pointer to void". Since it's not a basic type it's accounted as a symbol.
- `S_` refers to the first symbol encoded, here `Pv`.

Note: `foo` is a declaration, not a type and so it doesn't account as a substituable symbol.

### namespace

namespaces are considered as symbols.

```
namespace a {
	struct A{};
	void foo(A){}
}
```
`foo` would be encoded as `_ZN1a3fooENS_1AE`
- `a::foo` is encoded as `N1a3fooE`
  - It is enclosed by `N`..`E` (symbol is nested and not in `std`)
  - `a` is encoded `1a`
  - `foo` is encoded `3foo`
- `a::A` is encoded `NS_1AE`
  - enclosed in `N`..`E` (symbol is nested and not in `std`)
  - `a` is encoded `S_`
  - `A` is encoded `1A`

Note: if namespace is `std` then it is abbreviated and nested symbol are no more enclosed in `N`..`E`
```
namespace std {
	struct A{};
	void foo(A){}
}
```
`foo` is encoded as `_ZSt3fooSt1A`
 - `std::foo` is encoded as `St3foo`
 - `std::A` is encoded as `St1A`

Note: `std` is not substituted since it is an abbreviation.

### Structs/Classes

Member functions are encoded as if they were in a namespace with the exception of const members which starts with a `K`.

```
class C {
	void foo() const {}
};
```
`foo` is encoded as `_ZNK1C3fooEv`
 - `C::foo` is encoded as `NK1C3fooE`
   - `K` is added at the beginning of the symbol because `foo` is `const`.
   - It is enclosed by `N`..`E` (symbol is nested)
