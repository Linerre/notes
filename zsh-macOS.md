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
suggest that I put `$PATH`, together with a few other environment variables,
somewhere in my `~/.zshenv`. I followed the advice and my `~/.zshenv` looks
like this:

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
leaving the system-wide ones alone. Kind of clean in my eyes. And it'd worked
pretty well on all my Linux/\*BSD machines. 

The `$KERNEL_NAME` and `case` statement is for zsh to detect which OSs I'm on.
If I'm on macOS, prepare for MacPorts and macOS Nodejs. 

## Zsh way of $PATH


## the special macOS

