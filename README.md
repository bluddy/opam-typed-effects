# OPAM Repository for OCaml Typed Effects

This repository provides OPAM packages for the experimental `typed_effects` branch of OCaml, as well as ported libraries and tools (such as Dune and `stdlib-v2`).

---

## Quick Start: Installing the Compiler & Environment

### 1. Register the repository in OPAM

Register the repository without modifying your existing switches or defaults:

```bash
opam repo add typed-effects git+https://github.com/bluddy/opam-typed-effects.git --dont-select
```

*(For local development from a clone:)*
```bash
opam repo add typed-effects ~/source/ocaml/typed-effects/opam --dont-select
```

### 2. Create an OPAM switch for typed effects

Create a new isolated switch with the `typed-effects` repository prioritized ahead of the `default` repository:

```bash
opam switch create typed-effects --repositories=typed-effects,default ocaml-variants.5.6.0+typed-effects
```

### 3. Activate the environment

```bash
eval $(opam env)
ocamlc -v
```

This should report: `The OCaml compiler, version 5.6.0+typed-effects`.

### 4. Install Dune & the Standard Library Overlay

```bash
opam install dune stdlib_v2 -y
```

---

## Building Your First Effectful Program

### Option A: Standalone File (with `ocamlopt`)

Create `test.ml`:

```ocaml
type _ Effect.t += Say : string -> unit Effect.t

let greet () =
  Effect.perform (Say "Hello, Typed Effects!")

let () =
  match greet () with
  | () -> ()
  | effect (Say msg), k ->
      print_endline msg;
      Effect.Deep.continue k ()
```

Compile and run:
```bash
ocamlopt -o test test.ml
./test
```

---

### Option B: With Dune & `stdlib-v2`

Create a new directory:
```bash
mkdir my_effect_app && cd my_effect_app
```

Create `dune-project`:
```lisp
(lang dune 3.14)
(name my_effect_app)
```

Create `bin/dune`:
```lisp
(executable
 (name main)
 (libraries stdlib_v2))
```

Create `bin/main.ml`:
```ocaml
open Stdlib_v2

type _ Effect.t += Log : string -> unit Effect.t

let () =
  let handle f =
    match f () with
    | v -> v
    | effect (Log msg), k ->
        Printf.printf "[Effect Log] %s\n%!" msg;
        Effect.Deep.continue k ()
  in
  handle (fun () ->
    let numbers = [1; 2; 3; 4; 5] in
    let evens =
      List.filter (fun x ->
        Effect.perform (Log (Printf.sprintf "Inspecting %d" x));
        x mod 2 = 0
      ) numbers
    in
    List.iter (fun x ->
      Effect.perform (Log (Printf.sprintf "Found even: %d" x))
    ) evens
  )
```

Run your program:
```bash
dune exec ./bin/main.exe
```

#### Opening `stdlib-v2` Globally (Without per-file `open`)

To avoid writing `open Stdlib_v2` in every source file, you can open it globally in Dune using the `-open Stdlib_v2` flag:

**Per executable / library (`bin/dune`):**
```lisp
(executable
 (name main)
 (libraries stdlib_v2)
 (flags :standard -open Stdlib_v2))
```

**Project-wide (`dune` or `dune-workspace`):**
```lisp
(env
 (_
  (flags :standard -open Stdlib_v2)))
```

---

## Typed Effects Syntax Cheat Sheet

| Feature | Syntax | Explanation |
| :--- | :--- | :--- |
| **Pure arrow (default)** | `'a -> 'b` | Functions are pure by default ($\emptyset$). |
| **Explicit pure arrow** | `'a -[]-> 'b` | Syntactic alias for pure empty row. |
| **Specific effect arrow** | `'a -[ Log ]-> 'b` | Function may perform the `Log` effect. |
| **Effect-polymorphic arrow** | `'a -[ 'e ]-> 'b` | Arrow carrying a row variable `'e`. |
| **Multiple effects** | `'a -[ Log, Yield \| 'e ]-> 'b` | Row with labels and row extension variable `'e`. |
| **Effect declaration** | `type _ Effect.t += Eff : arg -> res Effect.t` | Extensible variant for effect constructors. |
| **Performing an effect** | `Effect.perform (Eff arg)` | Yields control to the enclosing ambient handler. |
| **Handling effects** | `match body () with`<br>`\| v -> v`<br>`\| effect (Eff x), k -> Effect.Deep.continue k res` | Deep pattern matching on effects with continuation `k`. |
| **Effectful Standard Library** | `open Stdlib_v2` | Overlays standard library modules (`List`, `Array`, `Option`, `Result`, `Seq`, `Fun`) with effect-polymorphic HOFs. |
| **Abstract effect in signature** | `effect eff` | Declares an abstract effect row in a module signature. |
| **Manifest effect alias** | `effect eff = -[ Log, Yield ]-` | Defines or refines a concrete effect row in a module. |
| **Functor with effect parameter** | `module Make (E : sig effect eff val act : unit -[ eff ]-> unit end)` | Functor abstracting over effect capabilities. |
| **Module type constraint** | `RUNNER with effect eff = -[ Yield ]-` | Refines an abstract effect in a signature. |
| **Destructive substitution** | `RUNNER with effect eff := -[ ]-` | Destructively substitutes an effect (e.g. to pure), simplifying signatures. |

---

## Included Packages

- `ocaml-variants.5.6.0+typed-effects`: OCaml 5.6 development compiler with pure-by-default, row-polymorphic typed effects.
- `dune.3.24.2+typed-effects`: Dune build system compatible with OCaml 5.6 trunk and typed effects.
- `stdlib_v2`: Standard library overlay offering effect-polymorphic higher-order functions.
- `miou`: Composable concurrency primitives and effect-based scheduler ported to typed effects and `stdlib_v2`.

