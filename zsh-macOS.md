# macOS, Zsh, and more This gist is based on a twitter
[thread](https://twitter.com/giftboiling/status/1386359878797565955?s=20) I
just want to add a few more details on this topic.

## Zsh initialization files
I'm not going to repeat what has alreay  been well documented:
When zsh starts up, it looks for a few [startup
files](http://zsh.sourceforge.net/Guide/zshguide02.html#l). Among them,
`~/.zshenv` and `~/.zshrc` are the most relevant to a normal user like me. 

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
As you can see, I prefer to use my own Python and Nodejs under my $HOME while
leaving the system ones alone. Kind of clean in my eyes. And it'd worked
pretty well on all my Linux/\*BSD machines. 

The `$KERNEL_NAME` and `case` statement is for zsh to detect which OS I'm on.
If I'm on macOS, prepare for MacPorts and macOS Nodejs. 

## Zsh way of $PATH
The only difficulty that had baffled me was zsh's way of $PATH setting. It is
NOT that you can't do `export PATH=...` stuff. Zsh provides an array which is
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

