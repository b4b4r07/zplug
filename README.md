[![](https://raw.githubusercontent.com/b4b4r07/screenshots/master/zplug/logo.png)][repo] [![](https://travis-ci.org/b4b4r07/zplug_test.svg?branch=master)][travis]
===

[![Join the chat at https://gitter.im/b4b4r07/zplug](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/b4b4r07/zplug?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

###### `zplug` is a next-generation plugin manager for zsh

## Pros.

- Manage everything (plugin, command, and gist file)
- Super-fast parallel installation/update
- Branch/tag/commit support
- Can manage UNIX commands
- Post-update hooks
- Support for externally managed plugins (e.g., [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh))
- Can manage binaries (e.g., GitHub Releases)
- Understands dependencies between plugins
- Unlike [antigen](https://github.com/zsh-users/antigen), no ZSH plugin support file (`*.plugin.zsh`) is needed
- Interactive interface ([fzf](https://github.com/junegunn/fzf), [peco](https://github.com/peco/peco), [zaw](https://github.com/zsh-users/zaw), and so on)

***DEMO:***

[![](https://raw.githubusercontent.com/b4b4r07/screenshots/master/zplug/demo.gif)][repo]

## Installation

Download [zplug](https://git.io/zplug) and put it in `~/.zplug`

```console
$ curl -fLo ~/.zplug/zplug --create-dirs git.io/zplug
```

## Usage

Add a zplug section to your `.zshrc`:

1. List the plugins/commands with `zplug` commands
2. `zplug load` to source the plugins and add its commands to your `$PATH`

### Example

[![](https://raw.githubusercontent.com/b4b4r07/screenshots/master/zplug/example.png)][repo]

```zsh
source ~/.zplug/zplug

# Make sure you use double quotes
zplug "zsh-users/zsh-syntax-highlighting"
zplug "zsh-users/zsh-history-substring-search"

# Can manage a plugin as a command
zplug "junegunn/dotfiles", as:command, of:bin/vimcat

# Manage everything e.g. zshrc (alias)
zplug "tcnksm/docker-alias", of:zshrc

# Prohibit updates to a plugin by using the "frozen:" tag
zplug "k4rthik/git-cal", as:command, frozen:1

# Grab binaries (from GitHub Releases)
# and rename to use "file:" tag
zplug "junegunn/fzf-bin", \
    as:command, \
    from:gh-r, \
    file:fzf

# Support oh-my-zsh
zplug "plugins/git",   from:oh-my-zsh
zplug "themes/duellj", from:oh-my-zsh
zplug "lib/clipboard", from:oh-my-zsh

# Run a command after a plugin is installed
zplug "tj/n", do:"make install"

# Support checking out a specific branch/tag/commit of a plugin
zplug "b4b4r07/enhancd", at:v1
zplug "mollifier/anyframe", commit:4c23cb60

# Install if "if:" tag returns true
zplug "hchbaw/opp.zsh", if:"(( ${ZSH_VERSION%%.*} < 5 ))"

# Gist can be used
zplug "b4b4r07/79ee61f7c140c63d2786", \
    from:gist, \
    as:command, \
    of:get_last_pane_path.sh

# Group dependencies, emoji-cli depends on jq in this example
zplug "stedolan/jq", \
    as:command, \
    file:jq, \
    from:gh-r \
    | zplug "b4b4r07/emoji-cli"

# Install plugins if there are plugins that have not been installed
if ! zplug check --verbose; then
    printf "Install? [y/N]: "
    if read -q; then
        echo; zplug install
    fi
fi

# Then, source plugins and add commands to $PATH
zplug load --verbose
```

Finally, use `zplug install` to install your plugins and reload `.zshrc`.

### 1. `zplug` commands

|  Command  | Description | Option |
|-----------|-------------|--------|
| `install` | Install described items (plugins/commands) in parallel | `--verbose`,`--select` |
| `load`    | Load installed items | `--verbose` |
| `list`    | List installed items | `--select` |
| `update`  | Update items in parallel | `--self`,`--select` |
| `check`   | Check whether an installation is available | `--verbose`,`--select` |
| `status`  | Check if the remote is up-to-date | `--select` |
| `clean`   | Remove repositories which are no longer used | `--force`,`--select` |

#### Take a closer look

```zsh
# zplug check return true if all plugins are installed
# Therefore, when it returns not true (thus false),
# run zplug install
if ! zplug check; then
    zplug install
fi

# source and add to the PATH
zplug load

# zplug check returns true if argument repository exists
if zplug check b4b4r07/enhancd; then
    # setting if enhancd is available
    export ENHANCD_FILTER=fzf-tmux
fi
```

#### Let zplug manage zplug

If you want to manage zplug by itself, to run this command (after installing zplug, of course):

```console
$ zplug update --self
```

By using `--self` option, zplug will be cloned to `$ZPLUG_HOME/repos` and be created symlink to `$ZPLUG_HOME/zplug`.

Then to start to manage zplug in the same way as any other plugins, please write the following in your `.zshrc`.

```zsh
zplug "b4b4r07/zplug"
```

All that's left is to run `zplug update`.

[![](https://raw.githubusercontent.com/b4b4r07/screenshots/master/zplug/update.gif)][repo]

### 2. `zplug` tags

| Tag | Description | Value (default) | Example |
|-----------|-------------|-----------------|---------|
| `as`      | Specify whether to register as commands or to register as plugins | `plugin`,`command` (`plugin`) | `as:command` |
| `of`      | Specify the pattern to source files (for `plugin`) or specify relative path to add to the `$PATH` (for `command`) | - (-) | `of:bin`,`of:*.zsh` |
| `from`    | Grab external binaries from e.g., GitHub Releases | `gh-r`,`gist`,`oh-my-zsh` (-) | `from:gh-r` |
| `at`      | Support branch/tag installation | branch/tag name (`master`) | `at:v1.5.6` |
| `file`    | Specify filename you want to rename | filename (-) | `file:fzf` |
| `dir`     | Installation directory | **READ ONLY** | -
| `if`      | Specify the conditions under which to perform (`source`, add `$PATH`) | true/false (-) | `if:"[ -d ~/.zsh ]"` |
| `do`      | Run commands after installation | commands (-) | `do:make` |
| `frozen`  | Do not update unless explicitly specified | 0,1 (0) | `frozen:1` |
| `commit`  | Support commit installation (regardless of whether the `$ZPLUG_SHALLOW` is true or not) | commit hash (-) | `commit:4428d48` |
| `on`      | Dependencies | **READ ONLY** | - |

#### Available on CLI

You can register plugins or commands to zplug on the command-line. If you use zplug on the command-line, it is possible to write more easily its settings by grace of the command-line completion.

[![](https://raw.githubusercontent.com/b4b4r07/screenshots/master/zplug/cli.gif)][repo]

In this case, zplug spit out its settings to `$ZPLUG_EXTERNAL` instead of `.zshrc`. If you launch new zsh process, `zplug load` command automatically search this file and run `source` command.

### 3. `zplug` configurations

#### `ZPLUG_HOME`

Defaults to `~/.zplug`.

`zplug` will store/load plugins in this directory. The directory structure is below.

```
$ZPLUG_HOME
|-- bin
|   `-- some_command -> ../repos/username_A/reponame1/some_command
`-- repos
    |-- username_A
    |   |-- reponame1
    |   |   |-- README.md
    |   |   `-- some_command
    |   `-- reponame2
    |       |-- README.md
    |       `-- some_plugin.zsh
    `-- username_B
        `-- reponame1
```

If you specify `as:command` in `zplug` command, zplug will recognize the plugin as a command and create a symbolic link of the same name (if you want to rename it, set `file:` tag) within `$ZPLUG_HOME/bin`. Because zplug adds `$ZPLUG_HOME/bin` to the `$PATH`, you can run that command from any directories.

#### `ZPLUG_THREADS`

The number of threads zplug should use. The default value is 16.

#### `ZPLUG_PROTOCOL`

Defaults to HTTPS. Valid options for `$ZPLUG_PROTOCOL` are HTTPS or SSH. Unless you have a specific reason, you should use the HTTPS protocol.

For more information, see also [**Which remote URL should I use?** - GitHub Help](https://help.github.com/articles/which-remote-url-should-i-use/)

#### `ZPLUG_SHALLOW`

Defaults to `true`. Makes zplug use shallow clone with a history truncated to the specified number of revisions (depth 1).

#### `ZPLUG_FILTER`

Defaults to `fzf-tmux:fzf:peco:percol:zaw`. When `--select` option is specified, colon-separated first element that exists in the `$PATH` will be used by zplug as the interactive filter. The `ZPLUG_FILTER` also accepts the following values: `fzf-tmux -d "10%":/path/to/peco:my peco`.

#### `ZPLUG_EXTERNAL`

Defaults to `$ZPLUG_HOME/init.zsh`. This file is used to add plugins from zplug on the command-line. Currently it's read-only.

## Note

- :tada: Congrats! Released v1.0.0 version!!
- ~~:construction: Until version 1.0.0 is released, `zplug` may be changed in ways that are not backward compatible.~~
- Not antigen :syringe: but **zplug** :hibiscus: will be here to stay from now on.
- :hibiscus: It was heavily inspired by [vim-plug](https://github.com/junegunn/vim-plug) and the like.

## Other resources

[awesome-zsh-plugins](https://github.com/unixorn/awesome-zsh-plugins) is a list of ZSH plugins, themes and completions that you can use with zplug.

## License

[MIT][license] © BABAROT

[repo]: https://github.com/b4b4r07/zplug
[travis]: https://travis-ci.org/b4b4r07/zplug_test
[license]: http://b4b4r07.mit-license.org