[![MELPA](https://melpa.org/packages/verilog-ext-badge.svg)](https://melpa.org/#/verilog-ext)
[![MELPA Stable](https://stable.melpa.org/packages/verilog-ext-badge.svg)](https://stable.melpa.org/#/verilog-ext)
[![Build Status](https://github.com/gmlarumbe/verilog-ext/workflows/ERT-straight/badge.svg)](https://github.com/gmlarumbe/verilog-ext/actions/workflows/build_straight.yml)
[![Build Status](https://github.com/gmlarumbe/verilog-ext/workflows/package-el-basic/badge.svg)](https://github.com/gmlarumbe/verilog-ext/actions/workflows/build_package_melpa_basic.yml)
[![Build Status](https://github.com/gmlarumbe/verilog-ext/workflows/ERT-MELPA-Stable/badge.svg)](https://github.com/gmlarumbe/verilog-ext/actions/workflows/build_package_melpa_stable.yml)

# verilog-ext.el - SystemVerilog Extensions for Emacs #

This package provides useful extensions on top of [`verilog-mode`](https://github.com/veripool/verilog-mode)
and [`verilog-ts-mode`](https://github.com/gmlarumbe/verilog-ts-mode).

* [Tree-sitter `verilog-ts-mode` support](#tree-sitter)
* [Improve syntax highlighting](#syntax-highlighting)
* [Find definitions and references](#find-definitions-and-references)
* [Auto-completion with dot and scope completion](#auto-completion)
* [Hierarchy extraction and navigation](#hierarchy-extraction)
* [LSP configuration for `lsp-mode` and `eglot`](#language-server-protocol)
* [Support for many linters via `flycheck`](#linting)
* [Beautify modules and instances](#beautify-instances)
* [Code navigation functions for RTL and Verification environments](#navigation)
* [Extended collection of templates via `yasnippet` and `hydra`](#templates)
* [Code formatter via `apheleia`](#code-formatter)
* [Compilation with colored errors/warnings and jump to file/line](#compilation)
* [Improve `imenu` entries: detect instances, classes and methods](#imenu)
* [Enhanced support for `which-func`](#which-func)
* [Code folding via `hideshow`](#code-folding)
* [Highlight and align typedefs](#typedefs)
* [Auto-configure `time-stamp`](#time-stamp)
* [Auto-convert block end comments to names](#block-end-comments)
* [Port connection utilities](#port-connections)

## Requirements ##

- Emacs 29.1+
- Latest `verilog-mode` version
- Feature-specific binaries

Tree-sitter is optional but recommended and only required if using `verilog-ts-mode` for some of the features above.

For more info, see the [wiki](https://github.com/gmlarumbe/verilog-ext/wiki/Requirements).


## Installation ##

### MELPA ###

`verilog-ext` is available on MELPA.

### straight.el ###

To install it via [straight](https://github.com/radian-software/straight.el) with `use-package`:

```emacs-lisp
(straight-use-package 'use-package)
(use-package verilog-ext)
```

## Basic config ##

The most basic configuration just requires choosing which features you
want to load, setup the minor-mode and add it as a hook for `verilog-mode`.
By default all features are enabled:

```elisp
;; Can also be set through `M-x RET customize-group RET verilog-ext':
;; Comment out/remove the ones you do not need
(setq verilog-ext-feature-list
      '(font-lock
        xref
        capf
        hierarchy
        eglot
        lsp
        flycheck
        beautify
        navigation
        template
        formatter
        compilation
        imenu
        which-func
        hideshow
        typedefs
        time-stamp
        block-end-comments
        ports))
(require 'verilog-ext)
(verilog-ext-mode-setup)
(add-hook 'verilog-mode-hook #'verilog-ext-mode)
```

If installed and loaded via `use-package`:

```elisp
(use-package verilog-ext
  :hook ((verilog-mode . verilog-ext-mode))
  :init
  ;; Can also be set through `M-x RET customize-group RET verilog-ext':
  ;; Comment out/remove the ones you do not need
  (setq verilog-ext-feature-list
        '(font-lock
          xref
          capf
          hierarchy
          eglot
          lsp
          flycheck
          beautify
          navigation
          template
          formatter
          compilation
          imenu
          which-func
          hideshow
          typedefs
          time-stamp
          block-end-comments
          ports))
  :config
  (verilog-ext-mode-setup))
```

## Workspace ##

`verilog-ext` relies on the concept of a `workspace` to select which files will be analyzed
for navigation, completion, hierarchy extraction, compilation directory, etc...

For example, to analyze all the files inside a Git repo but excluding some 3rd party vendor libraries:

  ``` elisp
  (setq verilog-ext-workspace-ignore-dirs '("/path/to/my/repo/lib1" "/path/to/my/repo/lib2"))
  (setq verilog-ext-workspace-ignore-files '("/path/to/my/repo/lib3/lib_file.sv"))
  ```

To analyze only files inside rtl/tb directories inside a Git repo, plus a couple of specific ones in other directory:

  ``` elisp
  (setq verilog-ext-workspace-dirs '("/path/to/my/repo/src/rtl" "/path/to/my/repo/src/tb"))
  (setq verilog-ext-workspace-extra-files '("/path/to/my/repo/misc/specific_file1.sv" "/path/to/my/repo/misc/specific_file2.sv"))
  ```

To override the value of `workspace` root inside a Git repo:

  ``` elisp
  (setq verilog-ext-workspace-root-dir "/path/to/my/overriden/workspace/root")
  ```


## Keybindings ##

Enabling of `verilog-ext-mode` minor-mode creates the following keybindings:

* Features:
  * <kbd>C-c C-l</kbd> `verilog-ext-formatter-run`
  * <kbd>C-c \<f5\></kbd> `verilog-ext-workspace-compile`
  * <kbd>C-c C-p</kbd> `verilog-ext-preprocess`
  * <kbd>C-c C-f</kbd> `verilog-ext-flycheck-mode`
  * <kbd>C-c C-t</kbd> `verilog-ext-hydra/body`
  * <kbd>C-c C-v</kbd> `verilog-ext-hierarchy-current-buffer`
  * <kbd>C-\<tab\></kbd> `verilog-ext-hs-toggle-hiding`

* Code beautifying
  * <kbd>C-M-i</kbd> `verilog-ext-beautify-block-at-point-indent`
  * <kbd>C-c C-b</kbd> `verilog-ext-beautify-module-at-point`

* Dwim navigation
  * <kbd>C-M-a</kbd> `verilog-ext-nav-beg-of-defun-dwim`
  * <kbd>C-M-e</kbd> `verilog-ext-nav-end-of-defun-dwim`
  * <kbd>C-M-d</kbd> `verilog-ext-nav-down-dwim`
  * <kbd>C-M-u</kbd> `verilog-ext-nav-up-dwim`
  * <kbd>C-M-p</kbd> `verilog-ext-nav-prev-dwim`
  * <kbd>C-M-n</kbd> `verilog-ext-nav-next-dwim`

* Module at point
  * <kbd>C-c M-.</kbd> `verilog-ext-jump-to-module-at-point-def`
  * <kbd>C-c M-?</kbd> `verilog-ext-jump-to-module-at-point-ref`

* Jump to parent module
  * <kbd>C-M-.</kbd> `verilog-ext-workspace-jump-to-parent-module`

* Port connections
  * <kbd>C-c C-c c</kbd> `verilog-ext-ports-clean-blanks`
  * <kbd>C-c C-c t</kbd> `verilog-ext-ports-toggle-connect`
  * <kbd>C-c C-c r</kbd> `verilog-ext-ports-connect-recursively`

* Syntax table override functions:
  * <kbd>TAB</kbd> `verilog-ext-tab`
  * <kbd>M-d</kbd> `verilog-ext-kill-word`
  * <kbd>M-f</kbd> `verilog-ext-forward-word`
  * <kbd>M-b</kbd> `verilog-ext-backward-word`
  * <kbd>C-\<backspace\></kbd> `verilog-ext-backward-kill-word`


# Features #

## Tree-sitter ##

Some of the features that `verilog-ext` provides are based either on
builtin `verilog-mode` Emacs lisp parsing or on tree-sitter
`verilog-ts-mode`.  These features are hierarchy extraction and workspace
tags collection for completion and navigation of definitions and
references.

Using tree-sitter as a backend is recommended as it is much faster,
efficient and accurate than internal Emacs lisp parsing.

For information about installation of `verilog-ts-mode` check its
[repo](https://github.com/gmlarumbe/verilog-ts-mode).


## Syntax highlighting ##

<img src="https://user-images.githubusercontent.com/51021955/208774894-a0f3159e-0f41-45db-be28-8a8706ad49ec.gif" width=80%>

For configuration information, see the [wiki](https://github.com/gmlarumbe/verilog-ext/wiki/Syntax-highlighting).


## Find definitions and references ##

`verilog-ext` provides an `xref` backend to navigate definitions and references of the [workspace](#workspace).

<img src="https://github.com/gmlarumbe/verilog-ext/assets/51021955/d196a676-6d28-4bfa-9cee-2662d592b3fb" width=80%>

For configuration information, see the [wiki](https://github.com/gmlarumbe/verilog-ext/wiki/Xref).


## Auto-completion ##

Complete with tags from current [workspace](#workspace). Supports dot and scope completion for module signals, class attributes and methods.

<img src="https://github.com/gmlarumbe/verilog-ext/assets/51021955/7e0e6e49-8d5d-4be0-bb61-290c950e8623" width=80%>

For configuration information, see the [wiki](https://github.com/gmlarumbe/verilog-ext/wiki/Completion).


## Hierarchy extraction ##

Hierarchy extraction of module at current buffer.

<img src="https://github.com/gmlarumbe/verilog-ext/assets/51021955/94e009c3-e61c-496a-bacf-02e7d022157a" width=80%>

For configuration information, see the [wiki](https://github.com/gmlarumbe/verilog-ext/wiki/Hierarchy).


## Language Server Protocol ##

Auto-configure various SystemVerilog language servers for `lsp-mode` and `eglot`:

- [hdl_checker](https://github.com/suoto/hdl_checker)
- [svlangserver](https://github.com/imc-trading/svlangserver)
- [verible](https://github.com/chipsalliance/verible/tree/master/verilog/tools/ls)
- [svls](https://github.com/dalance/svls)
- [veridian](https://github.com/vivekmalneedi/veridian)

For configuration instructions, see the [wiki](https://github.com/gmlarumbe/verilog-ext/wiki/Language-Server-Protocol)

## Linting ##
Support via `flycheck` for the following linters:

* [Verilator](https://github.com/verilator/verilator)
* [Icarus Verilog](https://github.com/steveicarus/iverilog)
* [Verible](https://github.com/chipsalliance/verible/tree/master/verilog/tools/lint)
* [Slang](https://github.com/MikePopoloski/slang)
* [Svlint](https://github.com/dalance/svlint)
* [Surelog](https://github.com/chipsalliance/Surelog)
* Cadence HAL

For configuration and usage instructions, see the [wiki](https://github.com/gmlarumbe/verilog-ext/wiki/Linting)


## Beautify instances ##
Indent and align parameters and ports of RTL instances.

<img src="https://user-images.githubusercontent.com/51021955/208781782-dbf45c3e-df3f-405a-aacc-1d190ab87ae9.gif" width=80%>

Interactive functions:

* `verilog-ext-beautify-module-at-point`: <kbd>C-c C-b</kbd>
* `verilog-ext-beautify-current-buffer`

Batch-mode functions:

* `verilog-ext-beautify-files`
* `verilog-ext-beautify-dir-files`

## Navigation ##

Features:

* Context aware dwim functions for RTL/Verification environments
* Navigate instances inside a module
* Jump to definition/references of module at point
* Jump to parent module

<img src="https://user-images.githubusercontent.com/51021955/208782492-b2ff09b3-f662-4d22-a46c-64eb69f9f7b9.gif" width=80%>

For detailed info see the [wiki](https://github.com/gmlarumbe/verilog-ext/wiki/Navigation).


## Templates ##
Select among snippets that cover most frequently used SystemVerilog constructs:

<img src="https://user-images.githubusercontent.com/51021955/209577453-730014b7-d261-4884-9eb2-baa8eaa02a66.gif" width=80%>


Insert instances in current module from file:

<img src="https://user-images.githubusercontent.com/51021955/209577185-ad6b688d-158d-476f-94f5-e1d0eeb0fbd8.gif" width=80%>


Create basic testbench environment from DUT file:

<img src="https://user-images.githubusercontent.com/51021955/209578258-1db8eb6b-37ce-4be0-8cd6-ec380116d0cd.gif" width=80%>

UVM Agent template:

<img src="https://github.com/gmlarumbe/verilog-ext/assets/51021955/255e4b4b-afb7-4720-8b21-55796ae887eb" width=80%>

Functions:

* `verilog-ext-hydra/body`: <kbd>C-c C-t</kbd>


## Code formatter ##

Code-formatter setup via [apheleia](https://github.com/radian-software/apheleia) and [`verible-verilog-format`](https://github.com/chipsalliance/verible).

<img src="https://user-images.githubusercontent.com/51021955/220176079-f31ba086-7e64-434f-bb23-9c08e3f3ed6d.gif" width=80%>

See configuration in the [wiki](https://github.com/gmlarumbe/verilog-ext/wiki/Code-formatter).


## Compilation ##

Provides functions to perform compilations with syntax highlighting
and jump to error, buffer preprocessing and makefile development:

<img src="https://github.com/gmlarumbe/verilog-ext/assets/51021955/1a78cc1b-da3e-4219-baaf-cb1fb11d335c" width=80%>

  - `verilog-ext-workspace-compile`: <kbd>C-c \<f5\></kbd>
  - `verilog-ext-preprocess`: <kbd>C-c C-p</kbd>
  - `verilog-ext-workspace-makefile-create`
  - `verilog-ext-workspace-makefile-compile`

See configuration in the [wiki](https://github.com/gmlarumbe/verilog-ext/wiki/Compilation).


## Imenu ##
Support detection of instances and methods inside classes.

Instances:

<img src="https://user-images.githubusercontent.com/51021955/208779722-9b760d8d-796b-48cb-ad35-f95f1ec48786.gif" width=80%>

Methods:

<img src="https://user-images.githubusercontent.com/51021955/208780855-52166bf0-5897-48d1-83e8-698d0b1d6269.gif" width=80%>

Find more information [here](https://github.com/gmlarumbe/verilog-ext/wiki/Imenu).


## Which-func ##

Enhanced `which-func` support: show current block/instance at point in the mode-line

  <img src="https://user-images.githubusercontent.com/51021955/220174496-b35c99fd-2eb8-424b-9eca-49b9a1d6aa54.gif" width=80%>



## Code folding ##

 Code folding via `hideshow`: <kbd>C-\<tab\></kbd>

  <img src="https://user-images.githubusercontent.com/51021955/220174477-06beb019-3b2f-4329-8897-88e739ed5ea7.gif" width=80%>


## Typedefs ##

Add support for syntax-higlighting and alignment via
`verilog-pretty-declarations` of user defined types and classes.

<img src="https://github.com/gmlarumbe/verilog-ext/assets/51021955/5e654ba5-6eaa-4699-865c-628cadeda75a" width=80%>

For configuration see [wiki](https://github.com/gmlarumbe/verilog-ext/wiki/Typedefs)

## Time-stamp ##

Automatic update of header timestamp after file saving.

   - `verilog-ext-time-stamp-mode`

For configuration see [wiki](https://github.com/gmlarumbe/verilog-ext/wiki/Time-stamp)


## Block-end comments ##

Auto convert block comments to names after file saving.

   - `verilog-ext-block-end-comments-to-names-mode`


## Port connections ##

Toggle connections of ports under instance at point

  <img src="https://user-images.githubusercontent.com/51021955/220176192-d823ba19-099f-4484-abc7-8269fd92928b.gif" width=80%>

  * `verilog-ext-ports-toggle-connect`: <kbd>C-c C-c t</kbd>
  * `verilog-ext-ports-connect-recursively`: <kbd>C-c C-c r</kbd>
  * `verilog-ext-ports-clean-blanks`: <kbd>C-c C-c c</kbd>


## Misc ##

 Wrapper functions to stop cursor at underscores without breaking indentation

  - `verilog-ext-forward-word`: <kbd>M-f</kbd>
  - `verilog-ext-backward-word`: <kbd>M-b</kbd>
  - `verilog-ext-kill-word`: <kbd>M-d</kbd>
  - `verilog-ext-backward-kill-word`: <kbd>C-\<backspace\></kbd> and <kbd>M-DEL</kbd>


# Contributing #

Contributions are welcome! Just stick to common Elisp conventions and run the ERT suite after testing your changes and before submitting a new PR.

For new functionality add new ERT tests if possible.

Consider [sponsoring](https://github.com/sponsors/gmlarumbe) to help
maintaining the project and for the development of new features. *Thank you!*

## ERT Tests ###

### Setup ###

To run the whole ERT test suite change directory to the `verilog-ext`
root and make sure `test-hdl` Git submodule has been loaded:

```shell
git submodule update --init
```

### Targets ###

Then run the default target:

```shell
$ make
```

To run a subset of tests (e.g. navigation):

```shell
$ make TESTS=navigation
```

To regenerate all the expected outputs for the tests:

```shell
$ make gen
```

To regenerate the expected outputs for a group of tests (e.g. navigation):

```shell
$ make gen TESTS=navigation
```

## Other packages
* [verilog-ts-mode](https://github.com/gmlarumbe/verilog-ts-mode): SystemVerilog Tree-sitter mode
* [vhdl-ts-mode](https://github.com/gmlarumbe/vhdl-ts-mode): VHDL Tree-sitter mode
* [vhdl-ext](https://github.com/gmlarumbe/vhdl-ext): VHDL Extensions
* [fpga](https://github.com/gmlarumbe/fpga): FPGA & ASIC Utilities for tools of major vendors and open source
* [wavedrom-mode](https://github.com/gmlarumbe/wavedrom-mode): edit and render WaveJSON files to create timing diagrams
* [vunit-mode](https://github.com/embed-me/vunit-mode.git): Integration of [VUnit](https://github.com/VUnit/vunit) workflow
