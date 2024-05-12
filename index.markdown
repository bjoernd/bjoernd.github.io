---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

## Bjoern

In my day job I'm a Principal SDE at Amazon, working with our kernel, hypervisor and security teams. If you're interested in these kinds of things, please reach out on [LinkedIn](https://www.linkedin.com/in/bjoerndoebel/).

Beyond that, I spend my free time dabbling with Gaming, 3D printing, Linux, and other interesting technologies.


## Blog Posts
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.date | date: "%Y-%m-%d"}} -- {{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
