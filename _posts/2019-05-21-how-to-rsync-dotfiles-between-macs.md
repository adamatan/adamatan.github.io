---
layout: post
title: How to rsync dotfiles between Macs
permalink: /2019-05-21-how-to-rsync-dotfiles-between-macs
date: 2019-05-21
tags: rsync dotfile macos mac mabook mbp
---

Many CLI apps - like `git`, `vim`, `zsh` and `tmux` - use dot files to save their configuration. A *dot file* is a file or directory located in the user's home directory that begins with the "`.`" character.

Moving these files to a new laptop can save a lot of configuration time and keep a consistent development experience accross devices. **However, we want to avoid accidentally deleting existing dotfiles, and avoid copying junk dirs like `.Trash`.**.

# dotfiles

1. Make sure your computers are on the same network.
1. [Enable remote login on the source laptop](https://apple.stackexchange.com/a/2425/17965). The remote login window should list your username and address, e.g. `adam@adams-mbp` or `adam@192.168.120.130`.
![My helpful screenshot](/images/remote_login.png)
1. Create a temporary directory in the new computer and `cd` into it - you don't want to accidentally override existing dot files on the new computer.
1. Test:

    ```bash
    $ rsync --archive --verbose --dry-run -f"- */" 'adam@adams-mbp:~/.*' .
    ```
Giving something like:
    ```bash
    receiving file list ... done

    ...
    .zcompdump-Adam\#200\#231s MacBook Pro-5.2
    .zcompdump-Adam\#200\#231s MacBook Pro-5.3
    .zsh-update
    .zsh_history
    .zshrc
    ```
  * `--archive`: rsync [archive mode](https://serverfault.com/questions/141773/what-is-archive-mode-in-rsync) (essentially, preserve attributes).
  * `--verbose`: verbose: list files.
  * `--dry-run`: dry run. Just list, don't copy yet.
  * `-f"- */"`: Copy files, but not directories. This is important to avoid cache dirs like `.npm` or `.Trash`.
  * `'adam@adams-mbp:~/.*'`: Change `adam@adams-mbp` to the address given on the remote login screen. `.*` means all files beginning with `.`.
1. Once you're happy with the list, remove the `--dry-run` flag and copy the files. Selectively copy the dot files to your home directory. For example, you probably want to copy the `.zshrc` and `.zsh_history` file but avoid `.zcompdump` and `.zsh-update`.

# dot dirs

Some users have meaningful data in dot directories, for example `.aws` or `.ssh`. To selectively copy these directories, but avoid digital atrocities like copying `.Trash`, start with:

 ```bash
 rsync --archive --verbose --dry-run 'adam@adams-mbp:~/.*' .
 ```

 Start excluding junk directories, till you reach a reasonable list:
 ```bash
 rsync --archive --verbose --dry-run --exclude '.Trash' 'adam@adams-mbp:~/.*' .
 rsync --archive --verbose --dry-run --exclude '.Trash' --dry-run --exclude '.vscode' 'adam@adams-mbp:~/.*' .
 ```

 When done, copy the relevant dirs with `cp -R <dirname> ~` to your home (`~`) directory. **Double-check before overriding existing dirs, especially `.ssh`**.