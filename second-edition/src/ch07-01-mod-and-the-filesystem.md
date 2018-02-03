## `mod` et le sytème de fichier

Nous alons commencer notre exemple de module en créant un nouveau projet avec
Cargo, mais au lieu de créer un crate pour un binaire, nous allons créer un
crate pour une bibliothèque : ce sera un projet que les autres personnes
pourront intégrer dans leurs projets comme étant une dépendance. Par exemple,
le crate `rand` utilisé dans le chapitre 2 est un crate de bibliothèque que
nous avons utilisé comme dépendance quand le projet du jeu du plus ou moins.

Nous alons créer une structure de bibliothèque qui va apporter des
fonctionnalités pour des réseaux génériques; nous nous concentrerons sur
l'organisation des modules et des fonctions mais nous ne nous préoccuperons pas
du code qui est dans le corps des fonctions. Nous allons appeller notre
bibliothèque `communicator`. Par défaut, Cargo va créer une bibliothèque sauf
si un autre type de projet est précisé : si nous enlevons l'option `--bin` que
nous avons utilisé tout au long des chapitres précédents, notre projet sera une
bibliothèque :

```text
$ cargo new communicator
$ cd communicator
```

Notez que Cargo a généré *src/lib.rs* au lieu de *src/main.rs*. Dans
*src/lib.rs* nous avons ceci :

<span class="filename">Nom du fichier : src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

Cargo crée un exemple de test pour nous aider à démarrer notre bibliothèque,
à la place du code "Hello, world!" que nous avons quand nous utilisons l'option
`--bin`. Nous verrons les syntaxes `#[]` et `mod tests` dans la section
“Utiliser `super` pour accéder au module parent” plus tard dans ce chapitre,
mais pour le moment, laissez ce code en bas de *src/lib.rs*.

Comme nous n'avons pas de fichier *src/main.rs*, il n'y a rien à exécuter pour
Cargo avec la commande `cargo run`. C'est pourquoi nous allons utiliser la
commande `cargo build` pour compiler le code de notre crate de bibliothèque.

Nous examinerons plusieures façons d'organiser le code de notre bibliothèque
qui se prêterons à différentes situations, en fonction de la finalité du code.

### Définitions des Modules

For our `communicator` networking library, we’ll first define a module named
`network` that contains the definition of a function called `connect`. Every
module definition in Rust starts with the `mod` keyword. Add this code to the
beginning of the *src/lib.rs* file, above the test code:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }
}
```

After the `mod` keyword, we put the name of the module, `network`, and then a
block of code in curly brackets. Everything inside this block is inside the
namespace `network`. In this case, we have a single function, `connect`. If we
wanted to call this function from code outside the `network` module, we
would need to specify the module and use the namespace syntax `::`, like so:
`network::connect()` rather than just `connect()`.

We can also have multiple modules, side by side, in the same *src/lib.rs* file.
For example, to also have a `client` module that has a function named `connect`
as well, we can add it as shown in Listing 7-1:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }
}

mod client {
    fn connect() {
    }
}
```

<span class="caption">Listing 7-1: The `network` module and the `client` module
defined side by side in *src/lib.rs*</span>

Now we have a `network::connect` function and a `client::connect` function.
These can have completely different functionality, and the function names do
not conflict with each other because they’re in different modules.

In this case, because we’re building a library, the file that serves as the
entry point for building our library is *src/lib.rs*. However, in respect to
creating modules, there’s nothing special about *src/lib.rs*. We could also
create modules in *src/main.rs* for a binary crate in the same way as we’re
creating modules in *src/lib.rs* for the library crate. In fact, we can put
modules inside of modules, which can be useful as your modules grow to keep
related functionality organized together and separate functionality apart. The
choice of how you organize your code depends on how you think about the
relationship between the parts of your code. For instance, the `client` code
and its `connect` function might make more sense to users of our library if
they were inside the `network` namespace instead, as in Listing 7-2:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }

    mod client {
        fn connect() {
        }
    }
}
```

<span class="caption">Listing 7-2: Moving the `client` module inside the
`network` module</span>

In your *src/lib.rs* file, replace the existing `mod network` and `mod client`
definitions with the ones in Listing 7-2, which have the `client` module as an
inner module of `network`. Now we have the functions `network::connect` and
`network::client::connect`: again, the two functions named `connect` don’t
conflict with each other because they’re in different namespaces.

In this way, modules form a hierarchy. The contents of *src/lib.rs* are at the
topmost level, and the submodules are at lower levels. Here’s what the
organization of our example in Listing 7-1 looks like when thought of as a
hierarchy:

```text
communicator
 ├── network
 └── client
```

And here’s the hierarchy corresponding to the example in Listing 7-2:

```text
communicator
 └── network
     └── client
```

The hierarchy shows that in Listing 7-2, `client` is a child of the `network`
module rather than a sibling. More complicated projects can have many modules,
and they’ll need to be organized logically in order to keep track of them. What
“logically” means in your project is up to you and depends on how you and your
library’s users think about your project’s domain. Use the techniques shown
here to create side-by-side modules and nested modules in whatever structure
you would like.

### Moving Modules to Other Files

Modules form a hierarchical structure, much like another structure in computing
that you’re used to: filesystems! We can use Rust’s module system along with
multiple files to split up Rust projects so not everything lives in
*src/lib.rs* or *src/main.rs*. For this example, let’s start with the code in
Listing 7-3:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod client {
    fn connect() {
    }
}

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

<span class="caption">Listing 7-3: Three modules, `client`, `network`, and
`network::server`, all defined in *src/lib.rs*</span>

The file *src/lib.rs* has this module hierarchy:

```text
communicator
 ├── client
 └── network
     └── server
```

If these modules had many functions, and those functions were becoming lengthy,
it would be difficult to scroll through this file to find the code we wanted to
work with. Because the functions are nested inside one or more `mod` blocks,
the lines of code inside the functions will start getting lengthy as well.
These would be good reasons to separate the `client`, `network`, and `server`
modules from *src/lib.rs* and place them into their own files.

First, replace the `client` module code with only the declaration of the
`client` module, so that your *src/lib.rs* looks like code shown in Listing 7-4:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
mod client;

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

<span class="caption">Listing 7-4: Extracting the contents of the `client` module but leaving the declaration in *src/lib.rs*</span>

We’re still *declaring* the `client` module here, but by replacing the block
with a semicolon, we’re telling Rust to look in another location for the code
defined within the scope of the `client` module. In other words, the line `mod
client;` means:

```rust,ignore
mod client {
    // contents of client.rs
}
```

Now we need to create the external file with that module name. Create a
*client.rs* file in your *src/* directory and open it. Then enter the
following, which is the `connect` function in the `client` module that we
removed in the previous step:

<span class="filename">Filename: src/client.rs</span>

```rust
fn connect() {
}
```

Note that we don’t need a `mod` declaration in this file because we already
declared the `client` module with `mod` in *src/lib.rs*. This file just
provides the *contents* of the `client` module. If we put a `mod client` here,
we’d be giving the `client` module its own submodule named `client`!

Rust only knows to look in *src/lib.rs* by default. If we want to add more
files to our project, we need to tell Rust in *src/lib.rs* to look in other
files; this is why `mod client` needs to be defined in *src/lib.rs* and can’t
be defined in *src/client.rs*.

Now the project should compile successfully, although you’ll get a few
warnings. Remember to use `cargo build` instead of `cargo run` because we have
a library crate rather than a binary crate:

```text
$ cargo build
   Compiling communicator v0.1.0 (file:///projects/communicator)
warning: function is never used: `connect`
 --> src/client.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/lib.rs:4:5
  |
4 | /     fn connect() {
5 | |     }
  | |_____^

warning: function is never used: `connect`
 --> src/lib.rs:8:9
  |
8 | /         fn connect() {
9 | |         }
  | |_________^
```

These warnings tell us that we have functions that are never used. Don’t worry
about these warnings for now; we’ll address them later in this chapter in the
“Controlling Visibility with `pub`” section. The good news is that they’re just
warnings; our project built successfully!

Next, let’s extract the `network` module into its own file using the same
pattern. In *src/lib.rs*, delete the body of the `network` module and add a
semicolon to the declaration, like so:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
mod client;

mod network;
```

Then create a new *src/network.rs* file and enter the following:

<span class="filename">Filename: src/network.rs</span>

```rust
fn connect() {
}

mod server {
    fn connect() {
    }
}
```

Notice that we still have a `mod` declaration within this module file; this is
because we still want `server` to be a submodule of `network`.

Run `cargo build` again. Success! We have one more module to extract: `server`.
Because it’s a submodule—that is, a module within a module—our current tactic
of extracting a module into a file named after that module won’t work. We’ll
try anyway so you can see the error. First, change *src/network.rs* to have
`mod server;` instead of the `server` module’s contents:

<span class="filename">Filename: src/network.rs</span>

```rust,ignore
fn connect() {
}

mod server;
```

Then create a *src/server.rs* file and enter the contents of the `server`
module that we extracted:

<span class="filename">Filename: src/server.rs</span>

```rust
fn connect() {
}
```

When we try to `cargo build`, we’ll get the error shown in Listing 7-5:

```text
$ cargo build
   Compiling communicator v0.1.0 (file:///projects/communicator)
error: cannot declare a new module at this location
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
  |
note: maybe move this module `src/network.rs` to its own directory via `src/network/mod.rs`
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
note: ... or maybe `use` the module `server` instead of possibly redeclaring it
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
```

<span class="caption">Listing 7-5: Error when trying to extract the `server`
submodule into *src/server.rs*</span>

The error says we `cannot declare a new module at this location` and is
pointing to the `mod server;` line in *src/network.rs*. So *src/network.rs* is
different than *src/lib.rs* somehow: keep reading to understand why.

The note in the middle of Listing 7-5 is actually very helpful because it
points out something we haven’t yet talked about doing:

```text
note: maybe move this module `network` to its own directory via
`network/mod.rs`
```

Instead of continuing to follow the same file naming pattern we used
previously, we can do what the note suggests:

1. Make a new *directory* named *network*, the parent module’s name.
2. Move the *src/network.rs* file into the new *network* directory, and
   rename it to *src/network/mod.rs*.
3. Move the submodule file *src/server.rs* into the *network* directory.

Here are commands to carry out these steps:

```text
$ mkdir src/network
$ mv src/network.rs src/network/mod.rs
$ mv src/server.rs src/network
```

Now when we try to run `cargo build`, compilation will work (we’ll still have
warnings though). Our module layout still looks like this, which is exactly the
same as it did when we had all the code in *src/lib.rs* in Listing 7-3:

```text
communicator
 ├── client
 └── network
     └── server
```

The corresponding file layout now looks like this:

```text
├── src
│   ├── client.rs
│   ├── lib.rs
│   └── network
│       ├── mod.rs
│       └── server.rs
```

So when we wanted to extract the `network::server` module, why did we have to
also change the *src/network.rs* file to the *src/network/mod.rs* file and put
the code for `network::server` in the *network* directory in
*src/network/server.rs* instead of just being able to extract the
`network::server` module into *src/server.rs*? The reason is that Rust wouldn’t
be able to recognize that `server` was supposed to be a submodule of `network`
if the *server.rs* file was in the *src* directory. To clarify Rust’s behavior
here, let’s consider a different example with the following module hierarchy,
where all the definitions are in *src/lib.rs*:

```text
communicator
 ├── client
 └── network
     └── client
```

In this example, we have three modules again: `client`, `network`, and
`network::client`. Following the same steps we did earlier for extracting
modules into files, we would create *src/client.rs* for the `client` module.
For the `network` module, we would create *src/network.rs*. But we wouldn’t be
able to extract the `network::client` module into a *src/client.rs* file
because that already exists for the top-level `client` module! If we could put
the code for *both* the `client` and `network::client` modules in the
*src/client.rs* file, Rust wouldn’t have any way to know whether the code was
for `client` or for `network::client`.

Therefore, in order to extract a file for the `network::client` submodule of
the `network` module, we needed to create a directory for the `network` module
instead of a *src/network.rs* file. The code that is in the `network` module
then goes into the *src/network/mod.rs* file, and the submodule
`network::client` can have its own *src/network/client.rs* file. Now the
top-level *src/client.rs* is unambiguously the code that belongs to the
`client` module.

### Rules of Module Filesystems

Let’s summarize the rules of modules with regard to files:

* If a module named `foo` has no submodules, you should put the declarations
  for `foo` in a file named *foo.rs*.
* If a module named `foo` does have submodules, you should put the declarations
  for `foo` in a file named *foo/mod.rs*.

These rules apply recursively, so if a module named `foo` has a submodule named
`bar` and `bar` does not have submodules, you should have the following files
in your *src* directory:

```text
├── foo
│   ├── bar.rs (contains the declarations in `foo::bar`)
│   └── mod.rs (contains the declarations in `foo`, including `mod bar`)
```

The modules should be declared in their parent module’s file using the `mod`
keyword.

Next, we’ll talk about the `pub` keyword and get rid of those warnings!
