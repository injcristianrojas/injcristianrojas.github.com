---
title : ¿Estamos protegiendo bien los secretos de nuestros usuarios?
layout: post
category : howto
tags : [code, passwords]
---
{% include JB/setup %}

Hace unos meses atrás apareció un caso muy grave que comprometió a millones de usuarios de LinkedIn. En un foro
ruso fueron publicados varios hashes derivados de las passwords de ese sitio, todas codificadas en SHA1 y sin
una sal apropiada. LinkedIn poco después parchó su aplicación, rehasheando las passwords y aplicandoles una sal,
a lo cual llamaron _medidas de seguridad mejoradas_.

Más recientemente, ChileVisión recibió un ataque por parte de un grupo que se hacía llamar LulzSec Perú, en
represalia por el trato recibido por un nativo de allá en el programa Primer Plano. Posterior a este ataque, los
peruanos publicaron el dump de la base de datos del sitio web de CHV, en las cuales estaban codificadas las
passwords usando MD5 (Por el amor de Dios,
[¿aún hay gente que usa eso?](http://www.lnds.net/blog/2012/06/flame-y-md5.html)) y sin sal tampoco.

## ¿Le saco la sal?

Si bien los algoritmos de hashing son undireccionales y algunos son más resistentes que otros a ataques de colisión,
todos, absolutamente todos son vulnerables a un ataque llamado Tabla Arcoiris, el cual consiste en el uso de una
base de datos gigantesca de que contiene hashes con sus textos planos asociados.

Para revertir esta situación, se ideó un mecanismo que consiste en almacenar en el registro del usuario en la base
de datos un string generado aleatoriamente llamado _sal_. Esta sal se concatena con la password del usuario y el hash
resultante se almacena en la base de datos.

Esto quiere decir que, si antes se almacenaba sólo el hash de la password en el registro del usuario en la base
de datos:

	sha224('CqynrsbESpb&n.') = 'e4acfbf93decda1a9c1825412f1a0192257ee534f44f1a98be3c2879'

Al usar una sal nosotros estaríamos almacenando el siguiente verificador de passwords en la base de datos:

	sha224('5yxd0nPWJwU' + 'CqynrsbESpb&n.') = 'f6a218ad75d6e45fa9bd336ccdebb37aff3bf241dca4582b74648f61'

Esto permite añadir entropía y una mayor dificultad a un ataque de tabla arcoiris que la que habría si un usuario
usara (como suelen los usuarios hacerlo) una password como ésta:

	sha224('cristian123') = '3cf0c3184d53f362b03c357b7bf7d4784b3994c1085c871e976d28c8'

## La velocidad está sobrevalorada

Si bien el mecanismo de la sal es bueno, se puede mejorar. Una de las máximas en seguridad es no hacerlo seguro 100%,
sino que hacerle el trabajo lo más costoso posible a un atacante, cosa que desista, vaya al siguiente objetivo y nos
deje tranquilo. Y también el hacernos a nosotros el trabajo un poco más fácil. Para eso se crearon las funciones
de derivación de claves (Key Derivation Functions o KDF).

Estas funciones utilizan sus propias funciones de hash pero con una salvedad: Primero toman el texto plano, le aplican
la sal y lo hashean, toman ese hash, le vuelven a aplicar la sal y lo vuelven a hashear, toman de nuevo el hash, le
vuelven a aplicar la sal y lo vuelven a hashear... y así sucesivamente una determinada cantidad de veces, definida por
defecto, pero modificable. Así generan lo que se conoce como verificador de password. Esta repetición, llamada
_estiramiento de claves_, hace que la función sea lenta, y por una razón.

Una de estas funciones es Bcrypt, una función muy parecida a la que se usa para almacenar passwords en sistemas basados
en UNIX. El algoritmo toma en ejecutarse 0.1 segundos, algo que un usuario intentando loguearse quizás no notaría, pero
si un atacante obtiene el verificador de la password e intenta realizar un ataque de fuerza bruta sobre éste utilizando,
supongamos, un espacio de claves posible basado en el alfabeto español (28 letras contando el espacio) y con 8 caracteres
posibles máximo, le tomaría la friolera de 28^8 * 0.1 / 86400 = 326886 días recorrerlo completo utilizando esta función.

La otra ventaja de este sistema es que el verificador incluye la sal dentro del string, por lo tanto nosotros no tenemos
que preocuparnos de crear una sal y ponerla en otra columna del registro del usuario. Por ejemplo, el algoritmo PBKDF2
define toda la información necesaria, separada por $: Primero la versión del algoritmo, luego la sal, y finalmente el
verificador estirado, como se ve en el ejemplo:

	pbkdf2('cristian123') = '$p5k2$$8GUEN72T$n5/K4f5cCeZIQXJkNeQCioVsM/g2kpT6'

## Una nota acerca de passwords perdidas

Un error garrafal que cometen muchos sitios es el de almacenar passwords en texto plano. Eso se nota cada vez que un
usuario pierde su password y la solicita al sitio web, quien automáticamente le manda la password a través del mail.

No sólo eso es inseguro, sino que una innecesaria violación a la privacidad, ya que si bien se supone que las medidas
se toman para evitar intrusiones externas a la base de datos de credenciales de usuarios, eso no evita que usuarios
internos de la aplicación (administradores de sistemas, desarrolladores) tengan acceso a los secretos de los usuarios.

Si el almacenamiento de passwords se hace bien, se usarán los métodos indicados anteriormente, y en caso que el usuario
pierda su password, se puede tomar una de las dos siguientes medidas (de menos a más segura):

* Resetear la password y crear una password temporal generada aleatoriamente, la cual permitirá al usuario ingresar y
cambiarla
* Crear un token y con él un link el cual el usuario ingresa para resetear su password.

## Datos útiles

Si deseas integrar estas técnicas en tus próximos proyectos, aquí te van unos datos interesantes para diferentes 
lenguajes/frameworks:

* **Java/JEE/Android**: Si deseas implementar Bcrypt, usa [jBCrypt](http://www.mindrot.org/projects/jBCrypt/). Para
PBKDF2 está la [_Legión del Castillo Saltarín_](http://www.bouncycastle.org/java.html).
* **iOS (Objective-C)**: Para implementar PBKDF2 sigan [este link](http://stackoverflow.com/questions/5526853/how-to-create-pbkdf2-key-on-ios-device).
Para Bcrypt usen [esta biblioteca](http://www.jayfuerstenberg.com/blog/bcrypt-in-objective-c).
* **Python**: [py-bcrypt](https://code.google.com/p/py-bcrypt/), [PBKDF2](https://www.dlitz.net/software/python-pbkdf2/) ([Django](https://www.djangoproject.com/)
lo tiene incorporado en su módulo de administración de usuarios ;) )
* **Ruby**: [PBKDF2](https://github.com/emerose/pbkdf2-ruby), [Bcrypt](http://bcrypt-ruby.rubyforge.org/)
* **.NET**: PBKDF2 está incluido en la API usando la clase
[System.Security.Cryptography.Rfc2898DeriveBytes](http://msdn.microsoft.com/en-us/library/system.security.cryptography.rfc2898derivebytes.aspx).
Para usar Bcrypt está [esta biblioteca para C#](https://bcrypt.codeplex.com/).
* **PHP**: [PHP-PBKDF2](https://defuse.ca/php-pbkdf2.htm). La versión 5.5 de PHP (en versión alpha actualmente)
traerá una API especialmente diseñada para el uso de KDF's, el cual incluye Bcrypt. Mientras tanto pueden usar
[esta biblioteca](https://github.com/ircmaxell/password_compat).
* **NodeJS**: [Bcrypt](http://hermanjunge.com/post/41863341875/encrypt-with-bcrypt-in-nodejs)