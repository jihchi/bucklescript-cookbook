# The BuckleScript Cookbook

The BuckleScript Cookbook is a collection of simple examples intended to both showcase BuckleScript by example, and to demonstrate good practices for accomplishing common tasks.

This has been heavily inspired by the [Rust Cookbook](https://brson.github.io/rust-cookbook/).

<!-- toc -->

- [Reason](#reason)
- [Contributing](#contributing)
- [General](#general)
    + [Serialize a record to JSON](#serialize-a-record-to-json)
    + [Deserialize JSON to a record](#deserialize-json-to-a-record)
    + [Encode and decode Base64](#encode-and-decode-base64)
    + [Generate random numbers](#generate-random-numbers)
    + [Log a message to the console](#log-a-message-to-the-console)
    + [Use string interpolation](#use-string-interpolation)
    + [Format a string using Printf](#format-a-string-using-printf)
    + [Extract specific HTML tags from an HTML document using a Regular Expression](#extract-specific-html-tags-from-an-html-document-using-a-regular-expression)
    + [Create a map data structure, add or replace an entry, and print each key/value pair](#create-a-map-data-structure-add-or-replace-an-entry-and-print-each-keyvalue-pair)
      - [Map](#map)
      - [Js.Dict](#jsdict)
      - [Associative list](#associative-list)
      - [Hashtbl](#hashtbl)
- [FFI](#ffi)
    + [Bind to a simple function](#bind-to-a-simple-function)
    + [Bind to a function in another module](#bind-to-a-function-in-another-module)
    + [Bind to a function overloaded to take an argument of several different types](#bind-to-a-function-overloaded-to-take-an-argument-of-several-different-types)
      - [Mutiple externals](#mutiple-externals)
      - [bs.unwrap](#bsunwrap)
      - [GADT](#gadt)
    + [Create a Plain Old JavaScript Object](#create-a-plain-old-javascript-object)
    + [Raise a javascript exception, then catch it and print its message](#raise-a-javascript-exception-then-catch-it-and-print-its-message)
    + [Define composable bitflags constants](#define-composable-bitflags-constants)
    + [Bind to a function that takes a variable number of arguments of different types](#bind-to-a-function-that-takes-a-variable-number-of-arguments-of-different-types)
- [Browser-specific](#browser-specific)
    + [Extract all links from a webpage](#extract-all-links-from-a-webpage)
    + [Query the GitHub API](#query-the-github-api)
- [Node-specific](#node-specific)
    + [Read lines from a text file](#read-lines-from-a-text-file)
    + [Read and parse a JSON file](#read-and-parse-a-json-file)
    + [Find files using a given predicate](#find-files-using-a-given-predicate)
    + [Run an external command](#run-an-external-command)
    + [Parse command-line arguments](#parse-command-line-arguments)

<!-- tocstop -->

## Reason

All examples in this document use [Reason](https://facebook.github.io/reason/) syntax.

## Contributing

There are primarily two ways to contribute:

1. Suggest an example to include in the cookbook by [creating an issue](https://github.com/glennsl/bucklescript-cookbook/issues/new) to describe the task.
2. Add (or edit) an example by [editing this file directly](https://github.com/glennsl/bucklescript-cookbook/edit/master/README.md) and creating a pull request.

## General

#### Serialize a record to JSON
Uses [bs-json](https://github.com/reasonml-community/bs-json)
```re
type line = {
  start: point,
  end_: point,
  thickness: option(int)
}
and point = {
  x: float,
  y: float
};

module Encode = {
  let point = (r) => {
    open! Json.Encode;
    object_([("x", float(r.x)), ("y", float(r.y))]);
  };
  let line = (r) =>
    Json.Encode.(
      object_([
        ("start", point(r.start)),
        ("end", point(r.end_)),
        (
          "thickness",
          switch r.thickness {
          | Some(x) => int(x)
          | None => null
          }
        )
      ])
    );
};

let data = {
  start: {
    x: 1.1,
    y: (-0.4)
  },
  end_: {
    x: 5.3,
    y: 3.8
  },
  thickness: Some(2)
};

let json = data |> Encode.line |> Js.Json.stringify;
```

#### Deserialize JSON to a record
Uses [bs-json](https://github.com/reasonml-community/bs-json)
```re
type line = {
  start: point,
  end_: point,
  thickness: option(int)
}
and point = {
  x: float,
  y: float
};

module Decode = {
  let point = (json) => {
    open! Json.Decode;
    {x: json |> field("x", float), y: json |> field("y", float)};
  };
  let line = (json) =>
    Json.Decode.{
      start: json |> field("start", point),
      end_: json |> field("end", point),
      thickness: json |> optional(field("thickness", int))
    };
};

let data = {| {
  "start": { "x": 1.1, "y": -0.4 },
  "end":   { "x": 5.3, "y": 3.8 }
} |};

let line = data |> Js.Json.parseExn |> Decode.line;
```

#### Encode and decode Base64

To encode and decode Base64, you can bind to Javascript functions `btoa` and `atob`, respectively:

```re
[@bs.val] external btoa : string => string = "";

[@bs.val] external atob : string => string = "";

let () = {
  let text = "Hello World!";
  Js.log(text |> btoa);
  Js.log(text |> btoa |> atob);
};
```

Alternatively, if you have [bs-webapi](https://github.com/reasonml-community/bs-webapi-incubator) installed:

```re
open Webapi.Base64;

let () = {
  let text = "Hello World!";
  Js.log(text |> btoa);
  Js.log(text |> btoa |> atob);
};
```

#### Generate random numbers

Use [Random module](http://caml.inria.fr/pub/docs/manual-ocaml/libref/Random.html) to generate random numbers

```re
let () = Js.log(Random.int(5));
```

#### Log a message to the console

```re
let () = Js.log("Hello BuckleScript!");
```

#### Use string interpolation

```re
let () =
  for (a in 1 to 10) {
    for (b in 1 to 10) {
      let product = a * b;
      Js.log({j|$a times $b is $product|j});
    };
  };
```

#### Format a string using Printf

Use [Printf module](http://caml.inria.fr/pub/docs/manual-ocaml/libref/Printf.html)

```re
/* Prints "Foo 2 bar" */
let () = Printf.printf("Foo %d %s", 2, "bar");
```

#### Extract specific HTML tags from an HTML document using a Regular Expression

```re
let input = {|
<html>
  <head>
    <title>A Simple HTML Document</title>
  </head>
  <body>
    <p>This is a very simple HTML document</p>
    <p>It only has two paragraphs</p>
  </body>
</html>
|};

let () =
  input
  |> Js.String.match([%re "/<p\\b[^>]*>(.*?)<\\/p>/gi"])
  |> (
    fun
    | Some(result) => result |> Js.Array.forEach(Js.log)
    | None => Js.log("no matches")
  );
```

#### Create a map data structure, add or replace an entry, and print each key/value pair

##### Map

Immutable, any key type, cross-platform

```re
let () = {
  module StringMap =
    Map.Make(
      {
        type t = string;
        let compare = compare;
      }
    );
  let painIndexMap =
    StringMap.(
      empty
      |> add("western paper wasp", 1.0)
      |> add("yellowjacket", 2.0)
      |> add("honey bee", 2.0)
      |> add("red paper wasp", 3.0)
      |> add("tarantula hawk", 4.0)
      |> add("bullet ant", 4.0)
    );
  painIndexMap
  |> StringMap.add("bumble bee", 2.0)
  |> StringMap.iter((k, v) => Js.log({j|key:$k, val:$v|j}));
};
```

##### Js.Dict

Mutable, string key type, BuckleScript only

```re
let painIndexMap =
  Js.Dict.fromList([
    ("western paper wasp", 1.0),
    ("yellowjacket", 2.0),
    ("honey bee", 2.0),
    ("red paper wasp", 3.0),
    ("tarantula hawk", 4.0),
    ("bullet ant", 4.0)
  ]);

let () = Js.Dict.set(painIndexMap, "bumble bee", 2.0);

let () =
  painIndexMap |> Js.Dict.entries |> Js.Array.forEach(((k, v)) => Js.log({j|key:$k, val:$v|j}));
```

##### Associative list

Immutable, any key type, cross-platform

```re
let painIndexMap = [
  ("western paper wasp", 1.0),
  ("yellowjacket", 2.0),
  ("honey bee", 2.0),
  ("red paper wasp", 3.0),
  ("tarantula hawk", 4.0),
  ("bullet ant", 4.0)
];

let addOrReplace = ((k, v), l) => {
  let l' = List.remove_assoc(k, l);
  [(k, v), ...l'];
};

let () =
  painIndexMap
  |> addOrReplace(("bumble bee", 2.0))
  |> List.iter(((k, v)) => Js.log({j|key:$k, val:$v|j}));
```

##### Hashtbl

Mutable, string key type, cross-platform

```re
let painIndexMap = Hashtbl.create(10);

let () = {
  open Hashtbl;
  add(painIndexMap, "western paper wasp", 1.0);
  add(painIndexMap, "yellowjacket", 2.0);
  add(painIndexMap, "honey bee", 2.0);
  add(painIndexMap, "red paper wasp", 3.0);
  add(painIndexMap, "tarantula hawk", 4.0);
  add(painIndexMap, "bullet ant", 4.0);
};

let () = Hashtbl.replace(painIndexMap, "bumble bee", 2.0);

let () = painIndexMap |> Hashtbl.iter((k, v) => Js.log({j|key:$k, val:$v|j}));
```

## FFI

#### Bind to a simple function

```re
[@bs.val] external random : unit => float = "Math.random";
```

#### Bind to a function in another module

```re
[@bs.val] [@bs.module "left-pad"] external leftpad : (string, int, char) => string = "";
```

#### Bind to a function overloaded to take an argument of several different types

##### Mutiple externals

```re
module Date = {
  type t;
  [@bs.new] external fromValue : float => t = "Date";
  [@bs.new] external fromString : string => t = "Date";
};

let date1 = Date.fromValue(107849354.);

let date2 = Date.fromString("1995-12-17T03:24:00");
```

##### bs.unwrap

```re
module Date = {
  type t;
  [@bs.new] external make : ([@bs.unwrap] [ | `Value(float) | `String(string)]) => t = "Date";
};

let date1 = Date.make(`Value(107849354.));

let date2 = Date.make(`String("1995-12-17T03:24:00"));
```

##### GADT

```re
module Date = {
  type t;
  type makeArg('a) =
    | Value: makeArg(float)
    | String: makeArg(string);
  [@bs.new] external make : ([@bs.ignore] makeArg('a), 'a) => t = "Date";
};

let date1 = Date.make(Value, 107849354.);

let date2 = Date.make(String, "1995-12-17T03:24:00");
```

#### Create a Plain Old JavaScript Object

```re
let person = [%obj name <- (first <- "Bob"; last <- "Zhmith"); age <- 32]
```

#### Raise a javascript exception, then catch it and print its message

```re
let () =
  try (Js.Exn.raiseError("oops!")) {
  | Js.Exn.Error(e) =>
    switch (Js.Exn.message(e)) {
    | Some(message) => Js.log({j|Error: $message|j})
    | None => Js.log("An unknown error occurred")
    }
  };
```

#### Define composable bitflags constants
TODO

#### Bind to a function that takes a variable number of arguments of different types

```re
module Arg = {
  type t;
  external int : int => t = "%identity";
  external string : string => t = "%identity";
};

[@bs.val] [@bs.splice] external executeCommand : (string, array(Arg.t)) => unit = "";

let () = executeCommand("copy", Arg.([|string("text/html"), int(2)|]));
```

## Browser-specific

#### Extract all links from a webpage

```re
open Webapi.Dom;

let printAllLinks = () =>
  document
  |> Document.querySelectorAll("a")
  |> NodeList.toArray
  |> Array.iter(
       (n) =>
         n
         |> Element.ofNode
         |> (
           fun
           | None => failwith("Not an Element")
           | Some(el) => Element.innerHTML(el)
         )
         |> Js.log
     );

let () = Window.setOnLoad(window, printAllLinks);
```

#### Query the GitHub API
Uses [bs-json](https://github.com/reasonml-community/bs-json) and [bs-fetch](https://github.com/reasonml-community/bs-fetch)

```re
/* given an array of repositories object as a JSON string */
/* returns an array of names */
let names = (text) => text |> Js.Json.parseExn |> Json.Decode.(array(field("name", string)));

/* fetch all public repositories of user [reasonml-community] */
/* print their names to the console */
let printGithubRepos = () =>
  Js.Promise.(
    Fetch.fetch("https://api.github.com/users/reasonml-community/repos")
    |> then_(Fetch.Response.text)
    |> then_((text) => text |> names |> Array.iter(Js.log) |> resolve)
    |> ignore
  );

let () = printGithubRepos();
```

## Node-specific

#### Read lines from a text file
Uses [bs-node](https://github.com/reasonml-community/bs-node)

```re
let () = Node.Fs.readFileAsUtf8Sync("README.md") |> Js.String.split("\n") |> Array.iter(Js.log);
```

#### Read and parse a JSON file
Uses [bs-json](https://github.com/reasonml-community/bs-json) and [bs-node](https://github.com/reasonml-community/bs-node)

```re
let decodeName = (text) => Js.Json.parseExn(text) |> Json.Decode.(field("name", string));

let () =
  /* read [package.json] file */
  Node.Fs.readFileAsUtf8Sync("package.json") |> decodeName |> Js.log;
```

#### Find files using a given predicate
Uses [bs-glob](https://github.com/reasonml-community/bs-glob)

```re
let () =
  /* find and list all javascript files in subfolders */
  Glob.glob("**/*.js", (_, files) => Array.iter(Js.log, files));
```

#### Run an external command
Uses [bs-node](https://github.com/reasonml-community/bs-node)

```re
let () =
  /* prints node's version */
  Node.(ChildProcess.execSync("node -v", Options.options(~encoding="utf8", ()))) |> Js.log;
```

#### Parse command-line arguments
TODO (requires bindings to minimist, commander or a similar library)
