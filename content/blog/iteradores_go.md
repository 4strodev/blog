+++
title = 'Iteradores en go'
date = 2024-08-20T20:27:56+02:00
draft = false
tags = ['go', '1.23']
cardimage = 'images/go_iterators.webp'
featureimage = 'images/go_iterators.webp'
+++

# Iteradores en go
Este agosto ha salido la versión [1.23](https://tip.golang.org/doc/go1.23) de [Go](https://go.dev). Sin lugar a duda de los lanzamientos más controversiales
de nuevas versiones de Go. Él ¿por qué? Pues porque en la versión 1.22 introdujeron de manera experimental los iteradores
y en esta versión la han marcado como una funcionalidad estable.

# ¿Qué son los iteradores?
El patrón iterador es un patrón que permite acceder a elementos de una colección. Abstrayendo el cómo se accede
a estos elementos e incluso pudiendo generarlos al momento.

## ¿Cómo funcionan?
En este caso me voy a centrar en como se crean y se usan iteradores en Go. Si venís de lenguajes como Python o C# puede
que la palabra `yield` os suene.

Los iteradores son una función que tienen alguna de las siguientes cabeceras.
```go
// Suponiendo que K y V son tipos genericos
func(yield func() bool)

func(yield func(V) bool)

func(yield func(K, V) bool)
```

`yield` es la función que procesa el elemento que se está iterando. Si yield devuelve `false` se finaliza la
iteración si devuelve `true` continua con la iteración.

### Como se usan
Los iteradores son una abstracción que permiten iterar datos con un bucle for como si fueran un slice. Esto nos permite
abstraer la lógica de obtención de estos datos y poder plasmar en el bucle el cómo se procesan estos datos.

```go
package main

import (
	"fmt"
	"slices"
)

func main() {
	// Para este ejemplo crearemos un iterador a partir de un slice
	values := []int{1, 2, 3, 4, 5, 6, 7}
	iterator := slices.All(values)

	// Se pueden iterar con un bucle como un slice normal y corriente
	for i, v := range iterator {
		fmt.Printf("i=%d v=%d\n", i, v)
	}
    // O pasandole directamente la funcion yield
	iterator(func(index int, value int) bool {
		fmt.Printf("i=%d v=%d\n", index, value)
		return true
	})
}
```
En el caso de usar la función yield directamente es lo más parecido a usar un foreach en Go.

### Como crear uno
Ahora que sabemos como se usan los iteradores vamos a ver como funcionan por dentro y vamos a crear uno.

La manera más sencilla de crear un iterador es creando una función que reciba la función yield.
```go
package main

func main() {
    for i, v := range myIterator {
        fmt.Println(i, v)
    }
}

func myIterator(yield func(int, int) bool) {
    for i := 0; i < 10; i ++ {
        if !yield(i, i) {
            return
        }
    }
}
```

Sin embargo, la ventaja que nos dan los iteradores es poder crear iteradores sobre la marcha o con base en unos parámetros.
Por ejemplo, esta es la función Range de mi librería iterago. Un experimento para ver que tan lejos se puede llegar con los iteradores.

```go
// Range te permite crear un iterador con un inicio, un fin y especificando como se va a incrementar o decrementar
// el siguiente valor generado
func Range(start int, end int, jump int) iter.Seq2[int, int] {
    // Aquí devolvemos el iterador y este esta usando los parametros de Range para poder crear el bucle
	return func(yield func(int, int) bool) {
		var value = start
		var index = 0
		for {
			if value == end {
				return
			}
			if !yield(index, value) {
				return
			}
			index += 1
			value += jump
		}
	}
}
```

```go
package main

func main() {
    for i, v := range Range(0, 10, 1) {
        fmt.Println(i, v) // Imprime numeros del 1 al 10
    }
}
```

## El paquete iter
Si te has fijado, Range no usa ninguna de las cabeceras de yield que se han mencionado previamente. Este usa un
tipo llamado Seq2 del paquete [iter](https://pkg.go.dev/iter).

Este paquete se ha introducido recientemente con los iteradores. Contiene tipos y funciones que facilitan el desarrollo
con iteradores. Seq2 y Seq son los tipos que nos provee para crear iteradores. Seq2 es un tipo que nos provee una clave y un valor
mientras que Seq nos provee tan solo un valor. Recomiendo encarecidamente revisar el paquete iter para más información y utilidades.

También se han añadido funciones a los paquetes slices y maps entre otros paquetes nuevos en esta nueva versión.
Recomiendo revisar el post acerca del lanzamiento de [go 1.23](https://tip.golang.org/doc/go1.23#iterators)

## Pull vs push
En algunos casos es más conveniente obtener esos datos bajo ciertas condiciones y controlando el control de flujo al momento
de iterar datos obtenidos de un iterador. Aquí es cuando entran los pull iterators. Son iteradores que en vez de iterarse usando
una función a la que se le mandan los datos a iterar. Te proveen de una función con la que puedes hacer un *pull* del siguiente dato.

En el lanzamiento de go 1.23 nos proveen de un ejemplo.

```go
// Codigo extraido del blog https://go.dev/wiki/RangefuncExperiment#how-is-iterpull-used

// Zipped holds values from an iteration of a Seq returned by [Zip].
type Zipped[T1, T2 any] struct {
    V1  T1
    OK1 bool

    V2  T2
    OK2 bool
}

// Zip returns a new Seq that yields the values of seq1 and seq2 simultaneously.
func Zip[T1, T2 any](seq1 iter.Seq[T1], seq2 iter.Seq[T2]) iter.Seq[Zipped[T1, T2]] {
    return func(yield func(Zipped[T1, T2]) bool) {
        p1, stop := iter.Pull(seq1)
        defer stop()
        p2, stop := iter.Pull(seq2)
        defer stop()

        for {
            var val Zipped[T1, T2]
            val.V1, val.OK1 = p1()
            val.V2, val.OK2 = p2()
            if (!val.OK1 && !val.OK2) || !yield(val) {
                return
            }
        }
    }
}
```

## Usos
La primera ventaja que nos dan los iteradores respecto a un slice es el poder procesar datos de manera perezosa. Esto
permite ser más eficientes en el uso de memoria, ya que solo estaremos asignando la memoria necesaria para procesar él
dato que estemos iterando (o los que queramos guardar en memoria si queremos ganar en velocidad). El punto es que permite
poder procesar datos poco a poco sin tener que cargarlos todos en memoria.

Otra razón es poder crear datos iterables. De la misma manera que podemos obtener datos de manera perezosa, podemos
generarlos de manera perezosa. Por ejemplo, con la función Range.

```go
package main

import (
    "github.com/4strodev/iterago/builders"
    "slices"
    "fmt"
)

func main() {
    // Bajo estas condiciones iterator va a ser un iterador infinito ya que si incrementamos 0 en 1 nunca llegara a
    // ser igual que -1 por lo que este iterador no terminara hasta que nosotros se lo indiquemos
    iterator := builders.Range(0, -1, 1)

    for _, v := range iterator {
        if v > 10 {
            break
        }

        fmt.Println(v)
    }
}
```

## Rendimeinto
En el propio blog de go 1.23 tienen una [sección hablando del tema](https://go.dev/wiki/RangefuncExperiment#can-range-over-a-function-perform-as-well-as-hand-written-loops).
En resumen, cuentan que el compilador es capaz de convertir las funciones en *inline functions* (es decir, que en vez de llamar
a la función se inserta el código de esta directamente en el lugar donde se está llamando). Y tras optimizaciones y más
inlining el compilador de go es capaz de convertir los iteradores en bucles normales de Go.

Por lo que a nivel de rendimiento, teóricamente, están a la altura de bucles escritos a mano.

> ⚠️ Esto solo funciona para iteradores simples por lo que con iteradores mas complejos que usen dentro mas iteradores
la sobrecarga de llamar a funciones constantemente va a ser algo significativo. Así si no quieres tener cuellos de
botella evita hacer composiciones de iteradores demasiado complejas.

## Controversia
La controversia aquí viene porque muchos desarrolladores afirman que esto se aleja de la linea de que Go es un lenguaje
que destaca por su simpleza y esto es innecesario y puede añadir complejidad al código.

## Opinión personal
Si bien es cierto que la comunidad de Go es muy reacia a traer patrones y funcionalidades de otros lenguajes y paradigmas
creo que los iteradores pueden ser un punto de inflexión hacia un lenguaje más maduro y que permita evitar cogido repetitivo.

Los iteradores abren una ventana a abstraer el acceso y procesamiento de datos, permitiendo que en tus bucles solo tengas
la lógica que realmente afecta al negocio.

Al principio la sintaxis es algo confusa. Pero una vez entiendes que es cada parte, no te resulta extraña para nada. Admiro
lo cautelosos que son los desarrolladores de Go y como no se dejan llevar por el tren de hype. Por lo que no creo que esto
haya sido un patinazo por su parte.

Como algo a destacar desde que en Go se introdujeron los genéricos cada vez veo en más partes de la biblioteca
estandard paquetes y módulos que usan esta funcionalidad y los iteradores no son la excepción. A nadie le gustan los cambios
pero como muchos seniors dicen. Que algo exista no implica que tengas que usarlo. Somos libres de escoger como queremos
crear nuestros proyectos y en ningún momento los iteradores van a ser un remplazo de nada. Solo son una herramienta más
que tenemos a nuestra disposición.

Otra cosa que creo que es importante es que el tener un sistema estandarizado de iteradores permitirá que muchas librerías
se basan en este para desarrollar sus SDKs entre estas la propia librería estándar. Hay muchas partes del código que
usan su propio sistema de iteradores y muchos de estos no son ni compatibles ni iguales.

Por lo que creo que esto es un paso hacia adelante en código estandarizado y sobre todo menos repetitivo.
