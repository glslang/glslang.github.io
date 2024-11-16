## Latest Posts

{% for post in site.posts %}
* {{ post.date | date: "%Y-%m-%d" }} - [Revisiting Windows Kernel Shellcode on Windows 11: Stack Buffer Overflow with ACL Edit]({{ post.url | prepend: site.baseurl }})
{% endfor %}