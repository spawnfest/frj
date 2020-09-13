# Spawnfest 2020

## Rationale and terminology

[Erlang LS](http://erlang-ls.github.io/) is an editor-agnostic
[language server](erlang-ls.github.io) which provides language
features for the Erlang programming language using the [Language
Server
Protocol](https://microsoft.github.io/language-server-protocol/), or
_LSP_ in short.

The [Debug Adapter
Protocol](https://microsoft.github.io/debug-adapter-protocol/), or
_DAP_ in short, is a similar protocol for communication between a
client (a text editor or IDE) and a _Debug Server_. The protocol is
similar to the _LSP_ one and it allows the creation of step-by-step
debuggers directly in the editor.

One of the strengths of the Erlang programming language is the ability
to _debug_ and _trace_ Erlang code. Many tools and libraries exist,
but they are sometimes under-utilized by the Community, either because
their API is not intuitive (think to the
[dbg](https://erlang.org/doc/man/dbg.html) Erlang module), or because
they offer a limited UI (think the
[debugger](http://erlang.org/doc/apps/debugger/debugger_chapter.html)
application).

We aim at leverage existing debugging and tracing facilities provided
by Erlang/OTP, bringing them directly to the user in the text-editor,
next to the code, raising the user experience level when using such
tools.

## Goals

We would like to use the opportunity provided by the _Spawnfest 2020_
to implement a _Proof of Concept_. In the _POC_ we demonstrate that it
is possible, with a relatively small effort, to raising the debugging
and tracing user experience for an Erlang user to newer, higher
usability standards.

In the _POC_ we are focusing on the creation of a step-by-step
debugger based on the [Erlang
Interpreter](http://erlang.org/doc/man/int.html), a not very well
known interface by the Community, but used as a low-level API to build
tools such as the OTP
[debugger](http://erlang.org/doc/man/debugger.html) and
[cedb](https://github.com/hachreak/cedb).

The same approach has been followed by the Elixir Community as part of
the [Elixir LS](https://github.com/elixir-lsp/elixir-ls) project. That
approach has been used as a reference to implement our solution.

During the weekend, we will try to:

* Get some familiarity with the _DAP_ protocol
* Get some familiarity with the Erlang Interpreter
* Integrate a basic step-by-step debugger using the two above in Erlang LS

# TODO

* Do not start dispatchers in DAP mode
* Pass DAP Methods Handler as a parameter
* Add and document dap-erlang (including IO tweaks)
* Evaluate the "Run in terminal" in addition to "Launch"
* Cleanup duplication in els\_dap\_methods
* Ony considering the "happy path" for now (eg regarding Erlang distribution)
* Tests

# Future upstream contributions

* DAP logs should end up in separate buffers as for LSP mode
* vscode hard-coded in initialize message
* Add Erlang to list of available Debug Adapters
* ConfigurationDone request does not respect capabilities
* The API of the _int_ meta process is internal and not well documented

## Emacs Config

```
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;; Erlang DAP                                               ;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

(require 'dap-mode)

(setq dap-inhibit-io nil)
(setq dap-print-io t)

(defun dap-erlang--populate-start-file-args (conf)
  "Populate CONF with the required arguments."
  (-> conf
      (dap--put-if-absent :dap-server-path '("els_dap"))
      (dap--put-if-absent :type "mix_task")
      (dap--put-if-absent :name "mix test")
      (dap--put-if-absent :request "launch")
      (dap--put-if-absent :task "test")
      (dap--put-if-absent :taskArgs (list "--trace"))
      (dap--put-if-absent :projectDir (lsp-find-session-folder (lsp-session) (buffer-file-name)))
      (dap--put-if-absent :cwd (lsp-find-session-folder (lsp-session) (buffer-file-name)))
      (dap--put-if-absent :requireFiles (list
                                         "test/**/test_helper.exs"
                                         "test/**/*_test.exs"))))

(dap-register-debug-provider "Erlang" 'dap-erlang--populate-start-file-args)
(dap-register-debug-template "Erlang Run Configuration"
                             (list :type "Erlang"
                                   :cwd nil
                                   :request "launch"
                                   :program "rebar3"
                                   :args "shell"
                                   :name "Erlang::Run"))
```

## daptoy

https://github.com/erlang-ls/daptoy