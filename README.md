Weaver
======

An ergonomic command-line argument parser for the Roc language.

This library aims to provide a convenient interface for parsing CLI arguments
into structured data, in the style of Rust's [clap](https://github.com/clap-rs/clap).
Without code generation at compile time, the closest we can get in Roc is the use of the
[record builder syntax](https://www.roc-lang.org/examples/RecordBuilder/README.html).
This allows us to build our config and parser at the same time, in a type-safe way!

Read the documentation at <https://smores56.github.io/weaver/Cli/>.

## Status

An initial release has been made, so you can use Weaver right now! Just grab the download
URL from the [latest GitHub release](https://github.com/smores56/weaver/releases/latest)
and import it into your app.

More niceties will be added in the next few weeks, but the general structure of the library
is unlikely to change much unless I find a big improvement from a different parsing strategy.

## Example

```roc
app [main] {
    pf: platform "https://github.com/roc-lang/basic-cli/releases/download/0.10.0/vNe6s9hWzoTZtFmNkvEICPErI9ptji_ySjicO6CkucY.tar.br",
    weaver: "https://github.com/smores56/weaver/releases/download/0.1.0/MnJi0GTNzOI77qDnH99iuBNsM5ZKnc-gZTLFj7sIdqo.tar.br",
}

import pf.Stdout
import pf.Arg
import pf.Task exposing [Task]
import weaver.Opt
import weaver.Cli
import weaver.Param

main =
    args = Arg.list!

    when Cli.parseOrDisplayMessage cliParser args is
        Ok data ->
            Stdout.line! "Successfully parsed! Here's what I got:"
            Stdout.line! ""
            Stdout.line! (Inspect.toStr data)

        Err message ->
            Stdout.line! message

            Task.err (Exit 1 "")

cliParser =
    Cli.weave {
        force: <- Opt.flag { short: "f", help: "Force the task to complete." },
        file: <- Param.maybeStr { name: "file", help: "The file to process." },
        files: <- Param.strList { name: "files", help: "The rest of the files." },
    }
    |> Cli.finish {
        name: "basic",
        version: "v0.0.1",
        authors: ["Some One <some.one@mail.com>"],
        description: "This is a basic example of what you can build with Weaver. You get safe parsing, useful error messages, and help pages all for free!",
    }
    |> Cli.assertValid
```

And here's us calling the above example from the command line:

```console
$ roc readme.roc -- file1.txt file2.txt -f
Successfully parsed! Here's what I got:

{file: (Ok "file1.txt"), files: ["file2.txt"], force: Bool.true, sc: (Err NoSubcommand)}

$ roc readme.roc -- --help
basic v0.0.1
Some One <some.one@mail.com>

This is a basic example of what you can build with Weaver. You get safe parsing, useful error messages, and help pages all for free!

Usage:
  basic [OPTIONS] [file] [files]...
  basic <COMMAND>

Commands:
  s1  A first subcommand.
  s2  Another subcommand.

Arguments:
  [file]      The file to process.
  [files]...  The rest of the files.

Options:
  -f, --force    Force the task to complete.
  -h, --help     Show this help page.
  -V, --version  Show the version.
```

There are also some examples in the [examples](./examples) directory that are more feature-complete,
with more to come as this library matures.

## Roadmap

Now that an initial release has happened, these are some ideas I have for future development:

- [ ] Set default values in the arguments themselves
- [ ] Nested option records that all parse under a single command for better modularity
- [ ] Optionally set `{ group : Str }` per option so they are visually grouped in the help page
- [ ] Completion generation for popular shells (e.g. Bash, Zsh, Fish, etc.)
- [ ] Add terminal escape sequences to generated messages for prettier help/usage text formatting
- [ ] add convenient `Task` helpers (e.g. parse or print help and exit) once [module params](https://docs.google.com/document/u/0/d/110MwQi7Dpo1Y69ECFXyyvDWzF4OYv1BLojIm08qDTvg) land
- [ ] Clean up default parameter code if we can elide different fields on the same record type in different places (not currently allowed)
- [ ] Add more testing (always)
