### Timeline:

<ul>
  {% for post in site.posts %}
    <li>
      {{ post.date | date: "%Y-%m-%d" }} 
      <a href="{{ post.url }}"> {{ post.title }}</a> 
      [
      {% for tag in post.tags %}
        <a href="/tag/{{ tag }}.html">{{tag}}</a> 
      {% endfor %} 
      ]
      {{ post.excerpt }}
      <h4><a href="{{ post.url }}">Read more</a></h4>
    </li>
    <hr>
    <hr>
  {% endfor %}
</ul>

