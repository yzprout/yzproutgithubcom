---
layout: post
title: ELResolver Escapes JSP EL Values To Prevent Cross-Site Scripting
---

[Cross-site scripting](http://en.wikipedia.org/wiki/Cross-site_scripting) is a
computer security vulnerability enabling an attacker to inject malicious code
into a Web page that will be executed by the Web browser when other users view
the page.  If your Web application accepts data from the user and then outputs
that data unaltered in HTML, then it is vulnerable because user-controlled
data might contain executable code.

Since JSP 2.0, EL expressions can appear in the template text:

{% highlight jsp %}
<h1>Hello, ${user.name}</h1>
{% endhighlight %}

Unfortunately, the JSP container does not escape expression values, so if the
expression contains user-controlled data, then cross-site scripting is
possible.  JSTL provides a couple of ways to sanitize the output.  The `c:out`
tag escapes XML characters by default:

{% highlight jsp %}
<c:out value="${user.name}"/>
{% endhighlight %}

Alternatively, the EL function `fn:escapeXml` also escapes XML characters:

{% highlight jsp %}
${fn:escapeXml(user.name)}
{% endhighlight %}

The default option should be the safe option.  That's a sensible engineering
principle.  If EL values are escaped by default, then you're protected from
coders who forget to wrap expressions in `c:out` or `fn:escapeXml`.

Starting with JSP 2.1, a Web application can register a custom
[ELResolver](http://download.oracle.com/javaee/6/api/javax/el/ELResolver.html).
I'm going to present a
[custom ELResolver that escapes EL values](http://github.com/pukkaone/webappenhance/blob/master/src/com/github/pukkaone/jsp/EscapeXmlELResolver.java),
allowing you to use EL in JSPs while preventing cross-site scripting.

A
[custom servlet context listener](http://github.com/pukkaone/webappenhance/blob/master/src/com/github/pukkaone/jsp/EscapeXmlELResolverListener.java)
registers the custom ELResolver when the application starts.  To use
it, define a listener in the `web.xml` file:

{% highlight xml %}
<listener>
  <listener-class>com.github.pukkaone.jsp.EscapeXmlELResolverListener</listener-class>
</listener> 
{% endhighlight %}


### Disable escaping

When you register this custom ELResolver, all EL values will be escaped by
default.  If you want a JSP to programmatically output HTML, you can resort to
a
[JSP scriptlet](http://download.oracle.com/javaee/5/tutorial/doc/bnaou.html)
or
[JSP expression](http://download.oracle.com/javaee/5/tutorial/doc/bnaov.html),
unless the application configured `scripting-invalid` to `true`.

Another way uses a custom tag to surround JSP code in which EL values should
not be escaped:

{% highlight jsp %}
<%@ taglib prefix="enhance" uri="http://pukkaone.github.com/jsp" %>

<enhance:out escapeXml="false">
  I hope this expression returns safe HTML: ${user.name}
</enhance:out>
{% endhighlight %}

The `escapeXml` attribute is `true` by default.  You must explicitly set it to
`false` in the tag to disable escaping.


### Details

The servlet context listener's `contextInitialized` method calls the
[JspApplicationContext.addELResolver](http://download.oracle.com/javaee/6/api/javax/servlet/jsp/JspApplicationContext.html#addELResolver(javax.el.ELResolver\))
method to register the custom ELResolver.

{% highlight java %}
    public void contextInitialized(ServletContextEvent event) {
        JspFactory.getDefaultFactory()
                .getJspApplicationContext(event.getServletContext())
                .addELResolver(new EscapeXmlELResolver());
    }
{% endhighlight %}

The `addELResolver` method inserts the custom ELResolver into a chain of
standard resolvers.  When evaluating an expression, the JSP container consults
the chain of resolvers in the following order, stopping at the first resolver
to succeed:

 * ImplicitObjectELResolver
 * *ELResolvers registered by the addELResolver method.*
 * MapELResolver
 * ListELResolver
 * ArrayELResolver
 * BeanELResolver
 * ScopedAttributeELResolver

This presents a slight problem because the custom ELResolver wants to escape
the value that would have resulted from consulting the chain.  When asked for a
value, the custom ELResolver invokes the chain of resolvers.  The custom
ELResolver is itself in the chain of resolvers, so before invoking the chain,
it sets a flag telling itself to do nothing when its turn in the chain comes
around.

{% highlight java %}
    private boolean gettingValue;

    @Override
    public Object getValue(ELContext context, Object base, Object property)
        throws NullPointerException, PropertyNotFoundException, ELException
    {
        if (gettingValue) {
            return null;
        }
        
        gettingValue = true;
        Object value = context.getELResolver().getValue(
                context, base, property);
        gettingValue = false;

        if (value instanceof String) {
            value = EscapeXml.escape((String) value);
        }
        return value;
    }
{% endhighlight %}

There's a resolver in the chain before the custom ELResolver.  This resolver,
ImplicitObjectELResolver, will be invoked twice.  First, before reaching the
custom ELResolver, and again when the custom ELResolver invokes the chain.
Multiple invocations of ImplicitObjectELResolver is harmless because
ImplicitObjectELResolver had to fail in order for the custom ELResolver to be
invoked.  When the custom ELResolver invokes the chain, the
ImplicitObjectELResolver will fail again.

A resolver indicates success by setting the `propertyResolved` property of the
`ELContext` to `true`.  When consulting the chain, one of the resolvers very
likely set this property to `true`, so no other resolvers are invoked after
returning from the custom ELResolver.
