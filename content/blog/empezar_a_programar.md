+++
title = 'Quiero aprender a programar ¿Por donde empiezo?'
date = 2024-05-31T13:36:18+02:00
draft = false
+++

En algun momento te has hecho esa pregunta, te la estás haciendo o has visto en algun
foro a alguien preguntandose eso y pidiendo ayuda. Y siempre veo las mismas respuestas repetidas como loros.

Bien en este blog voy a aportar mi granito de arena y si todo sale bien saldrás de aquí sin ninguna duda,
así que empecemos.

## ¿Que quieres programar?
Cara al público parece que programar sea igual en todos lados pero no, hay muchas ramas dentro de la programación.
Si quieres aprender a programar primero decide cual de ellas es la que te interesa. Una vez lo sepas, sabrás
que lenguajes de programación o herramientas en general tendrás que aprender.

**SPOILER ALERT** el código va a ser la última de tus preocupaciones.

### Web
Esta es la mas popular hoy en dia y se divide en dos areas: **Frontend** y **Backend**.

#### Frontend:
El desarrollador frontend es el responsable de programar las aplicaciones web que se ejecutan en los navegadores.
Chrome, Edge, Firefox, etc. Su responsabilidad principal es crear aplicaciones que sean amigables con los usuarios
intuitivas y conectarse con los servicios que programa el backend.

Las aplicaciones web frontend se programan con HTML, CSS y JavaScript. HTML y CSS son los que se encargan de darle el
contenido y diseño a una aplicacion respectivamente. Mientras que con JavaScript le das la interactividad a tu aplicación.

Hoy en dia las aplicaciones se han vuelto muy complejas y lo que antaño se podia hacer con HTML, CSS y JS hoy en dia
requiere de librerias y frameworks que facilitan el trabajo a los programadores. Los mas conocidos son
React, Angular y Vue. Aunque no son las unicas herramientas que te puedes encontrar. El front es conocido
por su inmensidad y diversidad de herramientas. Así que esto no es mas que el punto de partida.

#### Backend:
El backend puede abarcar muchos ambitos pero basicamente es la parte que no se muestra directamente a los usuarios.
Cada vez que usas una aplicación como Twitter, Spotify, Netflix, Amazon, Google, etc. Todas tienen algo en comun...
Todas tienen servidores que se encargan de procesar y ejecutar las acciones de los usuarios. Y esa es la responsabilidad
de los desarrolladores backend (junto con otros profesionales).

Los back son los responsables de procesar la información
de la aplicación, establecer las reglas de seguridad (cosas como que un usuario no pueda borrar el contenido de otro),
gestionar la seguridad de la aplicación, entre otras tareas. Todo lo que tiene que ver con: gestion de datos, seguridad,
monitoreo, etc. Es responsabilidad de los backend.

La variedad de lenguajes y tecnologias a escoger es un mundo. Hay muchísimas opciones para escoger. Sin embargo dependiendo
del tipo de aplicación a desarrollar o el tamaño de esta, hay mejores opciones que otras. Algunos ejemplos son: **JavaScript**,
**Java**, **C#**, **Ruby**, **Python**, **PHP**, **Go**, etc. Mi recomendación es que investigues que lenguajes hay y que opciones ofrece cada uno.

### Aplicaciones de escritorio
Puede abarcar muchos tipos de aplicaciones: navegadores web, editores de video, lectores de PDF, gestores de tareas, etc.
Al final es cualqueir aplicación que se ejecute en un ordenador con acceso a los recursos del sistema operativo.

Puedes usar herramientas que te garantizan un rendimiento excelente como **C/C++** o **Rust**. O usar herramientas que te
facilitan el desarrollo de interfaces gráficas como electron: Que es un framework de **JavaScript**. Otras opciones
es usar el lenguaje que mejor se adapta para el sistema operativo que te interesa. C# para windows, Swift para Mac, C/C++
para Linux. Hay muchas opciones pero para que te hagas a una idea de que tipo de herramientas te puedes encontrar, te
lo dividire en dos tipos.

#### Multiplataforma
Te permiten crear aplicaciones que con una misma base de código puedes ejecutar tu aplicación en varios sistemas operativos.
Electron, QT y Flutter son algunos ejemplos.

#### No tan multiplataforma
Són tecnologias que, solo tienen soporte para un solo sistema operativo, para unos pocos o si lo tienen para varios
este es limitado. Swift para Mac, C# para Windows, GTK para Linux. Si solo quieres hacer aplicaciones para un solo
sistema operativo puede que te salgan rentables ya que por norma general tienen muy buena integración con el sistema
operativo para el que están diseñados.

### Aplicaciones móviles
Los desarrolladores moviles son los responsables de hacer las aplicaciones que se ejecutan en los telefonos.
A los desarrolladores móviles, del mismo modo que los desarrolladores de aplicaciones de escritorio, podemos dividirlos
en nativos y multiplataforma. Al igual que con el front su responsabilidad
es hacer aplicaciones que sean amigables con los usuarios y que se conecten con los servicios programados por los backend.

#### Nativos
Los desarrolladores nativos
son aquellos que usan los lenguajes y SDKs que ofrecen los propios desarrolladores de los sistemas operativos moviles.
Estos SDKs son específicos para cada sistema operativo y una aplicación programada para iOS usando sus SDKs nativos,
no sera compatible con Android y viceversa.

Para iOS se usa **Swift** (antiguamente **Objective-C**) y herramientas como SwiftUI para crear las aplicaciones que tenemos en iOS.

Para android se usa **Kotlin** (antiguamente **Java** pero cada vez menos). Y a dia de hoy la opción mas moderna es jetpack compose
aunque todavia se hacen aplicaciones usando XML para definir las interfaces graficas.

#### Multiplataforma
Los desarrolladores moviles multiplataforma. Usan herramientas que permiten reutilizar código o directamente hacer
aplicaciones que se ejecuten en multiples sistemas operativos con una sola base de código. **Flutter** (Dart),
**Kotlin multiplatform** (Kotlin), **Ionic** (JavaScript), **React native** (JavaScript), **Maui** (C#)... Son los ejemplos mas conocidos.

Si bien permite que con menos código y trabajadores se pueda obtener una aplicación que funcione en varios sistemas
operativos, no suelen ser las mas rápidas y es un factor a tener en cuenta.

### Sistemas embebidos
Hacer progamas para sistemas embebidos implica trabajar muy de cerca con el hardware de los dispositivos electronicos.
Es hacer firmwares, sistemas operativos, drivers y todo aquel software que se encargue de controlar el hardware
de cualquier dispositivo. También puedes encontrarte con programar sistemas industriales como PLCs.

Los lenguajes mas usuales son: C/C++, Assembly, Rust, etc. El más dominante aquí es C, así que si tienes interes en
programar a bajo nivel, es un buen punto de partida.

### Videojuegos
No hace falta presentación todos hemos soñado con hacer videojuegos. Debido a que es una area muy amplia ya que puede
ser desde hacer shaders, motores gráficos, librerias de gráficos en 3D, hacer videojuegos con motores ya programados...

Me voy a centrar en los desarrolladores de juegos como tal. Estos suelen usar motores gráficos ya programados. Los mas famosos
son Unity y Unreal Engine. Hay otras opciones como Cry Engine o Godot pero los que dominan la industria son Unity y Unreal.

Unity te permite hacer scripts con **C#** mientras que Unreal te permite hacerlo con **C++**.

Si quereis mi opinión personal yo optaría por Unreal antes que Unity. Debido a escandalos por sus politicas de monetización
en los últimos años optaria por una opción mas estable y potente como Unreal. Aunque si no tienes interes en hacer juegos
en 3D con gráficos hiperrealistas, prueba Godot. Puedes hacer scripts con C# o GDScript.

### Automatización
La palabra es muy ambigua pero para que veas a lo que me refiero voy a ponerte un ejemplo. Imagina que tienes un montón
de imágenes en tu ordenador que has bajado de internet a alta resolución. Para que no ocupen mucho espacio en disco decides
comprimir estas imágenes. Puedes hacerlo una por una o hacer un script que lo haga todo por ti. O si tienes que
hacer un healthceck periódico de tus servidores y quieres mandarlo por correo también lo puedes automatizar.

Normalmente esta ligado a procesos de gestión y es útil para no perder tiempo con tareas monotonas. Los lenguajes mas usados
són Python, Ruby, Bash, Perl, etc. Son lo que se conoce como lenguajes de scripting.

### IA, ciencias de datos y Big Data
He decidido meterlos a todos en el mismo saco porque van muy de la mano. Mientras que la IA és una area muy amplia y compleja
trata de crear modelos que aprendan solos y evolucionen con el tiempo, el Big Data está más enfocado en el procesamiento
de masivas cantidades de datos. Las ciencias de datos es por así decrilo lo que une estas dos ramas.

En IA, vas a necesitar un sólido conocimiento de matemáticas, en IA el lenguaje mas popular sin duda alguna es Python.
A menudo en ciencias de datos y Big Data se suelen usar modelos de IA para procesar todos estos datos.

Big Data va más ligado al procesamiento de datos y si que es mas técnico en este sentido. Trabajas con bases de datos
y creas algoritmos para poder extraer información útil de datos masivos.

## No todo es código
Te lo dejaré muy claro. El código es fácil no tiene mucho misterio. Lo jodido es adquirir esos conocimientos que van
mas ligados fundamentos matemáticos (para las ramas mas cientificas) o que van ligados a como se crean esas megas aplicaciones
que dan soporte a miles de usuarios por minuto.

Además que en muchos casos vas a tener que aprender a usar otras herramientas que se complementan con el código.
Sistemas de control de versiones, bases de datos, herramientas de depuración, herramientas de profiling para analizar
el rendimiento de tus programas, testing y control de calidad, etc.

Programar es una actividad relativamente senzilla mientras que desarrollar software requiere que sepas programar entre
otras cosas.
