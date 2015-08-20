---
layout: post
title:  "Bash Tips & Tricks"
---

Here are some of my Bash snippets which I use on daily basis. You should find them useful even if you use another shell because it should recognise [shebang][shebang] at the beginning of a script. For bash it's `#!/bin/bash`. Personally I use [Zsh][zsh], but I write scripts in Bash for two reasons. Fistly I often want them on work on Linux server machines where Bash is a standrad shell and I can't or don't want to install another one. Secondly since Bash is a standrad shell in Linux and OSX I can switch to another shell when I find it fits me better than my current one without rewriting all my scripts to avoid a mess.

# Multiline string

{% highlight bash %}
read -d '' sql << EOF
select c1, c2 from foo
where c1='something'
EOF

echo "$sql"
{% endhighlight %}

Multiline strings may be useful when you want for example call some RESTful API and you need to pass a JSON string with `curl` and you don't want to read its content from an external file.

# Redirect standard error and output to a file

{% highlight bash %}
>FILE 2>&1
{% endhighlight %}

I'm pretty sure you know that one, but I decided to put it here for a reason. Many times I wrote something like this `program 2>&1 >out.txt` and happily put it in some script on remote machine ending with losing all error logs of the program. There is a little nuance. Redirections are processed from left to right which means that stderr is redirected to stdout (a system console), then it redirects stdout to a file without touching stderr (i.e. it's still writing to the old stdout target). Such a situation is nicely explained [here](http://unix.stackexchange.com/questions/177525/redirecting-standard-output-and-standard-error-to-one-file). I tend to remember that the order matters, but I'm not sure what is the right form. Having it here allows me to copy-paste it without waisting time on experimentig or checking it on the web.

# Sanitise string

`echo "$NAME" | tr -C '[:alnum:]' '_' | tr '[:upper:]' '[:lower:]'`

Example: `echo "ABC-D E" | tr -C '[:alnum:]' '_' | tr '[:upper:]' '[:lower:]'` will output *abc_d_e*.

Sometimes you need to create a file depending on input arguments' values. You can expect everything there: tabs, spaces, dots or even some special characters. It's generally a good idea to avoid these in filenames - this is where this code snippet comes to the rescue. You can utilise it in a different way, though. First pipe `tr -C '[:alnum:]' '_'` replaces all non-alphanumeric characters with underscore. This is because of *-C* option passed to *tr* (see *tr* man page for details). If you find it more suitable, you can replace underscore with any other character. Second pipe `tr '[:upper:]' '[:lower:]'` will convert uppercase characters with lowercase ones. You can swap *lower* and *upper*, or drop that pipe if you want to keep original case. 

# Option parse

{% highlight bash %}
aflag=no
bflag=no

while [ $# -gt 0 ]
do
    case "$1" in
    -a) aflag=yes;;
    -b) bflag=yes;;
    -f) flist="$flist $2"; shift;;
    --single) single="$2"; shift;;
    --) shift; break;;
    -*) echo "unrecognised option $1" 1>&2; exit 1;;
    *)  break;;
    esac
    shift
done
{% endhighlight %}

It's handy when you want to create some script taking arguments **without dependences**. Of course it doesn't follow all [POSIX conventions][posix-argument-syntax] - if really you need it I think it would be reasonable to use an external library.

Let's go trough the code to understand what's going on. The loop breaks when:

* breaking condition `$# -gt 0` is fulfilled (i.e. there are no more arguments to [shift][bash-shift])
* *--* is consumed - this follows the [convention][posix-argument-syntax]
* argument not starting with dash is consumed
* unrecognised option is passed - in that case program exits immediately with *1* status code

There is a [*shift*][bash-shift] call at the end of each iteration. That's why case is matched against *$1* - first argument.

The loop consumes three different types of flags: boolean, single value, and lists. When boolean flag *-a* or *-b* is consumed then corresponding variable *aflag* or *bflag* respectively are set to *yes*. Upon arrival of single value flag named *--single* variable *single* is set to a value of current second argument and additional [*shift*][bash-shift] is called to not perform next match against the flag value (i.e. current value of *single* variable).

For example after loop finishes parsing argument list *-f f1 -a -f f2 --single s1 --single s2 -f f3* script variables will have assigned following values:

* *aflag='yes'*
* *bflag='no'*
* *single='s2'*
* *flist='f1 f2 f3'*

# Script directory

```
DIR=$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )
```

The last but not least, is obtaining value of absolute directory path of the executing script. It's invaluable when you need to source or operate on external files. [Here](http://stackoverflow.com/questions/59895/can-a-bash-script-tell-what-directory-its-stored-in) you can find more details.

[shebang]: https://en.wikipedia.org/wiki/Shebang_(Unix)
[zsh]:     http://www.zsh.org/
[posix-argument-syntax]: http://www.gnu.org/software/libc/manual/html_node/Argument-Syntax.html
[bash-shift]: http://tldp.org/LDP/Bash-Beginners-Guide/html/sect_09_07.html
