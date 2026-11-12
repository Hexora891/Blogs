---
title: "Code Blocks, Captions, and Other Hugo Tricks"
date: 2026-03-20T14:15:00-05:00
description: "A quick tour of the Markdown features this theme adds on top of plain Hugo."
draft: false
images:
  - /images/dev-image-03.jpeg
categories:
  - Hugo
tags:
  - hugo
  - markdown
  - theme
---

The Kana theme does not change how you write Markdown, but it does add a few helpful extras. Here are
the ones I use most.

## Syntax highlighting with a filename

Add a `file` attribute to a fenced code block and the theme will label it:

```go {file="main.go"}
package main

import "fmt"

func main() {
    fmt.Println("Hello from a Hugo code block")
}
```

The highlighting uses the Gruvbox palette.

## Collapsible code blocks

Long snippets can be folded away with `details=true`:

```bash {details=true}
hugo new content posts/2026/my-new-post.md
hugo server --buildDrafts
hugo --minify
```

## Responsive images

Images placed in `assets/` can be inserted with the `asset-image` shortcode, which handles WebP
conversion and sizing for you:

```text
{{</* asset-image "images/dev-image-03.jpeg" "600" "400" "A caption" */>}}
```

## That's it

Everything else is standard Markdown. Headings, lists, links, blockquotes, tables — they all just
work.
