---
Title: Add scopeImport helper function to package sets
Author: jonringer
Discussions-To: https://github.com/NixOS/nixpkgs/pull/274179
Status: Draft
Type: Standards Track
Topic: Packaging
Created: 2024-10-09
---

# Summary

`callPackage` has been an immensely useful tool to reduce the amount of boilerplate
needed to pass attrs to a nix expression. However, `callPackage` also universally
applies `mkOverridable` to the result which adds both `.override` and
`.overrideDerivation`. This isn't always desirable behavior, especially when
the resulting attr set should be free of these continuations; for example,
child package scope or partially applied functions. Omitting these attrs allows
for slight eval performance wins, but also a better user experience when functions
aren't expected in the resulting attr set.

## Detailed Implementation

See https://github.com/jonringer/nix-lib/pull/2 for implementation.

## Trivial example

```
nix-repl> callPackage = customisation.callPackageWith { foo = "bar"; }

nix-repl> scopeImport = customisation.scopeImportWith { foo = "bar"; }

nix-repl> myFunc = { foo }: { inherit foo; }

nix-repl> callPackage myFunc { }
{ foo = "bar"; override = { ... }; overrideDerivation = «lambda @ /home/jon/projects/lib/customisation.nix:151:32»; }

nix-repl> scopeImport myFunc { }
{ foo = "bar"; }
```

## Unresolved issues

- Error messages are slightly less helpful now
  - However, they previously were labeled as "callPackageWith" even though `callPackage` was used
```
nix-repl> scopeImport ({ doesntExist }: "hi") { }
error:
       … from call site

         at «string»:1:1:

            1| scopeImport ({ doesntExist }: "hi") { }
             | ^

       … while calling 'callGenericWith'

         at /home/jon/projects/lib/customisation.nix:227:59:

          226|   */
          227|   callGenericWith = funcName: continuation: autoArgs: fn: args:
             |                                                           ^
          228|     let

       error: evaluation aborted with the following error message: 'lib.customisation.scopeImportWith: Function called without required argument "doesntExist" at <unknown location>'
```

- Should the name just use the function names most likely to be used by users?
  - E.g. `error: evaluation aborted with the following error message: 'scopeImport: Function called without required argument "doesntExist" at <unknown location>'`
  - There isn't a strict correlation to what the partially applied function's name would be, however, in practice it almost always is supplied by `makeScope`

## Meta concerns

- Is there a better name than `scopeImport`?
  - This name plays on the theme of `makeScope` and `newScope` being used to provide a `scopeImport`
  - `callPackage` is "something when called, returns a package", so the analagous name would be `callAttSet` which doesn't feel as descriptive.
