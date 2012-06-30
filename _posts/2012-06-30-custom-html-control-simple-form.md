---
title: Custom HTML form code for Simple Form
layout: post
---

Recently during my day job I ran into a situation where I wanted to place HTML 5 data attributes on the option tag of a select box. The problem came because we are using simple_form to generate the form and simple_form does not allow you to pass in any attributes for the HTML tags on the option elements. So this post is how I solved this problem. Initially, I tried to find a way to solve this problem using the simple_form method wrappers, however, I could not find a satisfactory solution. In fact, nothing worked and there was no solution that I could find. So I had to look in other avenues and this is what I found.


The solution came in the ability to pass the simple_form `input` method a block. When the `input` method is called with a block it assumes the block will output the HTML for the control element.  So if we simply pass in a block that does nothing:

{% highlight ruby %}
form.input(:type) {}
{% endhighlight %} 

then simple_form will simply not generate a control element:


{% highlight html linenos %}
<div class="input string optional">
  <label class="string optional" for="note_type">Type</label>
</div>
{% endhighlight %}

As you can see it generates all of the wrapping elements and the label element, but not the control element.  By simply having the block return a string the return value will become the HTML control element.  So 

{% highlight ruby %}
form.input(:type) {"Control Element"}
{% endhighlight %}

Generates this HTML

{% highlight html linenos %}
<div class="input string optional">
  <label class="string optional" for="note_type">Type</label>
  Control Element
</div>
{% endhighlight %}


So back to my original problem of getting custom data elements on the option tag I used this ruby code:


{% highlight ruby linenos %}
= form.input :type do
  = form.select :type, options_for_select([['Blog', 'blog',  {'data-key' => 'blog'}], ['Article', 'article', {'data-key' => 'blog'}]])
{% endhighlight %}


With the HTML code that I want:


{% highlight html linenos %}
<div class="input string optional">
  <label class="string optional" for="note_type">Type</label>
  <select id="note_type" name="note[type]">
    <option value="blog" data-key="blog">Blog</option>
    <option value="article" data-key="blog">Article</option>
  </select>
</div>
{% endhighlight %}

So while his is a simple solution to a simple problem.  Since I could not find a good source of information about it on the web I hope you found this helpful.
