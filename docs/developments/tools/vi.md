# Vi and Vim Tips

## Introduction

vi (VIsual editor) was developed by William Nelson Joy, the founder of BSD, to comfortably create the Pascal compiler he was developing when releasing the first BSD.

Vim (Vi IMproved) was born when Dutch programmer Bram Moolenaar tried to use vi on the Amiga he had purchased, but there was no vi for the Amiga at the time. So I created version 1.0 of Vim based on the vi clone Stevie.

<!-- spell-checker: words Bram Moolenaar -->

## Commands

### Move cursor

| Mode | Key input | Description |
| :-: | :-: | :-: |
| Normal | k, CTRL+p | up ↑ |
| Normal | j, CTRL+n | down ↓ |
| Normal | h, CTRL+h, BS | left ← |
| normal | l, space | right → |
| Normal | CTRL+b | before page |
| Normal | CTRL+u | before half page |
| Normal | CTRL+d | next half page |
| Normal | CTRL+f | next page |
| Normal | nl | nth character from the beginning of the current sentence |
| Normal | b, B | Start of current word |
| Normal | w, W | Start of next word |
| Normal | e, E | End of current or next word |
| Normal | 0, ( | Start of current sentence |
| Normal | % | End of current sentence |
| Normal | ) | Start of next sentence |
| Normal | \- | Start of previous line |
| Normal | \+ | Start of next line |
| Normal | H | Home |
| Normal | M | Start of center row |
| Normal | L | Start of bottom line |
| Normal | nG | nth row |
| Normal | G | Last line |

### Edit

| Mode | Key input | Description |
| :-: | :-: | :-: |
| Normal | a | Input after the current cursor position |
| Normal | i | Input from current cursor position |
| Normal | o | Input from the beginning of the next line |
| Normal | yy, nyy | Copy |
| Normal | p | Paste after current line |
| Normal | P | Paste before current line |
| Normal | Y, nYp | Copy directly |
| Normal | x | Delete 1 character |
| Normal | dd, ndd | Delete 1 line |
| Normal | r | Single character replacement |
| Normal | R | Replace after cursor position |
| Normal | s, ns | Replace n characters at cursor position with string |
| Normal | S | Replace entire line |
| Normal | cw | Word Replacement |
| Normal | c$ | Replace to end of line |
| Normal | u | UNDO |
| Normal | J, nJ | JOIN |
| Normal | "{Register name} yy | Copy to specified register |
| Normal | "{Register name} p | Paste from specified register |
| Normal | Vu (guu) | Lowercase letters on the current line |
| Normal | VU (gUU) | Capitalize letters on current line |
| Normal | vEu | Make each word lowercase |
| Normal | vEU | Capitalize each word |
| Normal | ; | Repeat |
| Normal | . | Repeat (opposite direction) |
| ex | :reg | Display current register |

### Search/Replace

| Mode | Key input | Description |
| :-: | :-: | :-: |
| Normal | F{Char} | One character on the current line (toward the beginning of the line) |
| Normal | f{Char} | One character on the current line (towards the end of the line) |
| Normal | T{Char} | After one character on the current line (toward the beginning of the line) |
| Normal | t{Char} | After one character on the current line (toward the end of the line) |
| ex | ?{String} | String search toward the beginning of the sentence |
| ex | ? | Repeat the above |
| ex | /{String} | String search towards the end of the sentence |
| ex | / | Repeat the above |
| Normal | n | ? or / repetition |
| Normal | N | ? or / repetition (opposite direction) |

### Batch replacement of strings

```text
:{start position},{end position}s/string1/string2/option
````

| Option | Description |
| :-: | :-: |
| Start position, end position | Start/end "%" Entire file, "." Current line, "$" End of file |
| Options | "g" Replace all when there are multiple targets on one line, "c" Query |

```text
:1,$s/foo/bar/g
/* Replace uppercase letters with lowercase letters */
:%s/[A-Z]/\L&/g
/* Replace lowercase letters with uppercase letters */
:%s/[a-z]/\U&/g
/* Compile the current source */
:!cc % -Wall
/* Change encoding */
:e ++enc=UTF-8
````

### File

| Mode | Key input | Description |
| :-: | :-: | :-: |
| ex | :w[rite], :w! | write |
| ex | :wq, ZZ | write And quit |
| ex | :e[dit] {file name} | read |
| ex | :e! | Re-Read |
| ex | :e# | Read the previously edited file |
| ex | :r {file name} | Insert |

### Others

| Mode | Key input | Description |
| :-: | :-: | :-: |
| ex | [Esc] | Go to ex mode |
| ex | :h[elp] <command, etc.> | Show help |
| ex | :q[uit], :q! | quit |
| ex | :noh | Unhighlight search results |
| ex | :ve[rsion] | Version display. You can also see the configuration file name. |

<!-- spell-checker: words rsion -->

### set command

| Mode | Key input | Description |
| :-: | :-: | :-: |
| ex | :set nu[meric] | Line number display |
| ex | :set nonu[meric] | Hide line number |
| ex | :set tabstop=4 | Setting the number of tabs |
| ex | :set showmode | mode display |
| ex | :set all | Display setting items |

<!-- spell-checker: words meric nonu showmode -->

```text
:set all
--- Options ---
ambiwidth=single keywordprg=man nosplitbelownoautoindent laststatus=1 nosplitrightnoautoread nolazyredraw startofline...
-- More --
````

<!-- spell-checker: words ambiwidth keywordprg nosplitbelownoautoindent laststatus nosplitrightnoautoread nolazyredraw startofline -->

### External command

| Mode | Key input | Description |
| :-: | :-: | :-: |
| ex | :! | Execute external command |
| ex | :!! | Execute the last command |

```text
# Just run the command
:!make
Press ENTER or type command to continue [Enter]

# Specify the file being edited with %
:!node %
Press ENTER or type command to continue [Enter]

# Execute the command and insert the result into the current line below
:r !date

# Execute command and replace with result line
:.!echo foo
````

### Visual Mode

| Mode | Key input | Description |
| :-: | :-: | :-: |
| Normal | v | Shift to character-by-character visual mode |
| Normal | SHIFT+v | Go to line-by-line visual mode |
| Normal | CTRL+v | Go to rectangle selection visual mode |
| Normal | gv | Select the previously selected range again |

## Installation

### on Fedora

unexpected vim-minimal

```bash
sudo dnf install vim-enhanced
````

## Configuration

### Which of "vi" and "vim" is this?

For fedora, it includes a shell that adds an alias if vim is installed.

```bash
/etc/profile.d/vim.sh

if [ -n "${BASH_VERSION-}" -o -n "${KSH_VERSION-}" -o -n "${ZSH_VERSION-}" ]; then
  [ "`/usr/bin/id -u 2>/dev/null || echo 0`" -le 200 ] && return
  # for bash and zsh, only if no alias is already set
  alias vi >/dev/null 2>&1 || alias vi=vim
fi
````

### .vimrc configuration file

The system-wide settings are /etc/vimrc, but normally the following files are created in the home folder.

```vim
$HOME/.vimrc

<!-- spell-checker: words vimrc -->

"----------------------------------------------------
"Display related
"----------------------------------------------------
" Display title in window frame
set title
" Show line numbers
set number
" Showing tabs, line breaks, and unnecessary trailing whitespace
set listchars=tab:>.,trail:_,eol:↲,extends:>,precedes:<,nbsp:%
set list
" Display full-width spaces
" highlight ZenkakuSpace cterm=underline ctermfg=lightblue guibg=darkgray
" match ZenkakuSpace /　/
" Arrow keys
set nocompatible

<!-- spell-checker: words listchars nocompatible Zenkaku cterm ctermfg guibg -->

"----------------------------------------------------
" indent
"----------------------------------------------------
" Replace tab input with multiple blank inputs
set expandtab
" The number of spaces the tab corresponds to
set tabstop=2
" The number of spaces a tab corresponds to when performing editing operations such as using tabs and backspaces.
set softtabstop=2
" Number of spaces used for each level of indentation
set shiftwidth=2
" Continue indentation of previous line on line break
set autoindent
" Increase or decrease the indentation of the next line to match the end of the line entered at line break
set smartindent
````

<!-- spell-checker: words expandtab tabstop softtabstop shiftwidth autoindent smartindent -->

## Troubleshooting

### [SOLVED] Freeze ?

New Linux users often accidentally press CTRL + S (^S) (or the ``Stop Scroll'' key), causing mysterious screen freezes.

The solution seems to be CTRL + Q. Magic car.

## References

- [http://vim-jp.org/vimdoc-ja/](http://vim-jp.org/vimdoc-ja/)
