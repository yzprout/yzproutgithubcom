---
layout: post
title: color bash configuartion
---

工欲善其事，必先利其器。

作为开发使用最多的就是bash了。

{% highlight %}
export PS1="\[\e[0;33m\]\u\[\e[0m\]@\[\e[34m\]\h \[\e[32m\]\w\[\e[35m\] \n\[\e[31m\]\$>>\[\e[0m\] "
{% endhighlight%}

写的挺长，看起来毫无头绪，分解开来就简单了。
{% highlight %}

"\e[0;33m\]" 这一格式的都是颜色标识

"\u" 用户名

"\h" 主机名

"\w" 绝对路径

"\n" 换行符

"\$" 身份说明，$为普通拥护，#为root用户

{% endhighlight%}

除去颜色就只剩下:

{% highlight %}
export PS1="\u@\h \w \n \$>> "
效果图:
yuen@enmatoMacBook-Air /usr/local 
$>> 
{% endhighlight% %}


这里有一个较为详细的配置说明:
[Color Bash Prompt](https://wiki.archlinux.org/index.php/Color_Bash_Prompt)

中秋快乐!  ^_^

EOF

