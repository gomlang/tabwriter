# tabwriter

`ecosystem::tabwriter` aligns tab-delimited text using a bounded streaming formatter. Its module depends on `ecosystem::unicode_text` only for optional terminal-column measurement; all output is plain text.

```toml
[dependencies]
"ecosystem::tabwriter" = "0.1.0"
```

```gom
use ecosystem::tabwriter;

fn main() -> () {
    let writer = tabwriter::Writer::new(tabwriter::Options::new()).unwrap();
    print(writer.push("name\tcount\nlong\t2\n").unwrap());
    print(writer.flush().unwrap());
}
```

`push` accepts valid UTF-8 chunks and returns text as soon as a line without a tab ends an aligned block. Tabbed rows remain buffered until that boundary or `flush`, since later rows may widen a column. `flush` emits an unterminated tail and can be called repeatedly. LF and CRLF endings are preserved. A bare CR is ordinary cell content. Each tab separates a cell; every nonfinal cell is padded to the widest cell in its column, at least `min_width`, plus `padding` spaces. Plain lines pass through unchanged and reset alignment. There is no tab or escape sequence in formatted output.

`Options::new()` uses byte width, zero minimum width, one padding space, a 1 MiB buffered-input limit, and a 16 MiB output limit per `push` or `flush` call. `WidthPolicy::Terminal(unicode_text::WidthOptions)` instead measures grapheme display columns using the documented Unicode terminal policy. That policy does not parse ANSI control sequences or simulate tab stops. The byte policy counts UTF-8 bytes. Both policies preserve the original cell content. Width and padding are limited to 4096 columns; negative limits are rejected.

The formatter buffers a complete tabbed block and any incomplete line. A single unlimited tabbed block cannot be streamed with fixed alignment, so exceeding `max_buffer_bytes` returns an error. An error leaves the writer in a failed state; create a new writer before continuing. `format` is a convenience wrapper over `push` and `flush`. There is no native Go implementation or Go `text/tabwriter` API compatibility claim.

## Development and examples

Requires GoML 0.1.55 or newer. The `examples/basic/` example shares the root manifest. From the library root, run:

```sh
goml run --example basic
goml test
goml verify --timeout 300s
```

`goml test` builds the example and runs its tests. `goml verify` repeats the example checks as an independent module against an isolated registry snapshot. `(cd ../verification && just ecosystem-test tabwriter)` also retains the library-specific smoke and compatibility checks.
