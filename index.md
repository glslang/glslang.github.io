## Latest Posts

{% for post in site.posts %}
* {{ post.date | date: "%Y-%m-%d" }} - [{{ post.title }}]({{ post.url | prepend: site.baseurl }})
{% endfor %}