---
laytou: post
title: mac os x development environment configuration
---
新机器到手，2011版macbook air。这些天都在不停地捣腾着它，从linux转到这下面还真是有些不习惯。

一个顺手地工作环境还是非常重要的，对于开发者来说用到最多的还是bash了。
{% highlight java %}
in .bash_profile
#彩色显示不同文件类型
alias ls="ls -G"
#给bash换个提示符
export PS1='`a=$?;if [ $a -ne 0 ]; then a="  "$a; echo -ne "\[\e[s\e[1A\e[$((COLUMNS-2))G\e[31m\e[1;41m${a:(-3)}\e[u\]\[\e[0m\e[7m\e[2m\]"; fi`\[\e[1;32m\]\u@\h:\[\e[0m\e[1;34m\]\W\[\e[1;34m\]\$ \[\e[0m\]'
{% endhighlight %}
