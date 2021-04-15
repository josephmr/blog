+++
title = "Auto fill in org-mode"
author = ["Joseph Rollins"]
description = "Set auto-fill-mode to wrap columns in org-mode."
date = 2021-04-15
tags = ["tip"]
categories = ["emacs"]
draft = false
+++

When writing in org-mode I want my lines to automatically break when I reach a
certain column limit.


## org-fill-paragraph {#org-fill-paragraph}

My first solution was to call `org-fill-paragraph`, bound to `M-q` for each
paragraph or region I wanted to fix. This achieves the desired effect and the
length of lines can be configured via the `fill-column` variable:

> Column beyond which automatic line-wrapping should happen.
>
> It is used by filling commands, such as fill-region and fill-paragraph,
> and by auto-fill-mode, which see.

Manually calling this to fix paragraphs or regions was becoming annoying, luckily
there is a way to automate it.


## auto-fill-mode {#auto-fill-mode}

The `auto-fill-mode` function can be used to toggle auto fill mode for a buffer.

> Toggle automatic line breaking (Auto Fill mode).
>
> When Auto Fill mode is enabled, inserting a space at a column
> beyond current-fill-column automatically breaks the line at a
> previous space.

All that is needed is to enable this for org-mode via a hook so it affects all
org-mode buffers. The documentation for the `auto-fill-mode` function specified
to pass a positive value to enable:

> If called interactively, toggle Auto-Fill mode.  If the prefix
> argument is positive, enable the mode, and if it is zero or negative,
> disable the mode.

I use Doom so you may need to adjust your command slightly based on your configuration.

```elisp
(add-hook! org-mode
           (auto-fill-mode 1))
```

Now line breaks are automatically inserted as I type in org-mode buffers. It is
still occassionally necessary to manually call `org-fill-paragraph` when
editing; however, this covers the majority of the cases.
