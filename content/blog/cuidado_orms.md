+++
title = 'Cuidado con los ORM'
date = 2024-08-14T20:38:33+02:00
draft = false
+++

Una de las cosas que más nos toca los huevos a los programadores es manejar bases de datos. Allí es donde solemos tener
la mayoría de problemas de rendimiento y muchos profesionales o no saben lidiar con SQL o no quieren. Es por eso que en
muchos casos, es normal ver proyectos que usan ORMs. Con el tiempo cada vez se han vuelto más completos y prácticamente
te hacen la mitad del trabajo.

Pero cuando tu aplicación empieza volverse más compleja o empieza a tener una mayor demanda de peticiones
es cuando esa librería que a priori te estaba ahorrando mucho tiempo y trabajo ahora te está causando muchos problemas
, está provocando retrasos y te genera el doble de trabajo.


## El problema
Si bien es cierto que con el tiempo los ORMs pueden volverse cada vez más completos y eficientes, aquí hay un problema de base.
Si tu aplicación depende de la librería con la que te conectes a la base de datos, **tienes un problema de arquitectura**.
Idealmente en el mundo de la POO una aplicación debería ser modular y no tener acoplamiento a ningún sistema externo.
Es decir, para que tu aplicación sea fácil de mantener, testear y escalar, esta debería ser agnóstica
a cualquier detalle de implementación y debería tener unos contratos que definan como se deben comportar esos módulos
que quieres conectar a tu aplicación.

El problema es que si toda la lógica de algo tan importante como el sistema de persistencia se centra a nivel contractual en como está diseñado
el ORM, dependes de las decisiones que tome el equipo que está detrás de esa librería. Un equipo que no trabaja para tu empresa
y que tiene que dar soporte a un proyecto que usan miles de desarrolladores.

Estos te facilitan el *bootstraping* de tu aplicación, pero no te garantizan ni un buen rendimiento, ni mantenibilidad ni
escalabilidad. Pero para MVP, pruebas de concepto o incluso aplicaciones pequeñas o medianas son de lo mejor que hay.

Entonces, como eres tú quien está a cargo de la aplicación que tiene que dar solución al problema de tu empresa, tienes
que ser tú como desarrollador quien adapte el ORM a las necesidades concretas de tu proyecto.

## Consideraciones al usar un ORM
Las bases de datos tienen características únicas. Muchas veces usamos una base de datos porque tiene tablas y podemos hacer queries
no hay más que añadir. Pero a medida que tu aplicación crece, te vas dando cuenta de que hay herramientas que te facilitan
mucho el trabajo que son únicas en cada base de datos y que no están disponibles en muchos ORM. Ya que estos mayoritariamente
buscan hacer una API única para diferentes bases de datos y por el camino tienen que sacrificar muchas funcionalidades.

Por ejemplo, no tienen soporte directo para vistas. Cuando trabajamos con queries complejas, las vistas nos permiten simplificar
el acceso a nuestros datos unificando una query que puede ser todo lo compleja que tú quieras en un simple `select`. Esto
no está soportado por la mayoría de ORMs. Mucho menos soportan cosas como vistas materializadas, triggers o hacer ciertas
automatizaciones en tu base de datos.

Cuando tienes que hacer queries complejas, recurres a usar SQL puro y duro a través de tu ORM. Eso te permite hacer él
trabajo, pero luego pierdes ese mapeo a objetos de tu aplicación, por lo que en esos casos complejos no te sirve de nada ese ORM.

Además, muchos de ellos te proveen de sistemas de migración de cambios o datos. Estos te permiten replicar tu base de
datos locales en producción y permite que el resto del equipo trabaje con los últimos cambios en la base de datos. Todo
gestionado por el ORM en cuestión. Esto si bien es cierto que es un puntazo a la larga, implica que la estrategia de
migración de datos de tu aplicación depende del ORM que estés usando. No es malo per se, pero puede volverse limitante en
algunos casos.

## ¿Lo uso o no lo uso? Esa es la cuestión
Aquí mi propuesta es simple. Úsalo, pero desacopla tu ORM de tu aplicación. Una manera de lograr esto es usando patrones
como DAO o Repository. Te permiten abstraer tu sistema de persistencia a la vez que te permiten separar la infraestructura
de la lógica de tus controladores, casos de uso, servicios, etc. Esto permite acotar y reducir la complejidad de tus tests.

Puedes seguir aprovechando las ventajas de tu ORM, pero permitiendo que tu aplicación no esté acoplada a este. Eso en
lo que respecta a acceso a datos. Si bien puedes aprovechar las herramientas de migración de datos, ten en cuenta que
estas no son una bala de plata. Lo que sí que puedo decir es que independientemente de que framework o librería uses para
gestionar los cambios de tu base de datos hay que seguir unas buenas prácticas que son innegociables.

Las migraciones de datos son un tema complejo y lo que no recomiendo es que todo ese proceso este centrado en una pieza
que el día de mañana haya que reemplazar. Pueden ser útiles al principio, pero son herramientas y son reemplazables.
Como bien dije, no son balas de plata y en algún momento puede ser que te encuentres con los límites de esa librería.

Las bases de datos son una parte crucial del software a día de hoy y lo que no es para nada una buena idea es delegar
el trabajo de la administración y diseño de estas a un software externo. Las bases de datos muy probablemente son junto con él
capital lo que le da de comer a tu empresa. Por lo que saber cuáles son los límites de las herramientas que usas a diario,
cuáles son las necesidades de tu empresa y de tu equipo son los criterios que tienes que tener en cuenta para saber si
es buena idea o no usar un ORM, como usarlo y como agilizar el desarrollo de tu aplicación con este.
