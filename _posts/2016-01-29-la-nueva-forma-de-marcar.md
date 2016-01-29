---
title : La nueva forma de desarrollar apps móviles
layout: post
category : analisis
tags : [mobile, apps]
---
{% include JB/setup %}

En enero de 2016, fue lanzada a las stores móviles la aplicación __Nueva
forma de marcar__, encargada por la Subsecretaría de Telecomunicaciones del
Gobierno de Chile a la empresa de desarrollo móvil Cursor S.A.. En cuestión de
horas, comenzaron a surgir una serie de cuestionamientos respecto de la
privacidad de la aplicación, principalmente generados a partir de informes
publicados por [Zerial](http://blog.zerial.org/seguridad/nueva-forma-de-marcar-analisis-a-la-aplicacion-publicada-por-el-gobierno/) y
[SPECT Research](http://www.spect.cl/blog/2016/01/el-tema-de-la-aplicacion-nueva-forma-de-marcar/). Mi intención hoy no es hacer un "tercer informe" al respecto. De hecho, me
avergüenza un poco el haber llegado tarde a esta discusión. Sin embargo, creo
que puedo aportar en algo a ella, sobre todo considerando que estoy
haciendo mi [tesis de Magister](https://www.dcc.uchile.cl/node/1789) en
seguridad en aplicaciones móviles.

# Algunos datos interesantes

La aplicación creada por Cursor es una aplicación _híbrida_, es decir, está
hecha con HTML5/Javascript. Esta aplicación se encapsula en una aplicación
nativa, la cual se exporta a las plataformas seleccionadas en la forma en que
se desee. Un código, varias plataformas, menor costo. El framework utilizado
en esta ocasión es [Apache Cordova](https://cordova.apache.org/) con una
extensión llamada [ngCordova](http://ngcordova.com/), y esto es un detalle
importante que analizaremos más adelante.

Además, la aplicación, como parte de su uso, envía un conjunto de datos a un
webservice basado en REST/JSON. Al igual que Zerial y SPECT, hice un análisis
basado en ingeniería reversa de la aplicación, y me encontré con un comentario
interesante, el cual me llevó al webservice de prueba de la aplicación, y
basado en eso, se observa que la aplicación envía a los servidores de Cursor
la ubicación geográfica del usuario de la app, el total de contactos y números,
la cantidad de contactos convertidos, un `deviceUUID` (el cual trataremos un
poco más adelante) y un tiempo (mirando el código fuente observé que es el
tiempo que le toma a la app actualizar los contactos en el teléfono del usuario).

<figure style="text-align: center">
    <img src="/assets/img/api_nueva_forma.png" />
    <figcaption>Documentación de la API del Webservice</figcaption>
</figure>

# ¿Es segura la app?

Esta siempre es una pregunta interesante, y en este caso en particular, creo
que es conveniente desglosarla en dos subpreguntas:

## ¿Es segura la app? (valga la redundancia)

Aquí la respuesta es derechamente __no__. La información recolectada desde el
móvil del usuario es transmitida al servidor mediante una conexión HTTP, sin
cifrado alguno. Hoy en día es inaceptable que una app móvil se comunique con
sus servidores sin usar HTTPS (incluso
[Apple está de acuerdo con ésto](http://www.cso.com.au/article/577197/apple-tells-ios-9-developers-use-https-exclusively/)). Un certificado digital no es caro, e incluso se pueden obtener
[gratis](https://letsencrypt.org/). Y los buenos servidores tienen soporte
SSL/TLS. Ahora ojo, una cosa es configurar SSL/TLS y
[otra cosa es hacerlo bien](http://es.slideshare.net/injcristianrojas/https-usted-selo-bien).

Otro tema importante aquí es el abandono de código. Que yo haya podido encontrar
la URL del webservice de prueba para la app significa un problema de abandono
de código bastante grave. Y pasa mucho. He visto muchas apps que dejan código
de log incluido en sus versiones de producción, y esos logs que dejan muchas
veces contienen información sensible.

## ¿La app respeta la privacidad de sus usuarios?

Esta es probablemente la principal razón por la cual me demoré en _meter la
cuchara_ en este tema, ya que compartí los posts de SPECT y Zerial en mis redes
sociales, y eso dio pie a ásperas discusiones respecto de si los datos enviados
a los servidores de la app son datos sensibles o no.

Si tomamos cada dato por sí solo, probablemente no significa nada. Y si uso
ciertas combinaciones de datos (por ejemplo sólo las cantidades de contactos y
la información geográfica) tampoco significa nada. Si podría haber un problema
con el `deviceUUID`. Cabe aclarar que el `deviceUUID` no es el identificador
universal del equipo móvil. Para eso está el IMEI. De acuerdo con la
documentación de ngCordova, el `deviceUUID` es un identificador propio de la
plataforma móvil:

* En Android, es el ANDROID_ID, código de 16 caracteres asignado por Google al
momento de la activación del equipo (cuando se conecta por primera vez a
Internet y el usuario se loguea a su cuenta Google)
* En iOS, es un hash de 40 caracteres basado en varios aspectos del hardware del
teléfono.

OK, hasta este punto, un dato adicional que se puede inferir del `deviceUUID` es
si el equipo es Android o iOS. Sin embargo, ¿es el `deviceUUID` información
sensible?

Como me dijeron tantas veces en la universidad, la respuesta es _depende_. El
`deviceUUID` por sí sólo no permite identificar al usuario del equipo móvil.
Sin embargo, hay un aspecto interesante. El periodista y experto en Seguridad Jacob Appelbaum habla de un concepto llamado [__linkability__](https://www.youtube.com/watch?v=ACI7zZGgi80), que se refiere
a la capacidad de relacionar los datos de un usuario en bases de datos
diferentes y en base a eso identificarlo o hacer un perfil de él. La app no
envía información personal del usuario (nombre, correo electrónico, teléfono,
etc.), pero si la base de datos a la cual van a parar los datos recolectados
se cruza con otras bases de datos que incluyan el `deviceUUID` con información
personal, ahí se produce una situación de linkability y eso ya es un problema de
falta de protección de datos personales.

<figure style="text-align: center">
    <img src="/assets/img/linkability.png" />
    <figcaption>Ejemplo de linkability: A la izquierda, un registro de la base de datos de la app. A la derecha, un registro con el mismo deviceUUID en la base de datos de alguna otra aplicación que House haya usado.</figcaption>
</figure>

En resumen, la recolección de datos por sí sola no es un problema de privacidad,
pero si esos datos son cruzados con otras bases de datos, podría volverse un
problema, y en un país donde las leyes y buenas prácticas de protección de datos
personales son muy débiles, la preocupación de algunas personas (me incluyo) por
la recolección de información y su posterior manejo es muy relevante.

# ¿Cómo se podría haber hecho mejor?

De partida, usando HTTPS para la comunicación entre la app y el servidor.
Segundo, siempre es bueno auditar el código de la app cosa de buscar potenciales
problemas de seguridad, y eliminar código abandonado.

Respecto del `deviceUUID`, si el objetivo al recolectar este dato es el de
"limpiar información estadística" o "saber si la ejecución no está repetida"[^spect],
esto se puede lograr perfectamente generando en el servidor un número
correlativo o aleatorio cuando la app se conecta a éste, y que la app guarde
este número en su área de almacenamiento. Así, si hay un subsecuente uso de la
app por parte del mismo equipo, la app verifica si tiene este número, y si lo
tiene, hay un uso repetido de la app. Nada de envíos de información
sensible.

Otro aspecto importante: Si se busca que "la Subtel evalúe la efectividad de la
campaña de información y promoción de la aplicación"[^spect], y para ello se
requiere la información geográfica del usuario ¿no sería mejor usar la ubicación
aproximada y no la ubicación precisa del usuario al momento de usar la
aplicación? (esto asumiendo que la SUBTEL busca a lo más saber en qué comuna
se encuentran los usuarios al momento de usar la app y no su dirección exacta).

# Y una cosa más...

Ante la polémica, el Subsecretario de Telecomunicaciones, Pedro Huichalaf,
indicó que "si duda del uso de la app, no la use"[^zerial]. Es fácil decir eso
cuando los ciudadanos no tienen todos los antecedentes a la mano. Antes de
emitir ese tipo de declaraciones, y sobre todo tratándose de una app para los
ciudadanos, se debe transparentar detalladamente qué información será
recolectada y para qué será utilizada, para que los usuarios tengan todos los
antecendentes a la mano y puedan decidir si usan la app o no.

[^spect]: Ver el informe de SPECT Research: <http://www.spect.cl/blog/2016/01/el-tema-de-la-aplicacion-nueva-forma-de-marcar/>
[^zerial]: Ver el informe de Zerial: <http://blog.zerial.org/seguridad/nueva-forma-de-marcar-analisis-a-la-aplicacion-publicada-por-el-gobierno/>
