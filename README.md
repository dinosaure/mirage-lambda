## mirage-lambda -- an eDSL to ship computation to MirageOS applications

Mirage Lambda allows to describe functions using a monorphic typed lambda
calculus with well-typed host primitives (for instance calls to MirageOS APIs).
A client can then describe a bit of computation, serialise the function and send
it, over the network, to be (safely) executed on a remote location.

## Installation

Mirage Lambda can be installed with `opam`:

    opam install mirage-lambda

If you don't use `opam` consult the [`opam`](opam) file for build
instructions.


## Documentation

The documentation and API reference is automatically generated by from
the source interfaces. It can be consulted [online][doc] or via
`odig doc cmdliner`.

[doc]: https://mirage.github.io/mirage-lambda

## Example

The factorial can be defined as a `('a, int -> int) Lambda.Expr.t` value.
The `'a` is the of the environment, `int -> int` is the type of the expression:

```ocaml
open Lambda

let fact =
  let open Expr in
  let main =
    let_rec Type.("x", int ** int) Type.int (fun ~context ~return ~continue ->
        let acc = fst context in
        let n = snd context in
        (if_ (n = int 0)
           (return acc)
           (continue (pair (acc * n) (n - int 1))))
      ) in
  lambda ("x", Type.int) (main $ (pair (int 1) (var Var.o)))
```

To ship the code and waiting for a server response:

```ocaml
  let u  = Expr.untype fact in      (* Generate an AST *)
  let s  = Parsetree.to_string u in (* Pretty-print the AST *)
  send s >>= receive
```

The lambda server, on the other-side will receive the code, type it, and
evaluate it and send the response back:

```ocaml
  receive () >>= fun s ->
  let u = parse_exn s in (* Parse *)
  let e = typ_exn u in   (* Type *)
  let v = eval e in      (* Evaluate *)
  send (string_of_value v)
```

The server can also defines a list of host function primitives that it can
exposes to the clients:

```ocaml
  let primitives = [
    primitive "string_of_int" [Type.int] Type.string string_of_int
    primitive "string_of_float" [Type.float] Type.string string_of_int
  ] in
  ...
  (* Exposes the names [string_of_int] and [string_of_float] in the context. *)
  let v = parse_exn ~primitives s in
  ...
```

## Sponsor

Mirage Lambda has been sponsored by [Tweag I/O](https://www.tweag.io/).

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
[<img src="https://www.tweag.io/img/tweag-med.png" height="65">](http://tweag.io)
