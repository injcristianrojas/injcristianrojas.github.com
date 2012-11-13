---
layout: page
title: Bienvenidos
tagline: al blog de Seguridad de Software
---
{% include JB/setup %}

Mi nombre es [Cristián Rojas](https://www.github.com/injcristianrojas), y llevo un tiempo dedicándome a la protección de aplicaciones, evaluación del estado de su seguridad y
educando a presentes y futuros desarrolladores profesionales e ingenieros de software acerca de conceptos, técnicas e importancia
de integrar seguridad a las aplicaciones que éstos desarrollan.

Es por eso, y aprovechando las capacidades que ofrece GitHub, decidí lanzar este blog, en el cual me gustaría comentar a la comunidad
acerca de temas que tienen que ver con la disciplina de la Seguridad de Software: Casos relevantes, técnicas y opiniones al respecto.
Sean todos bienvenidos :)
    
## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


