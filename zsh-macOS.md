# macOS, Zsh, and more 
This gist is based on a twitter
[thread](https://twitter.com/giftboiling/status/1386359878797565955?s=20) I
just want to add a few more details on this topic.

## Zsh initialization files
I'm not going to repeat what has alreay  been well documented:
When zsh starts up, it looks for a few [startup
files](http://zsh.sourceforge.net/Guide/zshguide02.html#l). Among them,
`~/.zshenv` and `~/.zshrc` are the most relevant to a normal user like me. They are also the focus of this gist.

Both the [Zsh User Guide](http://zsh.sourceforge.net/Guide/zshguide02.html#l23)
and [Arch Wiki](https://wiki.archlinux.org/index.php/Zsh#Configuring_$PATH)
suggest that I put environment variables such as `$PATH` in my `~/.zshenv`. I
followed the advice and my `~/.zshenv` looks like this:

```shell    
# ########################
# Environment variables  #
# ########################
#
export EDITOR=nvim
export PAGER=less
export ZDOTDIR=$HOME/.config/zsh
export XDG_CONFIG_HOME=$HOME/.config
export KERNEL_NAME=$( uname | tr '[:upper:]' '[:lower:]' )

# remove duplicat entries from $PATH
# zsh uses $path array along with $PATH 
typeset -U PATH path

# user compiled python as default python
export PATH=$HOME/python/bin:$PATH
export PYTHONPATH=$HOME/python/

# user installed node as default node
export PATH="$HOME/node/node-v16.0.0-${KERNEL_NAME}-x64"/bin:$PATH
export NODE_MIRROR=https://mirrors.ustc.edu.cn/node/
	
case $KERNEL_NAME in
    'linux')
        source "$HOME/.cargo/env"
        ;;
    'darwin')
        PATH:/opt/local/bin:/opt/local/sbin:$PATH
        ;;
    *) ;;
esac
```
As you can see, I prefer to use my own Python and Nodejs under my `$HOME` while
leaving the system ones alone. Kind of clean in my eyes. And it'd worked
pretty well on all my Linux/\*BSD machines. 

The `$KERNEL_NAME` and `case` statement is for zsh to detect which OS I'm on.
If I'm on macOS, prepare for MacPorts and macOS Nodejs. 

## Zsh way of $PATH
The only difficulty that had baffled me was zsh's way of $PATH setting. It is
NOT that you can't do `export PATH=...` stuff, but that zsh provides an array which is
tied to `$PATH`, meaning that you can either change `$PATH` using the `export`
convention (changing a scalar string) or you can change `$path` (lowercase)
which is an array and makes it easier to append, prepend, and even insert new
paths to the exisiting variable.

Over time, `$PATH` can become quite messy, including a lot of dupplicates.
`$path` comes in handy and there are already many mentioning/talking about
this. Just to list a new:
1. [Arch Wiki](https://wiki.archlinux.org/index.php/Zsh#Configuring_$PATH):
>The line `typeset -U PATH path`, where the `-U` stands for unique, instructs
>the shell to discard duplicates from both `$PATH` and `$path`
2. [Zsh User Guide](http://zsh.sourceforge.net/Guide/zshguide02.html#l24):
>The incantation `typeset -U path`, where the `-U` stands for unique, tells the
>shell that it should not add anything to `$path` if it's there already.
3. [StackOverFlow](https://stackoverflow.com/a/18077919):
>ZSH allows you to use special mapping of environment variables ... For me
>that's a very neat feature which can be propagated to other variables.
4. [StaclExchange](https://unix.stackexchange.com/a/532155):
>That's useful in that it makes it easier to add remove components or loop over them.
5. [StackExchange](https://superuser.com/a/1447959):
>...benefit of using an array ...

## the special macOS
The above `$path` and `$PATH` kept working well until I tried to migrate them to my M1 running macOS. If I run `echo $PATH` or `echo $path`, I got this:

```
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Applications/VMware Fusion.app/Contents/Public:/Library/TeX/texbin:/Library/Frameworks/Mono.framework/Versions/Current/Commands:/opt/local/bin:/opt/local/sbin:/Users/leon/node/node-v16.0.0-darwin-x64/bin:/Users/leon/python/bin
```
It is wired that even I **prepend** some paths to `$PATH`, they end up appearing at the last! The paths for system utilities come first! How come? I searched a lot and finally nailed down the root cause: `/usr/libexec/path_helper`.

## `/usr/libexec/path_helper`
### What is it?
It is a binary file. No way to see the source code. As is the case with many unix utilities, it has a man page:
```
path_helper(8)            BSD System Manager's Manual           path_helper(8)

NAME
     path_helper -- helper for constructing PATH environment variable

SYNOPSIS
     path_helper [-c | -s]

DESCRIPTION
     The path_helper utility reads the contents of the files in the direc-
     tories /etc/paths.d and /etc/manpaths.d and appends their contents to
     the PATH and MANPATH environment variables respectively.  (The MANPATH
     environment variable will not be modified unless it is already set in
     the environment.)

     Files in these directories should contain one path element per line.

     Prior to reading these directories, default PATH and MANPATH values
     are obtained from the files /etc/paths and /etc/manpaths respectively.

     Options:

     -c      Generate C-shell commands on stdout.  This is the default if
             SHELL ends with "csh".

     -s      Generate Bourne shell commands on stdout.  This is the default
             if SHELL does not end with "csh".

NOTE
     The path_helper utility should not be invoked directly.  It is
     intended only for use by the shell profile.

Mac OS X                        March 15, 2007                        Mac OS X
(END)
```
This has been mentioned in a few places on the Internet too:
1. [Setting system path for command line on Mac OSX](https://www.chrissearle.org/2014/01/28/setting-system-path-for-command-line-on-mac-osx/)
2. [Where is the default terminal $PATH located on Mac?](https://izziswift.com/where-is-the-default-terminal-path-located-on-mac/)
3. [Where is the default terminal $PATH located on Mac?](https://stackoverflow.com/questions/9832770/where-is-the-default-terminal-path-located-on-mac)

### What does it do?
From the man page and info above, it is pretty clear what this tool does:
1. it looks for the file `/etc/paths` and files under `/etc/paths.d/`
2. it combined all the paths in these files
3. it returns the combined paths (as a string) and assigns it to `$PATH`

It is easy to confirm this. Simply run: `$ /usr/libexec/path_helper`

And it return something like:

```
PATH="/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Applications/VMware Fusion.app/Contents/Public:/Library/TeX/texbin:/Library/Frameworks/Mono.framework/Versions/Current/Commands:/opt/local/bin:/opt/local/sbin:/Users/leon/node/node-v16.0.0-darwin-x64/bin:/Users/leon/python/bin";
export PATH 
``` 
On a fresh macOS, it can be as simple as
`PATH="/usr/local/bin:/usr/bin:/bin:/usr/sbin"; export $PATH`.  This article,
[Where PATHs come
from](https://scriptingosx.com/2017/05/where-paths-come-from/), explains pretty
much everything about `$PATH` on macOS.

### What does it break?
The only problem, in my opinion, is that `path_helper` **re-order** the
`$PATH`! *Where PATHs come from* mentioned this:
>We have seen `path_helper` is extremely useful. There is one caveat, however.
>`path_helper` may reorder your `PATH`. Imagine you are pre-pending `~/bin` to
>your `PATH` because you want to override some standard tools with your own.
>(Dangerous, but let’s assume you know what you are doing.) Then some process
>launches a subshell which can call `path_helper` again. `path_helper` will
>‘find’ your additions to the defined `PATH`, but it will append to the list of
>default paths from `/etc/paths` and `/etc/paths.d`, changing your order and
>thus which tools will be used.

Note it will **append** any user-level path to the default one, which breaks
things. [This
guy](https://github.com/sorin-ionescu/prezto/issues/381#issuecomment-12800590)
was so angry that he thought `path_helper` is **"broken"**:
>@mislav, I understand how it works. It's broken.
>
>I also understand that having it enabled causes problems. You forget the path
>set in ~/.MacOSX/environment.plist, which is needed for GUI apps, and
>Terminal.app, being a GUI app, will get that environment before it launches a
>shell subprocesses.
>
>`path_helper` reorders the PATH entries set in environment.plist, which causes
>GUI programs to fail when they execute their scripts via a shell subprocess. A
>good example is any editor that depends on Exuberant Ctags. `path_helper`
>reorders the PATH entries and /usr/bin/ctags gets executed instead of Exuberant
>Ctags that is installed elsewhere. The editor complains that the wrong ctags is
>installed.
>
>If I don't set the PATH in environment.plist for GUI apps, then they complain
>that they can't find programs.
>
>Setting PATH on Mac OS X is a mess, and `path_helper` doesn't help.


