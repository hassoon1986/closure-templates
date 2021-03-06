# Print Directives


Print directives are post-processing on the output of a [`print`
command](print.md).

Here are all print directives that are available by default. For information on
writing custom print directives, see [Plugins](../dev/plugins.md).

[TOC]

## Built-in directives

### `|bidiSpanWrap` {#bidiSpanWrap}

If the overall directionality of the print command is different from the global
directionality, then the compiler wraps the print command output in a span with
`dir=ltr` or `dir=rtl`.

NOTE: The template compiler applies autoescaping before evaluating
`|bidiSpanWrap`, which is safe because `|bidiSpanWrap` correctly handles
HTML-escaped text.

### `|bidiUnicodeWrap` {#bidiUnicodeWrap}

If the overall directionality the print command is different from the global
directionality, then the compiler wraps the print command output with Unicode
bidi formatting characters `LRE` or `RLE` at the start and `PDF` at the end.

NOTE: This directive serves the same purpose as `|bidiSpanWrap`, but you should
only use it in situations where HTML markup is not applicable, for example
inside an HTML `<option>` element.

### `|blessStringAsTrustedResourceUrlForLegacy` {#blessStringAsTrustedResourceUrlForLegacy}


Can be used to disable the autoescaper for sensitive resource urls. For example,

```soy
<script src={$myScriptUrl}>
```

When the compiler sees this, it inserts a runtime check to make sure that
`$myScriptUrl` is a trusted resource url. The
`|blessStringAsTrustedResourceUrlForLegacy` can be used to disable this as part
of a migration to strict autoescaping.

See [trusted_resource_uri](./security.md#trusted_resource_url) in the security
documentation for more information.

### `|changeNewlineToBr` {#changeNewlineToBr}

Changes newlines sequences: `\n`, `\r`, or `\r\n` to `<br>`.

### `|cleanHtml`, `|cleanHtml:'ul','li'` {#cleanHtml}

Removes all but a small, safe subset of HTML from its input. This converts other
[content kinds](../dev/security.md#content_kinds) (such as `text`) into `html`.
If content with the `html` kind is passed to this print directive then the
content is not changed.

Note that all attributes are removed except `dir` for directionality.

The default set of allowed tags is: `b`, `br`, `em`, `i`, `s`, `sub`, `sup`, and
`u`.

It may take a variable number of arguments which are additional tags to be
considered safe, only `ul`, `ol`, `li`, and `span` can be added to the
whitelist.

### `|insertWordBreaks:NNN` {#insertWordBreaks}

**WARNING**: This print directive is deprecated, and should not be used. Prefer
wrapping with CSS `word-wrap: break-word`.

This can insert `<wbr>` tags into content if there are more than `NNN`
characters between word breaks.

### `|noAutoescape` {#noAutoescape}

Disables the autoescaper for a particular print statement. This is disallowed
when using `autoescape="strict"` (the default), but may be used in older
templates.

This will allow all content through with the exception of content that has been
explicitly marked with `kind="text"` such content is explicitly unsafe and will
be replaced with the innocuous text `zSoyz`.

### `|truncate` {#truncate}

**IMPORTANT**: This print directive is generally unadvisable.

Truncates a string to a maximum length with trailing ellipsis. Add the optional
parameter: `false` to truncate without an ellipsis. For example, `{'Lorem Ipsum'
│truncate:8}` produces `Lorem...`, while `{'Lorem Ipsum' │truncate:8,false}`
produces `Lorem Ip`.

However, the simple truncation does not guarantee consistent visual results. For
example, in common fonts, the character "x" is about half the width of a Chinese
character, and "l" is half that. It's better to use CSS size constraints and
`text-overflow: ellipsis` when possible.

Furthermore, this print directive is no unicode sensitive so special characters
like emojis which are encoded using multiple UTF-16 code points, can be
corrupted when truncated.

### `|formatNum` {#formatNum}

Formats a number using the current locale.

It may take 4 optional arguments.

1.  A lower-case string describing the type of format to apply, which can be one
    of 'decimal', 'currency', 'percent', 'scientific', 'compact_short', or
    'compact_long'. If this argument is not provided, the default 'decimal' will
    be used.
1.  The "numbers" keyword passed to the ICU4J's locale. For instance, it can be
    "native" so that we show native characters in languages like arabic (this
    argument is ignored for templates running in JavaScript).
1.  The minimum number of fractional digits to display. If this is specified but
    the fourth parameter (maximum number of fractional digits), then this is
    interpreted as significant digits. If you wish to have trailing zeros
    removed, minFractionalDigits should be set to 0.
1.  The maximum number of fractional digits to display

NOTE: min and max fractional digits are not supported in the python backend.

For example:

*   `{$value|formatNum}`
*   `{$value|formatNum:'decimal'}`
*   `{$value|formatNum:'decimal','native'}`
*   `{$value|formatNum:'decimal','native', 2}`

### `|filterImageDataUri` {#filterImageDataUri}

Accepts only data URI's that contain an image.

Developers use this simultaneously to allow data URI's, but also to ensure that
the image tag won't initiate any HTTP requests.

NOTE: We may consider deprecating this now that img/data URIs are allowed by
default, since it's unlikely too many projects need a mechanism to double-check
that images are only loaded from data URIs; anyone else that does can simply
scan the URL and fail if it detects http/https.

### `|filterSipUri` {#filterSipUri}

Accepts only sip URIs but does not verify complete correctness.

The RFC for sip: https://tools.ietf.org/html/rfc3261

The RFC for URIs: https://tools.ietf.org/html/rfc3986

### `|filterTelUri` {#filterTelUri}

Accepts only 'tel://' URIs but does not verify complete correctness.

The RFC for the tel: URI https://tools.ietf.org/html/rfc3966

## Directives used primarily by the autoescaper

`Closure Templates` autoescaper internally uses many directives which are not
available to users. To trigger them, just print a variable in the appropriate
context, e.g. `<hr {$attributes}>`. Some of these directives are available to
users for legacy reasons but their usage is discouraged.

### `|escapeUri` {#escapeUri}

Escapes a value so it is safe to embed in a URI, for example in an `href`.

### `|normalizeUri` {#normalizeUri}

Allows arbitrary content to be included in a URI regardless of the string
delimiters of the surrounding language. This normalizes, but does not escape, so
it does not affect URI special characters, but instead escapes HTML, CSS, and JS
delimiters.
