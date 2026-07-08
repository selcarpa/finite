---
title: "How to Use reportlab's TTFont Font Fallback"
date: 2026-06-09T10:00:00+08:00
lastmod: 2026-06-09T10:00:00+08:00
authors: ["selcarpa"]
description: "How to configure TTFont font fallback in reportlab_enhanced for automatic font switching in multilingual text"
categories:
- docs
tags:
- reportlab
- pdf
- font
- python
toc:
    enable: true
    auto: true
---

## References

- [reportlab TTFont Font Fallback Implementation Analysis](/docs/reportlab-ttfont-fallback-impl/) — Companion article on implementation principles (from reportlab_enhanced)
- [reportlab-enhanced Docs - TrueType Font Fallback](https://reportlab-enhanced.tain.one/en/ch2a_fonts/#truetype)
- [reportlab-enhanced Docs](https://reportlab-enhanced.tain.one/)
- [selcarpa/reportlab_enhanced](https://github.com/selcarpa/reportlab_enhanced)

<!--more-->

## Introduction

[reportlab_enhanced](https://github.com/selcarpa/reportlab_enhanced) is a fork of [reportlab](https://www.reportlab.com/) that enhances font capabilities on top of the upstream. TTFont font fallback is one of its core improvements.

In the original reportlab, TTFont has always lacked a font fallback mechanism. When processing multilingual text (e.g., Latin + Chinese, Japanese, etc.), if the main font lacks glyphs for certain characters, the rendering is poor—displaying tofu boxes at best, or missing content at worst. reportlab_enhanced adds Type1-level fallback support for TTFont, which is enabled by default starting from version **0.1.0**.

This article explains how to use this feature. For implementation details, see [reportlab TTFont Font Fallback Implementation Analysis](/docs/reportlab-ttfont-fallback-impl/).

## Enabling the Feature

Starting with **reportlab_enhanced 0.1.0**, TTFont font fallback is enabled by default and requires no manual configuration.

> For earlier versions, set the environment variable `REPORTLAB_FONT_FALLBACK=1` to enable this feature.

## Basic Usage: Per-Font Configuration

```python
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont

# Register primary font (missing some glyphs)
latin_font = TTFont('NotoSans', 'NotoSans-Regular.ttf')

# Register fallback font (contains glyphs missing from primary)
cjk_font = TTFont('NotoSansCJK', 'NotoSansCJK-Regular.ttf')

pdfmetrics.registerFont(latin_font)
pdfmetrics.registerFont(cjk_font)

# Set fallback
latin_font.substitutionFonts = [cjk_font]

# Usage
canvas.setFont('NotoSans', 12)
canvas.drawString(100, 700, 'Hello 你好 World')  # Chinese chars auto-use NotoSansCJK
```

## Convenience Function: One-Line Setup

`registerFontWithFallback` handles registration and fallback setup in one step:

```python
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont

font = pdfmetrics.registerFontWithFallback(
    'NotoSans',
    'NotoSans-Regular.ttf',
    fallbackFonts=[
        TTFont('NotoSansCJK', 'NotoSansCJK-Regular.ttf')
    ]
)

canvas.setFont('NotoSans', 12)
canvas.drawString(100, 700, 'Hello 你好 World')
```

The `fallbackFonts` parameter accepts font name strings or TTFont instances:

```python
# Method 1: Font name strings
font = pdfmetrics.registerFontWithFallback(
    'NotoSans', 'NotoSans-Regular.ttf',
    fallbackFonts=['NotoSansCJK']  # Must be pre-registered via getFont()
)

# Method 2: TTFont instances
font = pdfmetrics.registerFontWithFallback(
    'NotoSans', 'NotoSans-Regular.ttf',
    fallbackFonts=[TTFont('NotoSansCJK', 'NotoSansCJK-Regular.ttf')]
)
```

## Checking Glyphs: `hasGlyph()`

To manually check if a character exists in a font:

```python
font = TTFont('NotoSans', 'NotoSans-Regular.ttf')

font.hasGlyph('A')       # True, Latin letters usually present
font.hasGlyph('你')       # False, primary font lacks CJK glyphs
font.hasGlyph(0x4F60)     # False, Unicode code point
font.hasGlyph(ord('你'))  # Same as above
```

## Multi-Level Fallback

Multiple fallback fonts can be configured in order:

```python
latin_font = TTFont('NotoSans', 'NotoSans-Regular.ttf')
cjk_font = TTFont('NotoSansCJK', 'NotoSansCJK-Regular.ttf')
emoji_font = TTFont('NotoEmoji', 'NotoEmoji.ttf')

latin_font.substitutionFonts = [cjk_font, emoji_font]

canvas.setFont('NotoSans', 12)
canvas.drawString(100, 700, 'Hello 你好 😄 World')  # CJK → NotoSansCJK, emoji → NotoEmoji
```

## Usage in Paragraph Text

The `<font>` tag is also supported:

```python
from reportlab.platypus import Paragraph, SimpleDocTemplate

doc = SimpleDocTemplate('output.pdf')
story = [
    Paragraph(
        '<font name="NotoSans">Hello 你好 World 世界</font>',
        style=ParagraphStyle(fontName='NotoSans', fontSize=14)
    )
]
doc.build(story)
```

Note: Explicitly specify the font name (configured with fallback) in the style or tag.

## Notes

1. **Feature from reportlab_enhanced** — This is a font enhancement from reportlab_enhanced on top of upstream reportlab. Enabled by default starting from version 0.1.0.
2. **Performance cost** — Each text rendering checks the fallback chain for glyphs missing from the primary font. May impact performance with large text volumes.

## Complete Example

```python
from reportlab.pdfgen import canvas
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont

font = pdfmetrics.registerFontWithFallback(
    'NotoSans', 'NotoSans-Regular.ttf',
    fallbackFonts=[
        TTFont('NotoSansCJK', 'NotoSansCJK-Regular.ttf')
    ]
)

c = canvas.Canvas('mixed_lang.pdf')
c.setFont('NotoSans', 16)
c.drawString(100, 800, 'ReportLab: Hello 你好 World 世界')
c.save()
```

In the generated PDF, Latin characters use NotoSans and CJK characters automatically use NotoSansCJK—no manual segmentation needed.

*AI-assisted article.*

> *This article is translated by deepseek-v4-flash (model: deepseek/deepseek-v4-flash).*
