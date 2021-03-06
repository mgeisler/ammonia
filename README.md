HTML Sanitization
=================

[![Build Status](https://travis-ci.org/notriddle/ammonia.svg?branch=master)](https://travis-ci.org/notriddle/ammonia)
[![Crates.IO](https://img.shields.io/crates/v/ammonia.svg)](https://crates.io/crates/ammonia)
[![Docs.RS](https://docs.rs/ammonia/badge.svg)](https://docs.rs/ammonia/)


Ammonia is a whitelist-based HTML sanitization library. It is designed to
prevent cross-site scripting, layout breaking, and clickjacking caused
by untrusted user-provided HTML being mixed into a larger web page.

Ammonia uses [html5ever] to parse and serialize document fragments the same way browsers do,
so it is extremely resilient to syntactic obfuscation.

Ammonia parses its input exactly according to the HTML5 specification;
it will not linkify bare URLs, insert line or paragraph breaks, or convert `(C)` into &copy;.
If you want that, use a markup processor before running the sanitizer, like [pulldown-cmark].

[html5ever]: https://github.com/servo/html5ever "The HTML parser in Servo"
[pulldown-cmark]: https://github.com/google/pulldown-cmark


Changes
-----------
Please see the [CHANGELOG](CHANGELOG.md) for a release history.


Example
-------

Using [pulldown-cmark] together with Ammonia for a friendly user-facing comment
site.

```rust
extern crate pulldown_cmark;
extern crate ammonia;
use pulldown_cmark::{push_html, Parser};
use ammonia::clean;
let text = "[a link](http://www.notriddle.com/)";
let mut md_parse = Parser::new_ext(text, OPTION_ENABLE_TABLES);
let mut unsafe_html = String::new();
push_html(&mut unsafe_html, md_parse);
let safe_html = clean(&*unsafe_html);
assert_eq!(safe_html, "<a href=\"http://www.notriddle.com/\">a link</a>");
```


Performance
-----------

Ammonia builds a DOM, traverses it (replacing unwanted nodes along the way),
and serializes it again. It could be faster for what it does, and if you don't
want to allow any HTML it is possible to be even faster than that.

However, it takes about fifty times longer to sanitize an HTML string using
[Bleach] than it does using Ammonia.

    $ cd benchmarks
    $ cargo run --release
        Running `target/release/ammonia_bench`
    56829 nanoseconds to clean up the intro to the Ammonia docs.
    $ python3 bleach_bench.py
    2910792.875289917 nanoseconds to clean up the intro to the Ammonia docs.


Thanks
------

Thanks to the other sanitizer libraries, particularly [Bleach] for Python and [sanitize-html] for Node,
which we blatantly copied most of our API from.

Thanks to ChALkeR, whos [Improper Markup Sanitization] document helped us find high-level semantic holes in Ammonia.

And finally, thanks to [the contributors].


[sanitize-html]: https://www.npmjs.com/package/sanitize-html
[Bleach]: https://github.com/jsocol/bleach
[Improper Markup Sanitization]: https://github.com/ChALkeR/notes/blob/master/Improper-markup-sanitization.md
[the contributors]: https://github.com/notriddle/ammonia/graphs/contributors
