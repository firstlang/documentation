# Comments

Comments are free-form text for human readers. They begin with `//` and continue to the end of the physical line.

```first
// Explain why this value is fixed.
retryLimit = 3 // Temporary limit while the service is in preview.
```

A comment may occupy its own line or follow code on the same line. The compiler ignores the comment text.

First has no block-comment syntax. In particular, JavaScript-style `/* ... */` comments are not supported. Use `//` on each line instead.

```first
// This explanation spans
// more than one line.
```

Comments are not anchors. Comments are informal notes for people and carry no program intent or instructions for language-model tooling. Use an anchor when text is intended to guide implementation or describe the meaning of nearby code.
