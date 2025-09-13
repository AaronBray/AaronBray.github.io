---
layout: post
title: Table of content
date:   2015-12-12 
description: For some big articles you can use table on content
toc: true
---

Above you can see how it looks like. 

To enable it add a `toc` variable to the front matter of your post:

{% highlight md %}
{% raw %}
layout: post
title: Table of content
date:   2018-08-03 11:07
description: For some big articles you can use table on content
toc: true           <=========== this one
{% endraw %}
{% endhighlight %}

You can also customize it by styling `.toc` class in **theme.scss** 

This solution is based on [github.com/allejo/jekyll-toc](https://github.com/allejo/jekyll-toc).

# Analytics
 
#### [Google Analytics](http://www.google.com/analytics/)

To enable Google Analytics create an account [here](https://analytics.google.com). Then add your tracking id in `config.xml`, it should look something like `UA-********-1`
 
#### [Yandex Metrica](http://metrica.yandex.com)
 
To enable Yandex Metrica you need to register, create a 'counter' and then copy-paste it's code in `/_includes/yandex-metrica.html` file.

# Tags

To use this feature you simply will need to create a markdown file for each tag which you are using in you site in **tag** folder. To simplify this procedure there is an [/admin]({{site.baseurl}}/admin) page, which outputs the bash command which you just need to run inside **tag** folder of your site. Also don't forget to rerun it when you add a post with new tag.

# Comments

To enable [Disqus](http://disqus.com) register on the site and then just put your name in `_config.xml`. Comments could be switched on and off on per post basis, just put `comments: true` to enable them.
## Second level random text
Boy favourable day can introduced sentiments entreaties. Noisier carried of in warrant because. So mr plate seems cause chief widen first. Two differed husbands met screened his. Bed was form wife out ask draw. Wholly coming at we no enable. Offending sir delivered questions now new met. Acceptance she interested new boisterous day discretion celebrated. 

# Social icons

You can have social icons which could lead to your social profile.
Out-of-the box it has: 

<ul class="social-media">
  <li>
    <a title="Github"
      href="https://github.com/{{ site.social.github }}"
      target="_blank"><i class="fab fa-github fa-2x"></i></a>
  </li>
  <li>
    <a title="StackOverflow"
      href="http://stackoverflow.com/users/1252056/{{ site.social.stackoverflow }}"
      target="_blank"><i class="fab fa-stack-overflow fa-2x"></i></a>
  </li>
  <li>
    <a title="LinkedIn"
      href="https://www.linkedin.com/in/{{ site.social.linkedin }}"
      target="_blank"><i class="fab fa-linkedin fa-2x"></i></a>
  </li>
  <li>
    <a title="Instagram"
      href="https://instagram.com/{{ site.social.instagram }}"
      target="_blank"><i class="fab fa-instagram fa-2x"></i></a>
  </li>
  <li>
    <a title="Last.fm"
      href="http://lastfm.com/user/{{ site.social.lastfm }}"
      target="_blank"><i class="fab fa-lastfm fa-2x"></i></a>
  </li>
  <li>
    <a title="RSS"
      href="{{site.url}}/{{ site.social.rss }}"
      target="_blank"><i class="fa fa-rss fa-2x"></i></a>
  </li>
</ul>

They could be setup in `_config.yml`.

To add more icons do following steps:

 - choose an icon you want to use: [Font Awesome Icons](https://fortawesome.github.io/Font-Awesome/icons/)
 - add variable in `_config.yml`
 - add icon in `social.html` with check if variable exists:
 
{% highlight html %}
{% raw %}
{% if site.social.rss %}
  <li>
    <a title="{{ site.social.<your_social_variable> }}" 
       href="{{site.url}}/{{ site.social.<your_social_variable> }}" 
       target="_blank"><font_awesome_icon></i></a>
  </li>
{% endif %}
{% endraw %}
{% endhighlight html %}

_____________________________________________________

# Share buttons

This theme comes with built-in share buttons. You can see them in the bottom of this post.
To turn them on in the header of your post add:

{% highlight yml %}
layout: post
title: "Be sociable"
date: 2016-05-15 16:25:06
description: Built-in share buttons!
share: true <-- here
{% endhighlight yml %}

If you want to disable some of them - go to **_config.yml**:

>_config.yml
{:.filename}
{% highlight yml%}
share:
  facebook: true
  twitter: true
  gplus: true
  linkedin: true
  pinterest: true
  email: true
{% endhighlight yml%}

To add new buttons:

1. add icon name in **_config.yml**;
2. add section in **_includes/share.html**;
3. add styles in **css/theme.css**.



_______________________________________

## Introduction

For code syntax coloration I'm using Darcula theme from Intellij IDEA, which I've found in this post [Darcula theme for Pygments](http://smasue.github.io/pygments-darcula).

XML with line numbers (linenos flag), `{{ "{%" }} highlight xml linenos %}`:
{% highlight xml linenos %}
{% raw %}
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>{{ site.name }}</title>
    <description>{{ site.description }}</description>
    <link>{{site.baseurl | prepend:site.url}}</link>
    <atom:link href="{{site.baseurl | prepend:site.url}}/feed.xml" rel="self" type="application/rss+xml" />
    {% for post in site.posts limit:10 %}
      <item>
        <title>{{ post.title }}</title>
        <description>{{ post.content | xml_escape }}</description>
        <pubDate>{{ post.date | date: "%a, %d %b %Y %H:%M:%S %z" }}</pubDate>
        <link>{{post.url | prepend:site.baseurl | prepend:site.url}}</link>
        <guid isPermaLink="true">{{post.url | prepend:site.baseurl | prepend:site.url}}</guid>
      </item>
    {% endfor %}
  </channel>
</rss>
{% endraw %}
{% endhighlight xml %}

>JSON
{:.filename}
{% highlight json %}
{"employees":[
    {"firstName":"John", "lastName":"Doe"},
    {"firstName":"Anna", "lastName":"Smith"},
    {"firstName":"Peter", "lastName":"Jones"}
]}
{% endhighlight %}

>SQL
{:.filename}
{% highlight SQL %}
select count(*) as cm_content_nodes
from alf_node nd, alf_qname qn, alf_namespace ns
where qn.ns_id = ns.id
  and nd.type_qname_id = qn.id
  and ns.uri = 'http://www.alfresco.org/model/content/1.0'
  and qn.local_name = 'content';
{% endhighlight %}

>Java
{:.filename}
{% highlight java %}
private String getToken(HttpClient client) throws UnsupportedEncodingException{
  Cookie[] cookies = client.getState().getCookies();
  for (Cookie cookie : cookies){
    if (cookie.getName().equals("Alfresco-CSRFToken")){
      return URLDecoder.decode(cookie.getValue(), "UTF-8");
    }
  }
  return null;
}
{% endhighlight %}

To add name to the code snippet, as in the examples above, add following construction before the snippet:

{% highlight bash %}
{% raw %}
>Java
{:.filename}
{% highlight java %}
...
{% endraw %}
{% endhighlight %}
____________________________________________

Here is a sample post for Jekyll-Clean-Dark theme. 

* get it from [github](https://github.com/streetturtle/jekyll-clean-dark)
* see the [live demo](http://pavelmakhov.com/jekyll-clean-dark)
* see it [in action on my blog](http://pavelmakhov.com)

This theme was created on top of [Jekyll Clean theme](https://scotte.github.io) by Scotte.

This theme uses some parts of Twitter Bootstrap, which allows it looks nice on a mobile devices using a collapsable nav bar and hiding the sidebar.

Here how it looks like on some portable devices:

![My helpful screenshot]({{ '/assets/images/iphone_portrait.PNG' | relative_url }}){: .center-image }*iPhone 5 portrait*

![My helpful screenshot]({{ '/assets/images/iphone_landscape.PNG' | relative_url }}){: .center-image }*iPhone 5 landscape*

![My helpful screenshot]({{ '/assets/images/ipad_portrait.PNG' | relative_url }}){: .center-image }*iPad mini portrait*

![My helpful screenshot]({{ '/assets/images/ipad_landscape.PNG' | relative_url }}){: .center-image }*iPad mini landscape*


_______________________________

# Text formatting example

Some examples of text formatting for some common text elements.

# Headers

# Header1

## Header2

### Header3

#### Header4

# Emphasis

Italics: `*asterisks*` -> *asterisks* or `_underscores_` -> _underscores_.

Bold: `**asterisks**` -> **asterisks** or `__underscores__` -> __underscores__.

You also can combine them: `**asterisks and _underscores_**` -> **asterisks and _underscores_**.

# Blockquotes and notes

{% highlight bash %}
>Blockquotes
{% endhighlight bash %}

>Blockquotes

Using very cool [feature](http://kramdown.gettalong.org/quickref.html#block-attributes) of kramdown which allows to assign any attribute to a block-level element I've added note and warning:

{% highlight bash %}
>Note 
{: .note}
{% endhighlight bash %}

>Note 
{: .note}

{% highlight bash %}
>Warning 
{: .note .warning}
{% endhighlight bash %}

>Warning 
{: .note .warning}

# Keyboard buttons

In case you need to show some keyboard shortcuts, like `Ctrl`{: .key}+`A`{:.key} use following construction:

{% highlight bash %}
`Ctrl`{: .key}+`A`{:.key}
{% endhighlight bash %}

Example of keyboard shortcuts in a terminal:

`Ctrl`{: .key} + `A`{: .key} = move cursor to beginning of line
`Ctrl`{: .key} + `E`{: .key} = move cursor to end of line
`Ctrl`{: .key} + `C`{: .key} = kills the current process.
`Ctrl`{: .key} + `Z`{: .key} = sends the current process to the background.
`Ctrl`{: .key} + `D`{: .key} = logs you out.
`Ctrl`{: .key} + `R`{: .key} = finds the last command matching the entered letters.

________________________________________





## Introduction

This theme supports two types of images:
 
- inline images: ![Battery Widget]({{ '/assets/images/batWid1.png' | relative_url }})

{% highlight html %}
{% raw %}
![Battery Widget]({{ '/assets/images/batWid1.png' | relative_url }})
{% endraw %}
{% endhighlight html %}

- centered images with caption (optional):
 
![img]({{ '/assets/images/deer.jpg' | relative_url }}){: .center-image }*(°0°)*

{% highlight html %}
{% raw %}
![img]({{ '/assets/images/deer.jpg' | relative_url }}){: .center-image }*(°0°)*
{% endraw %}
{% endhighlight html %}

You can apply your own styles to image by creating css class with style:

{% highlight css %}
.custom-image-style
{
/* your style */
}
{% endhighlight css %}

And then applying your style just after the image in curly brackets with colon:

{% highlight html %}
{% raw %}
[!image](path to image){:.custom-image-style}
{% endraw %} 
{% endhighlight html %}


________________________



# Accent color

Accent color is color for some important elements, such as links, buttons, icons. Currently accent color is <button class="btn" style="background-color:#3CA2A2; color:#444444">#3CA2A2</button>. This theme has some more predefined colors available in **theme.scss**:

>theme.scss
{:.filename}
{% highlight scss %}
// Several accent colors, choose one or create your own!
$accent-color: #3CA2A2;     // original =)
// $accent-color: #C38FD6;   velvet
// $accent-color: #8FD6B3;   greenish
// $accent-color: #35B4DE;   bluish
// $accent-color: #D2E354;   yellowish
// $accent-color: #52B54B;   green

{% endhighlight %}

You can use one of them (just click the button below to see accent color in action) or define your own!

<button class="btn" style="background-color:#C38FD6; color:#444444">#C38FD6</button>, <button class="btn" style="background-color:#8FD6B3; color:#444444">#8FD6B3</button>, <button class="btn" style="background-color:#35B4DE; color:#444444">#35B4DE</button>, <button class="btn" style="background-color:#D2E354; color:#444444">#D2E354</button>, <button class="btn" style="background-color:#52B54B; color:#444444">#52B54B</button>.
 
<script>
  $('.btn').click(function(){
    var color = $(this).text();
    [].forEach.call($('a'), function(item) {
      item.style.color = color
    })
  })
</script>

<style>
  .label{
    cursor: default;
    border-radius: 5px;
    padding: 5px 8px;
  }
</style>

# Other colors

As Jekyll comes with support of SASS I put colors in variables. Here are the ones which could be easily changed:

>theme.scss
{:.filename}
{% highlight scss %}
$font-color: #dddddd;
$background-color: #292929;
$post-panel-color: #444;
$footer-background-color: #292c2f;
$note-color: #87CEFA;
$warning-color: #ffff00;
{% endhighlight %}

# Background

It is also possible to change the background pattern and color. This theme comes with few patterns pre-installed -- you can check them by clicking on the images below. Or check the [transparenttextures.com](https://www.transparenttextures.com/) -- it has tons of different patterns for background.

<style>
.pattern-list{
    list-style-type: none;
    padding: 0;
}
.pattern{
    height: 100px;
    box-shadow: 0 0 3px 2px rgba(0,0,0,.1);

}
.pattern:hover {
    box-shadow: 0 0 3px 2px rgba(0,0,0,.3);
    transition: box-shadow .2s ease;
    cursor: pointer;
}
.smthg{
    max-width: none !important;
}
.col-sm-6 {
    padding: 5px !important;
}
</style>

<ul class="pattern-list">
<li class="col-sm-6"><div class="pattern" style="background-image:url('{{ '/assets/css/pics/background/3px-tile.png' | relative_url }}')"></div></li>
<li class="col-sm-6"><div class="pattern" style="background-image:url('{{ '/assets/css/pics/background/asfalt-light.png' | relative_url }}')"></div></li>
<li class="col-sm-6"><div class="pattern" style="background-image:url('{{ '/assets/css/pics/background/black-linen.png' | relative_url }}')"></div></li>
<li class="col-sm-6"><div class="pattern" style="background-image:url('{{ '/assets/css/pics/background/food.png' | relative_url }}')"></div></li>
<li class="col-sm-6"><div class="pattern" style="background-image:url('{{ '/assets/css/pics/background/gplay.png' | relative_url }}')"></div></li>
<li class="col-sm-6"><div class="pattern" style="background-image:url('{{ '/assets/css/pics/background/green-dust-and-scratches.png' | relative_url }}')"></div></li>
<li class="col-sm-6"><div class="pattern" style="background-image:url('{{ '/assets/css/pics/background/hexellence.png' | relative_url }}')"></div></li>
<li class="col-sm-6"><div class="pattern" style="background-image:url('{{ '/assets/css/pics/background/random-grey-variations.png' | relative_url }}')"></div></li>
<li class="col-sm-6"><div class="pattern" style="background-image:url('{{ '/assets/css/pics/background/shley-tree-1.png' | relative_url }}')"></div></li>
<li class="col-sm-6"><div class="pattern" style="background-image:url('{{ '/assets/css/pics/background/subtle-grey.png' | relative_url }}')"></div></li>
<li class="col-sm-6"><div class="pattern" style="background-image:url('{{ '/assets/css/pics/background/xv.png' | relative_url }}')"></div></li>
<li class="col-sm-6"><div class="pattern" style="background-image:url('{{ '/assets/css/pics/background/triangles.png' | relative_url }}')"></div></li>
</ul>

<script>
  $('.pattern').click(function(){
    var source = this.style.backgroundImage;
    document.getElementsByTagName('body')[0].style.backgroundImage = source;
    console.log("url('" + source + "'))");
  })
</script>

To change the color go to the **theme.scss** and change the `background-pattern` variable to the name of the pattern image file. To use custom pattern, download it from [transparenttextures.com](https://www.transparenttextures.com/) and place it under **css/pics/background/**.

>theme.scss
{:.filename}
{% highlight scss %}
// use this or pick any from /css/pics/background folder or from transparenttextures.com
$background-pattern: 'random-grey-variations.png';
{% endhighlight %}


________________________________________
