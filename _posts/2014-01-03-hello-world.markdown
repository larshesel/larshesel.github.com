---
layout: post
title:  "Hello World"
date:   2014-01-03 08:56:55
categories: jekyll update
---

I finally pulled myself together and got a basic github-pages blog up
and running. Below are a few examples of code highlighting:


First some Erlang:

{% highlight erlang %}
sum(A, B) ->
	A + B.

ismonday(monday) ->
   true;
ismonday(_) ->
   false.
{% endhighlight %}

Then some c:

{% highlight c %}
int add(int a, int b) {
	return a + b;
}
{% endhighlight %}

Last some ruby:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Seems to be working ;)
