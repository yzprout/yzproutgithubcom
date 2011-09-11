---
layout: post
title: compile nginx on mac os x lion
---

在lion下编译nginx会遇到openssl的一个错误，原因其中一些api被建议废弃了，编译时不停的报警告之后错误。
你可能得到一串下面类似的错误
{% highlight bash%}
cc1: warnings being treated as errors
src/core/ngx_crypt.c: In function ‘ngx_crypt_apr1’:
src/core/ngx_crypt.c:76: warning: ‘MD5_Init’ is deprecated (declared
at /usr/include/openssl/md5.h:113)
{% endhighlight %}

只需要在configure时加上一条:
{% highlight bash%}
./configure --with-cc-opt="-Wno-deprecated-declarations"
{% endhighlight %}
 
hava fun  : )