---
title : ¿Todas las vulnerabilidades se vuelven obvias?
layout: post
category : analisis
tags : [opensource, analisis estatico, fuzzing, pruebas, testing]
---
{% include JB/setup %}

Hace poco me llegó la noticia de una aplicación de lista de regalos desarrollada en PHP la cual tuvo una
vulnerabilidad SQL Injection la cual estuvo en el código durante...
[¡8 años sin ser mitigada!](http://www.h-online.com/security/news/item/Post-from-the-past-security-fix-after-8-years-1762101.html)
Con eso me acordé de aquella frase clasica del mundo opensource:
["given enough eyeballs, all bugs are shallow"](http://www.catb.org/~esr/writings/cathedral-bazaar/cathedral-bazaar/).
Esto me da el pie para referirme a dos cosas en el post de hoy: La influencia del opensource sobre la
seguridad del software y la necesidad de realizar pruebas de seguridad.

##¿Es opensource más seguro?

Los defensores del opensource (no se equivoquen, yo soy uno de ellos de hecho... me refiero a los más
fundamentalistas) dicen que el hecho de que el código de una aplicación esté disponible públicamente
significa automáticamente que la aplicación es más segura, ya que cualquier vulnerabilidad que ésta
pueda tener, está expuesta al público, por lo tanto alguien la verá y la reportará, o mejor aún, la
corregirá.

Hay tres inconvenientes con esa lógica:

1. El primero es el hecho que revisar código es tedioso. Los _ojos
que miran_ no van a mirar todo el código. Quizás módulos o clases específicas, pero no todo el código.
2. El segundo es que no sabemos si estos _ojos que miran_ tienen los conocimientos adecuados en seguridad.
Ya lo había mencionado [al partir este blog](/opinion/2012/11/15/Primera-medida-de-mitigacion/): La mayor
parte del tiempo no tenemos el training respecto de vulnerabilidades, formas de mitigación y mucho menos
de gestión de seguridad en proyectos de software. Por lo tanto es muy poco probable que los _ojos que
miran_ encuentren alguna vulnerabilidad en el código (y mucho menos todas ellas).
3. El tercer inconveniente tiene que ver con que, si bien los proyectos son públicos, el acceso a modificar
ese código está a cargo de los administradores del proyecto, quienes pueden (o no) hacer público el acceso a
modificarlo. El tema es que si un externo descubre una vulnerabilidad y la reporta a un proyecto de acceso
limitado a modificaciones,
[ya sea en forma abierta o responsable](http://www.lnds.net/blog/2012/11/disclosure-no-es-llegar-y-hacerlo.html),
depende de quienes lleven el proyecto adelante el corregir o no la vulnerabilidad. Y también está el tema
de los tiempos, porque puede ocurrir que quien reporta la vulnerabilidad decide llamar la atención de
los administradores en forma inadecuada. Un ejemplo de esto es
[el caso GitHub, en marzo de 2012](https://gist.github.com/1978249), cuando un hacker ruso descubrió
una vulnerabilidad en Rails, la reportó a los administradores del proyecto y luego realizó una prueba
de concepto atacando el sitio web GitHub (hecho en Rails) dos días después.

Entonces, ¿es opensource más o menos inseguro qué código cerrado?

Si tomamos el análisis de recién, hay que considerar que, de partida, el tercer inconveniente es prácticamente el
mismo en código cerrado, con la diferencia de que quien reporta encuentra la vulnerabilidad en el software en
funcionamiento y no en el código, lo que hace más difícil su ubicación y posterior mitigación.

Sin embargo, hay un tema con los otros dos inconvenientes: Uno esperaría que, al ser aplicaciones desarrolladas
por organizaciones privadas, de partida habrían ojos dedicados específicamente a mirar código buscando
vulnerabilidades y además estos ojos tendrían los conocimientos en seguridad adecuados para ubicarlas y mitigarlas.
Y la verdad, es que no sabemos bien si las organizaciones tienen personal dedicado a eso, por lo tanto estamos
en la penumbra al respecto.

##¿Probar la seguridad?

Lamentablemente, probar la seguridad de un sistema no es tan simple como hacer pruebas de sistema (por ejemplo,
[pruebas de unidad](https://en.wikipedia.org/wiki/Unit_Testing)). Mejor dicho, si ya de por sí es dificil hacer
pruebas de un sistema, hacer pruebas de seguridad lo es más aún.

Uno de los problemas con las pruebas de seguridad es la automatización. Desarrollar pruebas de unidad es escribir
un código relacionado o adosado a un módulo, el cual se ejecuta como parte de un proceso de integración continua
antes de la compilación. Sin embargo en seguridad no se tiene esa ventaja. Las pruebas de seguridad se dividen en
dos tipos:

* __Pruebas internas o de caja blanca__: Revisión de código, generalmente con herramientas como analizadores
estáticos.
* __Pruebas externas o de caja negra__: Pruebas sobre los binarios o servicios web expuestos, a través de pruebas
de penetración (el tan conocido _Ethical Hacking_) o herramientas automatizadas como fuzzers.

Como la seguridad absoluta es imposible de alcanzar, siempre es mejor aplicar ambos tipos de pruebas, y mientras más,
mejor. Sin embargo los analizadores estáticos y fuzzers podrían no incluir todo el conocimiento sobre seguridad
necesario, además, presentan un problema muy parecido al
[problema de la parada](https://es.wikipedia.org/wiki/Problema_de_la_parada), el cual es descrito por el
[Teorema de Rice](https://en.wikipedia.org/wiki/Rice%27s_theorem):

> "Cualquier pregunta no trivial que hagas acerca de un programa puede ser reducido al problema de la parada"

Una pregunta no trivial acerca de un programa podría ser perfectamente __"¿Qué vulnerabilidades tiene mi programa?"__.
Producto de esto, una herramienta automatizada podría ser tener sólo dos de las siguientes características: rapidez,
confiabilidad o bajo costo. No las tres.

Aún así, la seguridad es un juego en el cual gana, no quien logra que los sistemas sean impenetrables, sino quien
le hace la vida más difícil al cracker, cosa de que prefiera atacar otro objetivo menos costoso. Por eso, si bien
estas herramientas no son la panacea, integrarlas en el proceso de desarrollo es en extremo importante, así nos
evitamos problemas como el de la lista de regalos.

Felices fiestas :D