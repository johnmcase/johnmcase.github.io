---
layout: page
title: Welcome to my blog
tagline: 

---
{% include JB/setup %}

{%comment%}
Read [Jekyll Quick Start](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)

Complete usage and documentation available at: [Jekyll Bootstrap](http://jekyllbootstrap.com)
{%endcomment%}

I'm starting to blog!  This will mostly be a programming blog to start, but who knows how it will evolve over time.

<!--excerpt-->

### Recent Posts
<ul>
{% for post in site.posts %}
<li>
{{ post.date | date_to_string }} - <a href="{{ post.url }}">{{ post.title }}</a>
<p>{{ post.excerpt | markdownify }}</p>
</li>
{% endfor %}
</ul>

{%comment%}
### Code Highlighting

{% highlight scala linenos %}
object Replicator {
  case class Replicate(key: String, valueOption: Option[String], id: Long)
  case class Replicated(key: String, id: Long)

  case class Snapshot(key: String, valueOption: Option[String], seq: Long)
  case class SnapshotAck(key: String, seq: Long)

  def props(replica: ActorRef): Props = Props(new Replicator(replica))
}
{% endhighlight %}

### Gist Link

{% gist johnmcase/e8d487a585e5cba545c5 %}

## Update Author Attributes

In `_config.yml` remember to specify your own data:
    
    title : My Blog =)
    
    author :
      name : Name Lastname
      email : blah@email.test
      github : username
      twitter : username

The theme should reference these variables whenever needed.
    
## Sample Posts

This blog contains sample posts which help stage pages and blog data.
When you don't need the samples anymore just delete the `_posts/core-samples` folder.

    $ rm -rf _posts/core-samples

Here's a sample "posts list".

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## To-Do

This theme is still unfinished. If you'd like to be added as a contributor, [please fork](http://github.com/plusjade/jekyll-bootstrap)!
We need to clean up the themes, make theme usage guides with theme-specific markup examples.

{%endcomment%}


