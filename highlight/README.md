Tree-sitter Highlight
=========================

[![Build Status](https://travis-ci.org/tree-sitter/tree-sitter.svg?branch=master)](https://travis-ci.org/tree-sitter/tree-sitter)
[![Build status](https://ci.appveyor.com/api/projects/status/vtmbd6i92e97l55w/branch/master?svg=true)](https://ci.appveyor.com/project/maxbrunsfeld/tree-sitter/branch/master)
[![Crates.io](https://img.shields.io/crates/v/tree-sitter-highlight.svg)](https://crates.io/crates/tree-sitter-highlight)

### Usage

Compile some languages into your app, and declare them:

```rust
extern "C" tree_sitter_html();
extern "C" tree_sitter_javascript();
```

Define the list of highlight names that you will recognize:

```rust
let highlight_names = [
    "attribute",
    "constant",
    "function.builtin",
    "function",
    "keyword",
    "operator",
    "property",
    "punctuation",
    "punctuation.bracket",
    "punctuation.delimiter",
    "string",
    "string.special",
    "tag",
    "type",
    "type.builtin",
    "variable",
    "variable.builtin",
    "variable.parameter",
]
.iter()
.cloned()
.map(String::from)
.collect();
```

Create a highlighter. You need one of these for each thread that you're using for syntax highlighting:

```rust
use tree_sitter_highlight::Highlighter;

let highlighter = Highlighter::new();
```

Load some highlighting queries from the `queries` directory of some language repositories:

```rust
use tree_sitter_highlight::HighlightConfiguration;

let html_language = unsafe { tree_sitter_html() };
let javascript_language = unsafe { tree_sitter_javascript() };

let html_config = HighlightConfiguration::new(
    html_language,
    &fs::read_to_string("./tree-sitter-html/queries/highlights.scm").unwrap(),
    &fs::read_to_string("./tree-sitter-html/queries/injections.scm").unwrap(),
    "",
).unwrap();

let javascript_config = HighlightConfiguration::new(
    javascript_language,
    &fs::read_to_string("./tree-sitter-javascript/queries/highlights.scm").unwrap(),
    &fs::read_to_string("./tree-sitter-javascript/queries/injections.scm").unwrap(),
    &fs::read_to_string("./tree-sitter-javascript/queries/locals.scm").unwrap(),
).unwrap();
```

Configure the recognized names:

```rust
javascript_config.configure(&highlight_names);
```

Highlight some code:

```rust
use tree_sitter_highlight::HighlightEvent;

let highlights = highlighter.highlight(
    &javascript_config,
    b"const x = new Y();",
    None,
    |_| None
).unwrap();

for event in highlights {
    match event? {
        HighlightEvent::Source {start, end} => {
            eprintln!("source: {}-{}", start, end);
        },
        HighlightEvent::HighlightStart(s) {
            eprintln!("highlight style started: {:?}", s);
        },
        HighlightEvent::HighlightEnd {
            eprintln!("highlight style ended");
        },
    }
}
```

The last parameter to `highlight` is a *language injection* callback. This allows other languages to be retrieved when Tree-sitter detects an embedded document (for example, a piece of JavaScript code inside of a `script` tag within HTML).
