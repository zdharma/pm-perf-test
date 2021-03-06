# Performance test of some of Zsh plugin managers

To run the test, execute:

```zsh
% ./run.zsh
```

However, there are problems with `zplug` when the test is run in this way – the
test hangs with a message:

```
zsh: suspended (tty input)  ./run.zsh
```

so an invocation of `fg` is required. To address this, run the test script as a
function:

```zsh
fpath+=( $PWD )
autoload run.zsh
run.zsh
```

## Drawing the results

To compute average results and draw the plots, run `plot.py` Python script. It
needs `matplotlib` Python package.

## Details of the test

Following 28 plugins are being used in the test:

```
zdharma/zsh-unique-id
Oh My Zsh / lib/git.zsh
Oh My Zsh / plugins/git/git.plugin.zsh
trapd00r/LS_COLORS
zdharma/zconvey
jhawthorn/fzy
junegunn/fzf-bin
urbainvaes/fzf-marks
hlissner/zsh-autopair
psprint/zsh-editing-workbench
psprint/zsh-navigation-tools
zdharma/history-search-multi-word
geometry-zsh/geometry
zdharma/zui
voronkovich/gitignore.plugin.zsh
zsh-users/zsh-autosuggestions
ogham/exa
psprint/vramsteg-zsh
psprint/revolver
psprint/zunit
zdharma/declare-zshrc
zdharma/zsh-diff-so-fancy
iwata/git-now
tj/git-extras
k4rthik/git-cal
zdharma/git-url
Fakerr/git-recall
arzzen/git-quick-stats.git
```

Some of the plugins are rather regular `Makefile`-based projects, like
`arzzen/git-quick-stats.git`. The `atclone''`, `make''`, ice modifiers of
Zinit and `hook-build` tag of Zplug allow to install and use them. However
they're problematic with `zgen`, which doesn't have such hooks. For it, instead
an empty plugin [zdharma/null](https://github.com/zdharma/null) is being loaded
in a following way:

```zsh
…

# git-recall
zgen load zdharma/null null.plugin.zsh empty-plugin.zsh-13

# git-quick-stats
zgen load zdharma/null null.plugin.zsh empty-plugin.zsh-14
```

the `empty-plugin.zsh-14`, etc. is a branch. The file `null.plugin.zsh` contains
only 3 instructions:

```zsh
0="${${ZERO:-${0:#$ZSH_ARGZERO}}:-${(%):-%N}}"
0="${${(M)0:#/*}:-$PWD/$0}"
true
```

this way a little balance is introduced into the test. IMO it's a good lower
boundary for the comparison – to have `zgen` only clone and load an almost empty
plugin instead of setting up a `PATH` for a command-like plugin, especially
because of the way that `zgen` works – by storing a plain `source` commands and
not executing any code.

This however puts `zgen` on an overall better position, also because it has to
clone an almost empty repository and not a full project, so also the
installation-time test is biased. However, it's hard to address this without
simplifying the test because of limited `zgen` functionality. Also, `zgen`
doesn't run the compilation (i.e. `make`) during the installation of the
plugins.

Zplug and Zinit tests are rather identical
([zshrc](https://github.com/zdharma/pm-perf-test/blob/master/zplug/.zshrc) for
Zplug,
[zshrc](https://github.com/zdharma/pm-perf-test/blob/master/zinit-load/.zshrc)
for Zinit).

## Results

![Installation times](https://raw.githubusercontent.com/zdharma/pm-perf-test/master/plots/installation-times.png)

![Startup times](https://raw.githubusercontent.com/zdharma/pm-perf-test/master/plots/startup-times.png)

## Result comments

The three different Zinit results needs explaining:

1. Zinit light – plugins are being loaded without tracking, i.e.: cannot be
   unloaded and their reports are being empty.

2. Zinit load – plugins are being loaded with tracking, i.e.: are available for
   unload and their report data is gathered (available through `zinit report
   {plugin-name}` command).

3. Zinit (Turbo) load – plugins are being loaded with tracking **and in Turbo
   mode** – i.e.: in background & after prompt – the shell is instantly ready to
   use.

<!-- vim:set ft=markdown tw=80 autoindent: -->
