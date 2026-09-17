# OPAM Repository for OCaml Typed Effects

This repository provides OPAM packages for the experimental `typed_effects` branch of OCaml, as well as ported libraries and tools (such as Dune).

## Quick Start

### 1. Add this repository to OPAM

To make it available for new switches (recommended):
```bash
opam repo add typed-effects git+https://github.com/bluddy/opam-typed-effects.git --set-default
```

Or for local development:
```bash
opam repo add typed-effects ~/source/ocaml/typed-effects/opam --set-default
```

### 2. Create an OPAM switch with the typed effects compiler

```bash
opam switch create typed-effects ocaml-variants.5.6.0+typed-effects
```

*(Note: If you didn't add the repo with `--set-default`, specify the repositories explicitly: `opam switch create typed-effects --repositories=typed-effects,default ocaml-variants.5.6.0+typed-effects`)*

### 3. Verify installation

```bash
eval $(opam env)
ocamlc -v
```
