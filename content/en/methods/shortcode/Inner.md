---
title: Inner
description: Returns the content between opening and closing shortcode tags, applicable when the shortcode call includes a closing tag.
categories: []
keywords: []
params:
  functions_and_methods:
    related:
      - functions/strings/Trim
      - methods/page/RenderString
      - functions/transform/Markdownify
      - methods/shortcode/InnerDeindent
    returnType: template.HTML
    signatures: [SHORTCODE.Inner]
---

This content:

{{< code file=content/services.md lang=text >}}
{{</* card title="Product Design" */>}}
We design the **best** widgets in the world.
{{</* /card */>}}
{{< /code >}}

With this shortcode:

{{< code file=layouts/shortcodes/card.html >}}
<div class="card">
  {{ with .Get "title" }}
    <div class="card-title">{{ . }}</div>
  {{ end }}
  <div class="card-content">
    {{ .Inner | strings.TrimSpace }}
  </div>
</div>
{{< /code >}}

Is rendered to:

```html
<div class="card">
  <div class="card-title">Product Design</div>
  <div class="card-content">
    We design the **best** widgets in the world.
  </div>
</div>
```

{{< note >}}
Content between opening and closing shortcode tags may include leading and/or trailing newlines, depending on placement within the Markdown. Use the [`strings.TrimSpace`] function as shown above to remove carriage returns and newlines.

[`strings.TrimSpace`]: /functions/strings/trimspace/
{{< /note >}}

{{< note >}}
In the example above, the value returned by `Inner` is Markdown, but it was rendered as plain text. Use either of the following approaches to render Markdown to HTML.
{{< /note >}}

## Use RenderString

Let's modify the example above to pass the value returned by `Inner` through the [`RenderString`] method on the `Page` object:

[`RenderString`]: /methods/page/renderstring/

{{< code file=layouts/shortcodes/card.html >}}
<div class="card">
  {{ with .Get "title" }}
    <div class="card-title">{{ . }}</div>
  {{ end }}
  <div class="card-content">
    {{ .Inner | strings.TrimSpace | .Page.RenderString }}
  </div>
</div>
{{< /code >}}

Hugo renders this to:

```html
<div class="card">
  <div class="card-title">Product design</div>
  <div class="card-content">
    We produce the <strong>best</strong> widgets in the world.
  </div>
</div>
```

You can use the [`markdownify`] function instead of the `RenderString` method, but the latter is more flexible. See&nbsp;[details].

[details]: /methods/page/renderstring/
[`markdownify`]: /functions/transform/markdownify/

## Alternative notation

Instead of calling the shortcode with the `{{</* */>}}` notation, use the `{{%/* */%}}` notation:

{{< code file=content/services.md lang=text >}}
{{%/* card title="Product Design" */%}}
We design the **best** widgets in the world.
{{%/* /card */%}}
{{< /code >}}

When you use the `{{%/* */%}}` notation, Hugo renders the entire shortcode as Markdown, requiring the following changes.

First, configure the renderer to allow raw HTML within Markdown:

{{< code-toggle file=hugo >}}
[markup.goldmark.renderer]
unsafe = true
{{< /code-toggle >}}

This configuration is not unsafe if _you_ control the content. Read more about Hugo's [security model].

Second, because we are rendering the entire shortcode as Markdown, we must adhere to the rules governing [indentation] and inclusion of [raw HTML blocks] as provided in the [CommonMark] specification.

{{< code file=layouts/shortcodes/card.html >}}
<div class="card">
  {{ with .Get "title" }}
  <div class="card-title">{{ . }}</div>
  {{ end }}
  <div class="card-content">

  {{ .Inner | strings.TrimSpace }}
  </div>
</div>
{{< /code >}}

The difference between this and the previous example is subtle but required. Note the change in indentation, the addition of a blank line, and removal of the `RenderString` method.

```diff
--- layouts/shortcodes/a.html
+++ layouts/shortcodes/b.html
@@ -1,8 +1,9 @@
 <div class="card">
   {{ with .Get "title" }}
-    <div class="card-title">{{ . }}</div>
+  <div class="card-title">{{ . }}</div>
   {{ end }}
   <div class="card-content">
-    {{ .Inner | strings.TrimSpace | .Page.RenderString }}
+
+  {{ .Inner | strings.TrimSpace }}
   </div>
 </div>
```

{{< note >}}
Don't process the `Inner` value with `RenderString` or `markdownify` when using [Markdown notation] to call the shortcode.

[Markdown notation]: /content-management/shortcodes/#notation
{{< /note >}}

[commonmark]: https://commonmark.org/
[indentation]: https://spec.commonmark.org/0.30/#indented-code-blocks
[raw html blocks]: https://spec.commonmark.org/0.30/#html-blocks
[security model]: /about/security/
