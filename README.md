# OPAM Repository for OCaml Typed Effects

This repository provides OPAM packages for the experimental `typed_effects` branch of OCaml, as well as ported libraries and tools (such as Dune).

## Quick Start

### 1. Register the repository in OPAM (isolated)

Register the repository without modifying existing switches or defaults:
```bash
opam repo add typed-effects git+https://github.com/bluddy/opam-typed-effects.git --dont-select
```

Or for local development:
```bash
opam repo add typed-effects ~/source/ocaml/typed-effects/opam --dont-select
```

### 2. Create an OPAM switch for typed effects

Enable the `typed-effects` repository specifically for this new switch:
```bash
opam switch create typed-effects --repositories=typed-effects,default ocaml-variants.5.6.0+typed-effects
```

### 3. Verify installation

```bash
eval $(opam env)
ocamlc -v
```
