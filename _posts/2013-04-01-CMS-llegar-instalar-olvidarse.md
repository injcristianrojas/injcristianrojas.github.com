---
title : CMS... Llegar, instalar, ¿y olvidarse?
layout: post
category : analisis
tags : [cms, opensource, soporte]
---
{% include JB/setup %}

Una de las típicas formas de mostrar contenidos en Internet es el uso de un
[CMS](https://es.wikipedia.org/wiki/Sistema_de_gesti%C3%B3n_de_contenidos)
(Content Management System). Estos sistemas, cuyos representantes más
conocidos son [Drupal](https://drupal.org/), [WordPress](https://wordpress.org/),
[Joomla!](http://www.joomla.org/), etc. La mayor parte de ellos es
gratuito (opensource), incluyen instaladores e instrucciones que permiten
que su instalación, configuración y administración de contenidos sea lo más
simple posible.

Sin embargo, esto trae un inconveniente. Muchas veces quienes instalan
sistemas CMS, los instalan, comienzan a ingresarles contenido y se olvidan
de un aspecto fundamental: El soporte.

El software evoluciona. Y tiene que hacerlo, ya que estos CMS son expuestos
a Internet, que es un mundo peligroso. Cada día aparecen vulnerabilidades
las cuales afectan a estos CMS y pueden ser aprovechadas por inescrupulosos.
Y no es demasiado complicado, ya que existen herramientas automatizadas que
permiten descubrir el CMS que se está usando y su versión, a través de
características como la estructura de directorios del sitio, y ni siquiera
eso, ya que sus páginas incluyen un simple marcador HTML como éste:

    <meta name="generator" content="WordPress 3.5" />

En base a esa información, los crackers pueden buscar en los avisos de
seguridad qué vulnerabilidades hay para cada versión de la plataforma, y cómo
atacarlas.

Pero eso no es todo. Los CMS generalmente permiten añadir nuevas funcionalidades
y tipos de contenidos a través de plugins desarrollados la mayor parte del
tiempo por terceros, quienes hacen uso de la API del CMS pero la mayor parte
del tiempo no están preocupados de hacer uso de las funciones seguras de ésta (si
es que las hay). Esto da pie para el uso de aplicaciones que escanean sitios
buscando plugins vulnerables, como [WPScan](http://wpscan.org/) para WordPress.

Esto da pie para que ocurran varios
[defacements](https://en.wikipedia.org/wiki/Website_defacement) (el equivalente
digital del rayado de muros) como el ocurrido con el caso del sitio web de
[TECHO](http://www.elobservatodo.cl/noticia/sociedad/pagina-de-techo-fue-hackeada-por-revolucion).
Pero no sólo eso, ya que estos ataques pueden pasar de algo "inocuo" como un
defacement a algo peor, como robar información o instalar malware el cual dará
más de algún dolor de cabeza a quienes administran el sitio, ya que si no lo
pilla [Google](https://en.wikipedia.org/wiki/Google_Safe_Browsing) antes, quienes
ingresen al sitio se verán infectados.

Por esto es importante tener en cuenta lo siguiente, al momento de administrar
un CMS:

 1. Mantener, tanto el sitio como sus plugins, actualizados.
 2. Estar atento a los boletines de seguridad, tanto del CMS usado, como los
del [CVE](https://cve.mitre.org/), que generalmente reporta vulnerabilidades
relativas a los plugins.
 3. Respecto al punto anterior, si un plugin es reportado como inseguro y no
hay alguna actualización que corrija la vulnerabilidad, deshabilitarlo de
inmediato.

Algunos recursos interesantes:

 * [Joomla! Security](http://docs.joomla.org/Security)
 * [Drupal Security Advisories](https://drupal.org/security)
 * Para WordPress: [Guía para _endurecer_ WordPress](https://codex.wordpress.org/Hardening_WordPress)
y un [libro gratuito](http://build.codepoet.com/2012/07/10/locking-down-wordpress/)
para mantener bajo control la seguridad del CMS más popular de la red.
