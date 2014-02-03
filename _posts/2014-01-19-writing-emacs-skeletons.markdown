---
layout: post
title:  "Writing Emacs skeletons"
---

I use Emacs to write markdown (GitHub flavored markdown) and have
written quite a bit of it the last couple of months. So far I haven't
used any special tools beside the Emacs markdown-mode.

I was wondering if Emacs might be able to help me out, and I started
looking at different Emacs modes and quickly discovered that they use
something called tempo (or tempo-mode) to generate code skeletons.

There is an [entry](http://www.emacswiki.org/emacs/TempoMode) about
tempo on EmacsWiki, but frankly, the examples I tested there just
didn't work. I don't know if the documentation hasn't been properly
updated. I am using Emacs 24.3.1.

Before doing anything else, don't forget to load the tempo package:
`(require 'tempo)`.

From the EmacsWiki and the Emacs modes I quicky learned that the
important function for defining a template was the
`tempo-define-template` function, which has a compact but good
description (`C-h f tempo-define-template`) on how to write code
templates.

Consider the following example:

```scheme

(tempo-define-template
 "markdown-highlight-erlang"
 '("```erlang"
   n r p n
   "```" n ))

```

When this is executed, `tempo-define-template` generates the function
`tempo-template-markdown-highlight-erlang`. Invoking the generated
function will insert the following at point, placing the cursor in
between the begin and end markers:

    ```erlang

    ```

The above is an example of GitHub markdown, marking a piece of text as
erlang code and highlighting it as such.

The meaning of the special characters `n`, `r` and `p` in this example are:

`p`: Where to place the cursor after inserting the template.

`r`: If a region is selected, replace the region with the template and
insert the region at this point.

`n`: Insert newline.

Note: this is not the complete meaning of the above symbols, and there
are several more like `>` (meaning indent line). Consult the
description of `tempo-define-template` for more information.

### Mode activation ###

We need to add this to the context of the mode where we will use the
skeleton. We can add the following to our `.emacs`:

```scheme

(defun my-markdown-model-keys ()
  "Add my keybindings to `markdown-mode'."
  (local-set-key [f5] 'tempo-template-markdown-highlight-erlang))

(add-hook 'markdown-mode-hook 'my-markdown-model-keys)

```

Here we define a function `my-markdown-model-keys` which locally binds
`f5` to the generated template function
`tempo-template-markdown-highlight-erlang`, and then we add the
function to the `markdown-mode-hook`.
