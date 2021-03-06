Description
---------------

`org-roam-bibtex` is a library which offers a tighter integration between [`org-roam`](https://github.com/jethrokuan/org-roam), [`helm-bibtex`](https://github.com/tmalsburg/helm-bibtex), and [`org-ref`](https://github.com/jkitchin/org-ref).

It allows users to access their bibliographical notes in `org-roam-directory` via `helm-bibtex`, `ivy-bibtex`, or by opening `org-ref`’s `cite:` links and running `3. Add notes`.  If the note does not exist, it is created.

Demo
---------------

<img src="https://raw.githubusercontent.com/Zaeph/org-roam-bibtex/master/doc/demo.gif" width="700">

Installation
---------------

The package is not on MELPA yet, but a request has been filed.

For now, the only way to try the package is to clone the repository somewhere which `load-path` can access.

To do that:
1. Create a directory where you’d like to clone the repository, e.g. `mkdir ~/projects`.
2. `cd ~/projects`
3. `git clone https://github.com/Zaeph/org-roam-bibtex.git`

You now have the repository cloned in `~/projects/org-roam-bibtex/`.  See [Quick-start](#quick-start) to learn how to add it `load-path` and to get started with the package.

(You can also copy [`org-roam-bibtex.el`](https://github.com/Zaeph/org-roam-bibtex/blob/improve-readme/org-roam-bibtex.el) somewhere where `load-path` can access it, but you’d have to update the file manually.)

Quick-start
---------------

You can get `org-roam-bibtex` up and running by pasting the following sexps in your [init-file](https://www.gnu.org/software/emacs/manual/html_node/emacs/Init-File.html):

### With `use-package`
```el
(use-package org-roam-bibtex
  :load-path "~/projects/org-roam-bibtex/" ;Modify with your own path
  :config
  (org-roam-bibtex-mode))
  ```
  
### Without `use-package`
```el
(add-to-list 'load-path "~/projects/org-roam-bibtex/") ;Modify with your own path

(require 'org-roam-bibtex)

(org-roam-bibtex-mode)
```

Configuration
---------------

### Variables

### `org-roam-bibtex-template`

This variable specifies the template to use when creating a new note.  It builds on the syntax of `org-roam-capture-templates` by allowing you to expand predefined variables to the value of a BibTeX fields.

See `org-roam-bibtex-edit-notes` and [`org-roam-bibtex-preformat-keywords`](#org-roam-bibtex-preformat-keywords) for details.  (You can access the docstrings of any loaded function/variable from within Emacs with `C-h f` for functions/commands, and `C-h v` for variables.)

Here’s the default value of `org-roam-bibtex-template`:
```el
(("r" "ref" plain (function org-roam-capture--get-point) ""
      :file-name "${slug}"
      :head "#+TITLE: ${title}\n#+ROAM_KEY: ${ref}\n"
      :unnarrowed t))
```

You can modify it with `setq`.  For instance, if you want to add the cite-key in the title of the notes, you can modify the code like this (pay attention to the line with `:head`):
```el
(setq org-roam-bibtex-template
      '(("r" "ref" plain (function org-roam-capture--get-point) ""
         :file-name "${slug}"
         :head "#+TITLE: ${=key=}: ${title}\n#+ROAM_KEY: ${ref}\n" ; <--
         :unnarrowed t)))
```

### `org-roam-bibtex-preformat-keywords`

The template prompt wildcards for preformatting.  Only relevant when `org-roam-bibtex-preformat-templates` is set to `t` (default).  This can be a string, a list of strings or a cons-cell alist, where each element is `(STRING . STRING)`.

Use only alphanumerical characters, dash and underscore. See `org-roam-bibtex-edit-notes` for implementation details.

1. If the value is a string, a single keyword, it is treated as a BibTeX field name, such as `=key=`. In the following example all the prompts with the `=key=` keyword will be preformatted, as well as the corresponding match group `%\1`.

```el
(setq org-roam-bibtex-preformat-keywords "=key=")
(setq org-roam-capture-templates
      ’(("r" "reference" plain (function org-roam-capture--get-point)
         "#+ROAM_KEY: %^{=key=}%? fullcite: %\1"
         :file-name "references/${=key=}"
         :head "#+TITLE: ${title}"
         :unnarrowed t)))
```

2. If the value is a list of strings they are also treated as BibTeX field names. The respective prompts will be preformatted.

```el
(setq org-roam-bibtex-preformat-keywords ’("=key=" "title"))
(setq org-roam-capture-templates
      ’(("r" "reference" plain (function org-roam-capture--get-point)
         "#+ROAM_KEY: %^{=key=}%? fullcite: %\1"
         :file-name "references/${=key=}"
         :head "#+TITLE: ${title}"
         :unnarrowed t)))
```

3. If the value is a list of cons cells, then the car of the cons cell is treated as a prompt keyword and the cdr as a BibTeX field-name, and the latter will be used to retrieve the relevant value from the BibTeX entry. If cdr is omitted, then the car is treated as the field-name.

```el
(setq org-roam-bibtex-preformat-keywords
      '(("citekey" . "=key=")
       ("type" . "=type=")
       "title"))
(setq org-roam-capture-templates
      '(("r" "reference" plain (function org-roam-capture--get-point)
         "#+ROAM_KEY: %^{citekey}%? fullcite: %\1
          #+TAGS: %^{type}
          This %\2 deals with ..."
         :file-name "references/%<%Y-%m-%d-%H%M%S>_${title}"
         :head "#+TITLE: ${title}"
         :unnarrowed t)))
```

### `%(org-roam-bibtex-process-file-field \"${=key=}\")`

A convenience function has been added `org-roam-bibtex-process-file-field` that can be used as a `%sexp()` to find the documents associated with the bibtex entry and insert them into the template.

Below shows how this can be used to integrate with [org-noter](https://github.com/weirdNox/org-noter) or [interleave](https://github.com/rudolfochrist/interleave):

```el
(setq org-roam-bibtex-preformat-keywords
   '("=key=" "title" "url" "file" "author-or-editor" "keywords"))
  (setq org-roam-bibtex-template
        '(("r" "ref" plain (function org-roam-capture--get-point)
           ""
           :file-name "${slug}"
           :head "#+TITLE: ${=key=}: ${title}\n#+ROAM_KEY: ${ref}

\- tags ::
\- keywords :: ${keywords}

\n* ${title}\n  :PROPERTIES:\n  :Custom_ID: ${=key=}\n  :URL: ${url}\n  :AUTHOR: ${author-or-editor}\n  :NOTER_DOCUMENT: %(org-roam-bibtex-process-file-field \"${=key=}\")\n  :NOTER_PAGE: \n  :END:\n\n"
```



Consult the [`helm-bibtex`](https://github.com/tmalsburg/helm-bibtex) package for additional information about BibTeX field names.

Community
---------------
For help, support, or if you just want to hang out with us, you can find us here:
* **IRC**: channel **#org-roam** on [freenode](https://freenode.net/kb/answer/chat)
