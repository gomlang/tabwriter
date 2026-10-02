# tabwriter

`ecosystem::tabwriter` aligns tab-delimited text using a bounded streaming formatter. Its module depends on `ecosystem::unicode_text` only for optional terminal-column measurement; all output is plain text.

```toml
[dependencies]
"ecosystem::tabwriter" = "0.1.0"
```

```goml
use ecosystem::tabwriter;

fn main() -> () {
    let writer = tabwriter::Writer::new(tabwriter::Options::new()).unwrap();
    print(writer.push("name\tcount\nlong\t2\n").unwrap());
    print(writer.flush().unwrap());
}
```

`push` accepts valid UTF-8 chunks and returns text as soon as a line without a tab ends an aligned block. Tabbed rows remain buffered until that boundary or `flush`, since later rows may widen a column. `flush` emits an unterminated tail and can be called repeatedly. LF and CRLF endings are preserved. A bare CR is ordinary cell content. Each tab separates a cell; every nonfinal cell is padded to the widest cell in its column, at least `min_width`, plus `padding` spaces. Plain lines pass through unchanged and reset alignment. Tab separators become spaces. The formatter emits no ANSI sequences of its own; existing escape and control bytes in cell content are preserved. Sanitize untrusted cells before writing the result to a terminal.

`Options::new()` uses byte width, zero minimum width, one padding space, a 1 MiB buffered-input limit, and a 16 MiB output limit per `push` or `flush` call. `WidthPolicy::Terminal(unicode_text::WidthOptions)` instead measures grapheme display columns using the documented Unicode terminal policy. That policy does not parse ANSI control sequences or simulate tab stops. The byte policy counts UTF-8 bytes. Both policies preserve the original cell content. Width and padding are limited to 4096 columns; negative limits are rejected.

The formatter buffers a complete tabbed block and any incomplete line. Incomplete
lines append to a reusable byte buffer; each incoming chunk is scanned once,
without rescanning or copying the entire retained prefix on every `push`. CRLF
recognition works even when CR and LF arrive in different calls. A single unlimited tabbed block cannot be streamed with fixed alignment, so exceeding `max_buffer_bytes` returns an error. An error leaves the writer in a failed state; create a new writer before continuing. `format` is a convenience wrapper over `push` and `flush`. There is no native Go implementation or Go `text/tabwriter` API compatibility claim.

## Development and examples

Requires GoML 0.1.56 or newer. The `examples/basic/` example shares the root manifest. From the library root, run:

```sh
goml run --example basic
goml test
goml verify --timeout 300s
```

`goml test` builds the example and runs its tests. `goml verify` repeats the example checks as an independent module against an isolated registry snapshot. `(cd ../verification && just ecosystem-test tabwriter)` also retains the library-specific smoke and compatibility checks.

For an opt-in fragmented-line scaling measurement, run
`goml test --ignored --nocapture fragmented_line_scaling`. It reports three
complete runs at 8 KiB and 16 KiB without imposing timing thresholds.
