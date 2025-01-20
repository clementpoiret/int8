---
title: REPL-Driven Programming with Helix, Zellij, and DevEnv
# date: date '+%Y-%m-%dT%H:%M:%S%:z'
date: 2025-01-16T16:36:47+02:00
author: Clement POIRET
authorTwitter: clement_poiret1
cover: images/1_repl1.webp
tags: [programming, linux, guide]
keywords: [python, linux, nixos, guide, interactive]
description: description
showFullContent: false
readingTime: true
hideComments: false
toc: true
tocBorder: true
---

![Current Helix setup](/images/1_repl1.webp)

I have been using Neovim for ~3 years. As a lambda Linux ricer stuck in an infinite quest for the perfect setup to
achieve maximal productivity, I gave a try to Emacs. Few months later, even though I felt its incredible power, I missed
the snappiness of Neovim — yes, even with pgtk and [LSP Bridge](https://github.com/manateelazycat/lsp-bridge).

As my maximum time staying in the same config is around 42 hours, [Helix's 25.01](https://helix-editor.com/news/release-25-01-highlights/)
release was a great excuse to start a new journey. The first striking impression of Helix is its current lack of plugin
support (see [#3806](https://github.com/helix-editor/helix/discussions/3806)), meaning I would have to forget or rework
my usual developer experience, or would I?

In this post, I'll share how I achieved a seamless REPL-driven development workflow in Helix, leveraging Zellij and
DevEnv. We'll see how to create a powerful development environment that matches the experience I had with Emacs.

## Prerequisites

To follow along, you'll need:

- [Helix](https://helix-editor.com/),
- [Zellij](https://zellij.dev/),
- [DevEnv](https://devenv.sh/),
- [ripgrep](https://github.com/BurntSushi/ripgrep) (optional).

All code examples assume you have these tools installed and properly configured on your system.

## REPL-Drive Programming

Building AI models for a living, the language I spend the most writing in is Python, and I am pretty sure nearly all
Python developers are relying on some sort of REPL[^repl]-Driven Programming. To quote Jay Fields[^rdd], this workflow
allows you to answer the following questions:

1. Is my application doing what I believe it is?
1. What does this arbitrary code return when executed?
1. *[this one is mine]* What is the shape of this f\*cking matrix?

Of all the editors I tried, the experience on Emacs was clearly the best thanks to [Doom Emacs](https://github.com/doomemacs/doomemacs) builtins, and Emacs'
daemon, meaning I could have both a Python session and my code in different windows, i.e., different screens.

Having a decent REPL in Neovim implies extensions, such as [Iron](https://github.com/Vigemus/iron.nvim), or
[vimcmdline](https://github.com/jalvesaq/vimcmdline). This time, separating the REPL to another window required a
terminal multiplexer, such a [tmux](https://github.com/tmux/tmux) or [zellij](https://zellij.dev/). A terminal
multiplexer allows users to manage multiple terminal sessions within a single terminal window, with the ability to
detach and reattach sessions as needed.

But, Helix does not have plugins, what can we do to have a decent REPL workflow? By combining Helix's shell command
capabilities with Zellij's terminal management features.

## Helix and Zellij

Let's explore the Helix's and Zellij documentations to find our path to happiness. It turns out Helix exposes
[methods](https://docs.helix-editor.com/commands.html) to run shell commands: `:run-shell-command`, or `:sh`. The last
thing we require is the ability to pass text through it, the `:pipe-to` command.

Zellij exposes [cli actions](https://zellij.dev/documentation/cli-actions) to control any sessions via the CLI,
including what we require: `new-pane`, `move-focus`, and `write-chars`.

To implement this workflow, we'll need to configure both Zellij and Helix to work together.

### Configuration

#### Zellij

The first step is to avoid keybinding conflicts between Zellij and Helix. This can easily be done via Zellij's plugin
[zellij-autolock](https://github.com/fresh2dev/zellij-autolock), which can be used such as:

```json
plugins {
    autolock location="file:~/.config/zellij/plugins/zellij-autolock.wasm" {
        is_enabled true
        triggers "nvim|vim|git|fzf|zoxide|atuin|hx"
        reaction_seconds "0.3"
        print_to_log true
    }
    //...
}
// Load this "headless" plugin on start.
load_plugins {
    autolock
}

keybinds {
    // Keybindings specific to 'Normal' mode.
    normal {
        // Intercept `Enter`.
        bind "Enter" {
            // Passthru `Enter`.
            WriteChars "\u{000D}";
            // Invoke autolock to immediately assess proper lock state.
            // (This provides a snappier experience compared to
            // solely relying on `reaction_seconds` to elapse.)
            MessagePlugin "autolock" {};
        }
        //...
    }
    //...
}
```

Notice the magical part: `hx` in triggers. This will autolock Zellij as soon as you open Helix.

#### Helix

Once have Zellij ready to handle our dear Helix, let us define a few keybindings. First, we'll need basic keybindings such as
pane and tab creation, and movement between panes and tabs:

```toml
[keys.normal]
C-h = ":sh zellij ac move-focus-or-tab left"
C-j = ":sh zellij ac move-focus-or-tab down"
C-k = ":sh zellij ac move-focus-or-tab up"
C-l = ":sh zellij ac move-focus-or-tab right"

[keys.normal.C-a]
C-a = ":sh zellij ac toggle-floating-panes"
h = ":sh zellij ac new-pane -d down"
n = ":sh zellij ac new-pane"
r = [
  ":sh zellij ac new-pane -d right -- devenv shell repl",
  ":sh zellij ac move-focus left"
]
v = ":sh zellij ac new-pane -d right"
z = ":sh zellij ac toggle-fullscreen"

[keys.normal.C-t]
n = ":sh zellij ac new-tab"
```

Now that our REPL is alive, we just need to seed our current line, or selection, to the pane containing the REPL.
Unfortunately, without external Zellij plugin, it is not possible to write-chars to a named Zellij pane, so we have to
briefly focus the pane, send the chars, and focus the previous pane or tab:

```toml
[keys.insert]
C-esc = [
  "goto_first_nonwhitespace",
  "select_mode",
  "extend_to_line_end",
  ":sh zellij ac move-focus-or-tab right",
  ":pipe-to sh -c 'zellij ac write-chars \"$(cat)\n\"'",
  ":sh zellij ac move-focus-or-tab left",
  "collapse_selection",
  "insert_mode"
]
C-space = [
  "select_mode",
  "extend_to_line_bounds",
  ":sh zellij ac move-focus-or-tab right",
  ":pipe-to sh -c 'zellij ac write-chars \"$(cat)\n\"'",
  ":sh zellij ac move-focus-or-tab left",
  "collapse_selection",
  "insert_mode"
]

[keys.normal]
C-esc = [
  "goto_first_nonwhitespace",
  "select_mode",
  "extend_to_line_end",
  ":sh zellij ac move-focus-or-tab right",
  ":pipe-to sh -c 'zellij ac write-chars \"$(cat)\n\"'",
  ":sh zellij ac move-focus-or-tab left",
  "move_visual_line_down",
  "goto_first_nonwhitespace",
  "collapse_selection",
  "normal_mode"
]
C-space = [
  "select_mode",
  "extend_to_line_bounds",
  ":sh zellij ac move-focus-or-tab right",
  ":pipe-to sh -c 'zellij ac write-chars \"$(cat)\n\"'",
  ":sh zellij ac move-focus-or-tab left",
  "move_visual_line_down",
  "goto_first_nonwhitespace",
  "collapse_selection",
  "normal_mode"
]

[keys.select]
C-space = [
  ":sh zellij ac move-focus-or-tab right",
  ":pipe-to sh -c 'rg -v \"^[[:space:]]*$\" | zellij ac write-chars \"$(cat)\n\"'",
  ":sh zellij ac move-focus-or-tab left",
  "collapse_selection",
  "move_visual_line_down",
  "goto_first_nonwhitespace",
  "collapse_selection",
  "normal_mode"
]
```

The above configuration basically defines *C-space* and *C-esc*:

- *C-space* sends the current line or selection and preserves the indentation. This is useful to send functions.
- *C-esc* removes the indentation by passing the strings to [`ripgrep`](https://github.com/BurntSushi/ripgrep) first.
  If you don't have ripgrep installed, you can use `grep` instead. This is useful when you want to execute indented
  code.

So, your workflow can now be something like:

- Open your terminal emulator,
- `hx ~/projects/foo/bar.py`
- *<C-a>n* to open a new pane,
- `python`,
- *C-space* to send your lines into it.

You can even separate the REPL as a new tab, attach a new terminal to your session and switch to the second tab to
send lines to another window, like what I had with Emacs's daemon and client!

![Screenshot showing Helix editor on the left with Python code and a REPL session running in a separate window on the right, demonstrating multi-window REPL-driven development](/images/1_repl2.webp)

But notice the nice QoL keybinding we defined — *\<C-a>r* — calling `:sh zellij ac new-pane -d right -- devenv shell repl`.
[DevEnv](https://devenv.sh/) is an elegant way to manage projects and their dependencies, beyond what you can do with
tools like poetry or uv. It turns out we can use devenv to manager our python development environment, and define a `repl`
command to call whatever we want. This is a pleasant way to have a per-project varying shell command, which allows you
to use any kind of REPL, for any programming language!

#### DevEnv

Let's start with a dummy python project. We'll use the template I made to start new python projects quickly:
[https://github.com/clementpoiret/nix-python-devenv](https://github.com/clementpoiret/nix-python-devenv)

After that, we can define `scripts.repl.exec` to run any repl we want, here [ptpython](https://github.com/prompt-toolkit/ptpython):

```nix
{
  pkgs,
  lib,
  ...
}:
let
  buildInputs = with pkgs; [
    stdenv.cc.cc
    libuv
    zlib
  ];
in
{
  env = {
    LD_LIBRARY_PATH = "${lib.makeLibraryPath buildInputs}:/run/opengl-driver/lib:/run/opengl-driver-32/lib";
  };

  languages.python = {
    enable = true;
    uv = {
      enable = true;
      sync.enable = true;
    };
  };

  scripts.repl.exec = ''
    ptpython
  '';

  enterShell = ''
    . .devenv/state/venv/bin/activate
  '';
}
```

NB: don't forget to add `ptpython` to your `pyproject.toml` if you want to use it too:

```toml
[dependency-groups]
dev = [
    "ptpython>=3.0.29",
]
```

## Conclusion

Although this setup is not as optimal as it could because we can't write-chars to named panes in Zellij using CLI
actions, this is currently my daily driver, and it gives me joy, Helix feels even faster than Neovim. I just have to
work my muscle memory to integrate a new set of keybindings.

To see the config in action, here are my actual config files:

- [Zellij Config](https://github.com/clementpoiret/nixos-config/blob/f94a1b7d9f3fea8eed21f4768df603b8c3c2b01d/modules/home/zellij/config.kdl),
- [Helix Config](https://github.com/clementpoiret/nixos-config/blob/f94a1b7d9f3fea8eed21f4768df603b8c3c2b01d/modules/home/helix.nix).

I hope this post has been helpful :)

[^repl]: Short for Read-Eval-Print-Loop.

[^rdd]: [REPL-Driven Development](http://blog.jayfields.com/2014/01/repl-driven-development.html).
