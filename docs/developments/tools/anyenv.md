# Anyenv Tips

This is a simple wrapper for rbenv style environment managers. You don't have to git clone or modify your shell profile for each `**env` anymore if you install `anyenv`.

Oh, so you’re Japanese, just like me!

<!-- spell-checker: words anyenv rbenv pyenv nodenv ndenv anyenv-update -->

## Installing `anyenv`

> This is a record of:
>
> - installing anyenv master (3cb8ad1) on OSX 10.11, dated 2017-09-02
> - installing anyenv 1.1.1 from GitHub on Fedora 31, dated 2020-01-28

### Manual git checkout `anyenv`

Just as it is officially written.

- [riywo/anyenv \- github](https://github.com/riywo/anyenv)

<!-- spell-checker: words riywo -->

```bash
git clone https://github.com/anyenv/anyenv ~/.anyenv
```

```console
Cloning into '/home/akira/.anyenv'...
remote: Enumerating objects: 62, done.
remote: Counting objects: 100% (62/62), done.
remote: Compressing objects: 100% (41/41), done.
remote: Total 364 (delta 33), reused 44 (delta 21), pack-reused 302
Receiving objects: 100% (364/364), 65.13 KiB | 281.00 KiB/s, done.
Resolving deltas: 100% (150/150), done.
```

Set up anyenv in `.bash_profile`

~/.bash_profile ( ~/.bashrc on Ubuntu Desktop):

```bash
## add anyenv - 
export ANYENV_ROOT=${HOME}/.anyenv
if [ -d "${ANYENV_ROOT}" ]; then
    export PATH=${ANYENV_ROOT}/bin:$PATH
    eval "$(anyenv init -)"
fi
```

Set up anyenv in `.zshrc`:

```zsh
echo 'export PATH="$HOME/.anyenv/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(anyenv init -)"' >> ~/.zshrc
exec $SHELL -l   
```

```console
ANYENV_DEFINITION_ROOT(/Users/akira/.config/anyenv/anyenv-install) doesn't exist. You can initialize it by:
> anyenv install --init
```

```zsh
~/.anyenv/bin/anyenv init
```

Initialize install manifest directory:

```zsh
anyenv install --init

```console
Manifest directory doesn't exist: /Users/akira/.config/anyenv/anyenv-install
Do you want to checkout ? [y/N]: y
Cloning https://github.com/anyenv/anyenv-install.git master to /Users/akira/.config/anyenv/anyenv-install...
Cloning into '/Users/akira/.config/anyenv/anyenv-install'...
remote: Enumerating objects: 48, done.
remote: Total 48 (delta 0), reused 0 (delta 0), pack-reused 48
Unpacking objects: 100% (48/48), 6.65 KiB | 37.00 KiB/s, done.

Completed!
```

## Installing `anyenv-update`

I want to update the version of `**env` itself all at once.

> This is a record of:
>
> - installing anyenv-update master(b6cefdb) on Fedora 26, dated 2017-10-04

### Manual git checkout `anyenv-update`

This is also what is written on the site.

- [znz/anyenv-update \- github](https://github.com/znz/anyenv-update)

```bash
mkdir -p $(anyenv root)/plugins
git clone https://github.com/znz/anyenv-update.git $(anyenv root)/plugins/anyenv-update
```

```console
Cloning into '/home/akira/.anyenv/plugins/anyenv-update'...
remote: Enumerating objects: 87, done.
remote: Total 87 (delta 0), reused 0 (delta 0), pack-reused 87
Unpacking objects: 100% (87/87), done.
```

Run update with:

```bash
anyenv update
```

```console
Updating 'anyenv'...
Updating 'anyenv/anyenv-update'...
Updating 'ndenv'...
Updating 'ndenv/node-build'...
Updating 'pyenv'...
 |  From https://github.com/yyuu/pyenv
 |  fd0b8fc..5af8d2f  master     -> origin/master
 |  * [new branch]      skip-enable-unicode-py33 -> origin/skip-enable-unicode-py33
 |  * [new tag]         v1.1.5     -> v1.1.5
 |  * [new tag]         v1.1.4     -> v1.1.4
Skipping 'pyenv/python-build'; not git repo 

```

## Installing `pyenv`

```bash
anyenv install pyenv
```

```console
mp/pyenv.20200128232714.12813 ~
Cloning https://github.com/pyenv/pyenv.git master to pyenv...
Cloning into 'pyenv'...
remote: Enumerating objects: 16, done.
remote: Counting objects: 100% (16/16), done.
remote: Compressing objects: 100% (13/13), done.
remote: Total 17608 (delta 6), reused 7 (delta 3), pack-reused 17592
Receiving objects: 100% (17608/17608), 3.44 MiB | 3.07 MiB/s, done.
Resolving deltas: 100% (11958/11958), done.
~

Install pyenv succeeded!
Please reload your profile (exec $SHELL -l) or open a new session.
```

Apply to the login shell:

```bash
exec $SHELL -l
```

Confirm execution:

```console
$ pyenv --version

pyenv 1.2.16-5-g7097f820
```

### Installing `nodenv`

I thought `ndenv` was missing, but it was changed.

```bash
anyenv install nodenv
```

```console
/tmp/nodenv.20200128232947.13096 ~
Cloning https://github.com/nodenv/nodenv.git master to nodenv...
Cloning into 'nodenv'...
remote: Enumerating objects: 18, done.
remote: Counting objects: 100% (18/18), done.
remote: Compressing objects: 100% (13/13), done.
remote: Total 3915 (delta 4), reused 13 (delta 2), pack-reused 3897
Receiving objects: 100% (3915/3915), 696.59 KiB | 1.06 MiB/s, done.
Resolving deltas: 100% (2579/2579), done.
~
~/.anyenv/envs/nodenv/plugins ~
Cloning https://github.com/nodenv/node-build.git master to node-build...
Cloning into 'node-build'...
remote: Enumerating objects: 40, done.
remote: Counting objects: 100% (40/40), done.
remote: Compressing objects: 100% (29/29), done.
remote: Total 18353 (delta 13), reused 21 (delta 4), pack-reused 18313
Receiving objects: 100% (18353/18353), 3.21 MiB | 3.10 MiB/s, done.
Resolving deltas: 100% (11733/11733), done.
~
~/.anyenv/envs/nodenv/plugins ~
Cloning https://github.com/nodenv/nodenv-default-packages.git master to nodenv-default-packages...
Cloning into 'nodenv-default-packages'...
remote: Enumerating objects: 267, done.
remote: Counting objects: 100% (267/267), done.
remote: Compressing objects: 100% (151/151), done.
remote: Total 531 (delta 151), reused 202 (delta 105), pack-reused 264
Receiving objects: 100% (531/531), 113.88 KiB | 384.00 KiB/s, done.
Resolving deltas: 100% (272/272), done.
~
~/.anyenv/envs/nodenv/plugins ~
Cloning https://github.com/nodenv/nodenv-vars.git master to nodenv-vars...
Cloning into 'nodenv-vars'...
remote: Enumerating objects: 211, done.
remote: Total 211 (delta 0), reused 0 (delta 0), pack-reused 211
Receiving objects: 100% (211/211), 31.82 KiB | 276.00 KiB/s, done.
Resolving deltas: 100% (76/76), done.
~

Install nodenv succeeded!
Please reload your profile (exec $SHELL -l) or open a new session.
```

Apply to the login shell:

```bash
exec $SHELL -l
```

```console
$ nodenv --version

nodenv 1.3.1+11.5024679
```

### Installing `rbenv`

```bash
anyenv install rbenv
```

```console
/tmp/rbenv.20171004203339.4762 ~
Cloning https://github.com/rbenv/rbenv.git...
Cloning into 'rbenv'...
remote: Counting objects: 2629, done.
remote: Total 2629 (delta 0), reused 0 (delta 0), pack-reused 2629
Receiving objects: 100% (2629/2629), 488.96 KiB | 84.00 KiB/s, done.
Resolving deltas: 100% (1645/1645), done.
~
~/.anyenv/envs/rbenv/plugins ~
Cloning https://github.com/rbenv/ruby-build.git...
Cloning into 'ruby-build'...
remote: Counting objects: 7936, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 7936 (delta 0), reused 1 (delta 0), pack-reused 7934
Receiving objects: 100% (7936/7936), 1.67 MiB | 1.19 MiB/s, done.
Resolving deltas: 100% (4877/4877), done.
~

Install rbenv succeeded!
Please reload your profile (exec $SHELL -l) or open a new session.
```

Apply to the login shell:

```bash
exec $SHELL -l
```

```console
$ rbenv --version

rbenv 1.1.1-6-g2d7cefe
```
<!-- spell-checker: words: cefe -->

## Commands

### `anyenv global` - Show the global Any version

```bash
anyenv global
```

```console
ndenv: 6.11.2
pyenv: 3.6.6
rbenv: 2.4.2
```

### `anyenv version` - Show the global Any version

```bash
anyenv version
```

```console
ndenv: 6.11.2 (set by /home/akira/.anyenv/envs/ndenv/version)
pyenv: 3.6.6 (set by /home/akira/.anyenv/envs/pyenv/version)
rbenv: 2.4.2 (set by /home/akira/.anyenv/envs/rbenv/version)
```

### `anyenv versions` - List all Any versions

```bash
anyenv versions
```

```console
ndenv:
* 6.11.2 (set by /home/akira/.anyenv/envs/ndenv/version)
  8.4.0
pyenv:
  system
* 3.6.6 (set by /home/akira/.anyenv/envs/pyenv/version)
rbenv:
* 2.4.2 (set by /home/akira/.anyenv/envs/rbenv/version)
```
