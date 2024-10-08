+++
title = 'Inyección de Dependencias'
date = 2024-10-07T22:54:26+02:00
draft = false
+++

Uno de los patrones mas mencionados y menos entendidos es la Inyección de Dependencias o DI en inglés.
Lo confunden muchas veces con dos conceptos que si bien tienen relación, no son lo mismo. Inversion de Control y autowiring.
En este post vamos a hablar de los tres y para que te hagas a la idea de cual es su relación aquí te dejo en orden
des del mas abstracto a mas concreto:

- [Inversion de Control (IoC)](#inversión-de-control)
- [Inyección de dependencias (DI)](#inyección-de-dependencias)
- [Autowiring](#autowiring)

## Inversión de control
La Inversión de control muy resumidamente consiste en delegar partes de tu software a componentes externos.
Estos componentes pueden ser escritos por ti mismo o tu equipo o pueden ser librerías y frameworks que estáis usando.

### Ejemplo
> :cloud: Imaginemos que estais desarrollando una API REST

Cuando desarrollamos una API REST usamos el protocolo http. Lo normal es usar alguna librería o framework como express
en JS o ASP.NET para C#. En ese caso estamos delegando el comportamiento del enrutamiento, parseo del body, manejamiento
de las consultas, etc. Entonces el control lo tiene el framework y nosotros manejamos la lógica de negocio.

## Inyección de dependencias
La inyección de dependencias es tan sencillo como pasar por parametros instancias de aquellas clases que necesitas
en tu método o clase para poder funcionar, en vez de instanciarlas en tu constructor, método, etc.

### Ejemplo

> :cloud: Estas desarrollando un ecomerce. Una parte fundamental es el registro de los eventos que sucedan en tu
aplicación.

Para ello se suelen usar librerias que te permiten instanciar y configurar loggers que te permiten
configurar de mil maneras cómo y donde tienen que escribir tus logs.

Ejemplos de librerias de logging:
- [Winston](https://www.npmjs.com/package/winston) -> Node
- [Log4J](https://logging.apache.org/log4j/2.x/)-> Java
- [slog](https://pkg.go.dev/log/slog) -> Go
- [Serilog](https://serilog.net) -> .NET

Entonces la problemática que nos surge es que en muchas partes de nuestro código tenemos que escribir logs. La primera
opión que se nos ocurre es instanciar un logger nuevo por cada clase o método que necesite usar nuestro log. Vamos
a ver un ejemplo con Go y slog. (sencillamente porque me apetece).

> :warning: El código tiene un proposito ilustrativo no funcional

```go
package auth

import (
    "log/slog"
    "os"

    "github.com/4strodev/di_example/pkg/users"
)

type AuthService struct {}

type LoginRequest struct {
    Email       string
    Password    string
}

type LoginResponse struct {
    AccessToken string
    RefreshToken string
}

func (s *AuthService) Login(req LoginRequest) (LoginResponse, error) {
    var response LoginResponse
    // Instanciando el logger y el repositorio
    // Ambas instancias son ejemplos de valores que podemos inyectar como dependencias
    var logger = slog.New(slog.NewTextHandler(os.Stdout, nil))
    var userRepository = users.NewUserRepository()

    // Obteniendo el usuario de la base de datos
    user, err := userRepository.FindByEmail(req.Email)
    if err != nil {
        return response, err
    }

    // Validando las credenciales
    match := user.MatchPassword(req.Password)
    if !match {
        return response, errors.New("password does not match")
    }

    // Otra opción es emitir un evento a un event bus y tener un listener
    // que escuche ese evento y haga el log correspondiente. Pero vamos a
    // tratar de simplificar el ejemplo
    logger.Info("user logged in", "id", user.Id.String())

    // ... Creando los tokens and devolviendo la respuesta
}
```

Ahora imagina que tienes otro servicio que se encarga de añadir un producto en el catálogo. Podriamos tener algo como
esto.

```go
package products

import (
    "log/slog"
    "os"
    "github.com/google/uuid"
)

type ProductCatalogService struct {}

type AddProductRequest struct {
    ProductName     string
    Description     string
    Price           int
}

func (s *ProductCatalogService) AddProduct(req AddProductRequest) error {
    // Instanciando el logger y el repositorio
    // Ambas instancias son ejemplos de valores que podemos inyectar como dependencias
    var logger = slog.New(slog.NewTextHandler(os.Stdout, nil))
    var catalogRepository = NewProductCatalogRepository()

    // Obteniendo el usuario de la base de datos
    product := Product{
        Id: uuid.Must(uuid.NewV7())
        Price: req.Price,
        Name: req.ProductName,
        Description: req.Description,
    }
    err := catalogRepository.Save(product)
    if err != nil {
        return err
    }

    // Otra opción es emitir un evento a un event bus y tener un listener
    // que escuche ese evento y haga el log correspondiente. Pero vamos a
    // tratar de simplificar el ejemplo
    logger.Info("new product added to the catalog", "id", product.Id.String())

    // ... Creando los tokens and devolviendo la respuesta
}
```

Esto puede multiplicarse por cada una de los servicios que tengamos en nuestra aplicación. Al principio funciona ya que
todos los servicios usan el mismo sistema de logging con el mismo formato. El problema es que es dificil cambiar el
logger de nuestros servicios. Por ejemplo en un momento dado podria interesarnos que todos los loggers tengan un
atributo que sea siempre el nombre del servicio o que todos los atributos que se añadan a los logs esten en un grupo
llamado `attributes`.

El problema viene cuando tenemos que hacer todas estas modificaciones en nuestros servicios. Podria ser
que instanciar nuestro logger en nuestro caso concreto no sea tan sencillo como llamar a una funcion con algunos
parámetros por defecto. Podria ser algo mas parecido a esto.

```go
package shared

import (
	"io"
	"log/slog"
	"os"
)

func NewLogger() (*slog.Logger, error) {
	err := os.MkdirAll("/tmp/log/my_application", os.ModePerm|os.ModeDir)
	if err != nil {
		return nil, err
	}
	logFile, err := os.OpenFile("/tmp/log/my_application/logs.txt", os.O_WRONLY|os.O_TRUNC|os.O_CREATE, os.ModePerm)
	if err != nil {
		return nil, err
	}
	attributes := []slog.Attr{
		slog.Any("service_name", "my_application"),
	}

	output := io.MultiWriter(logFile, os.Stdout)

	handler := slog.NewJSONHandler(output, &slog.HandlerOptions{
		Level: slog.LevelDebug,
	}).WithAttrs(attributes)
	return slog.New(handler), nil
}
```

La solución que se nos puede venir a la mente es extraer esa lógica en un método o función
aparte o implementar patrones mas complejos como factory. Para este ejemplo vamos a dejarlo en la opción mas simple.

Ahora nuestros servicios ya no tienen que instanciar manualmente el logger y cualquier cambio va a tener una única fuente
de la verdad. Sin embargo seguimos teniendo un problema, que es la reemplazabilidad. De esto ya hablé en un [artículo
sobre POO](https://4strodev.com/blog/no_sabes_poo_1). Nuestros servicios no tienen la capacidad de usar diferentes
loggers. Si queremos hacer que nuestro servicio pueda ser versatil en diferentes contextos, ya sea en un entorno de
testing o en producción, etc. Tendriamos que modificar la funcion `NewLogger` para hacer que todos nuestros servicios
puedan usar un logger adaptado a las circunstancias.

Pero esto implica hacer que nuestra función de instanciación sea mas compleja y que ella misma tenga que comprobar bajo
que entorno se está ejecutando para saber que logger instanciar. Se puede conseguir con patrones como factory pero
todos fallan en un punto importante. Que no todos los servicios necesariamente usan el mismo logger o es el propio servicio
quién tiene el control de cómo se instancia una de sus dependencias.

Aquí es donde entra IoC y DI. Para hacer nuestro servicio mas versátil podemos delegar la instanciación del logger al
mismo que instancia el servicio. Esto permite liberar al servicio de la responsabilidad de instanciar un logger y permite
que el mismo servicio pueda ser configurado de diferentes maneras.

```go
package auth

import (
    "log/slog"
    "os"

    "github.com/4strodev/di_example/pkg/users"
)

type AuthService struct {
    Logger *slog.Logger
}

func NewAuthService(logger *slog.Logger) *AuthService {
    return &AuthService{
        Logger: logger,
    }
}

type LoginRequest struct {
    Email       string
    Password    string
}

type LoginResponse struct {
    AccessToken string
    RefreshToken string
}

func (s *AuthService) Login(req LoginRequest) (LoginResponse, error) {
    var response LoginResponse
    // Instanciando el logger y el repositorio
    // Ambas instancias son ejemplos de valores que podemos inyectar como dependencias
    var userRepository = users.NewUserRepository()

    // Obteniendo el usuario de la base de datos
    user, err := userRepository.FindByEmail(req.Email)
    if err != nil {
        return response, err
    }

    // Validando las credenciales
    match := user.MatchPassword(req.Password)
    if !match {
        return response, errors.New("password does not match")
    }

    // Otra opción es emitir un evento a un event bus y tener un listener
    // que escuche ese evento y haga el log correspondiente. Pero vamos a
    // tratar de simplificar el ejemplo
    s.logger.Info("user logged in", "id", user.Id.String())

    // ... Creando los tokens and devolviendo la respuesta
}
```

Ahora nuestro servicio tiene como atributo un logger. Cada vez que queramos usar este servicio podremos configurar
como queremos que se escriban los logs en nuestro servicio.

```go
package main

import (
    "github.com/4strodev/di_example/pkg/auth"
)

func main() {
    // ... Codigo para poner en marcha nuestra aplicación
    logger := shared.NewLogger()
    s := auth.NewAuthService(logger)
    // ... 
}
```

Pues esto es la inyección de dependencias. Ahora nuestro servicio tiene unas dependencias que se las mandamos por
parametro al momento de instanciarlo. También podriamos aplicar algo parecido a funciones y métodos. En vez de instanciar
las dependencias en el propio método, puedes mandarlas por parámetro.

Esto permite que en cualquier momento podamos crear loggers diferentes con configuraciones diferentes y solo tenemos
que cambiar el logger que le mandamos a nuestro servicio. Incluso con esto podemos crear una sola instancia y reusarla
para diferentes servicios.

```go
package main

import (
    "github.com/4strodev/di_example/pkg/auth"
    "github.com/4strodev/di_example/pkg/product"
)

func main() {
    // ... Codigo para poner en marcha nuestra aplicación
    logger := shared.JSONLogger()
    // Estamos usando la misma instancia para ambos servicios lo que nos permite ahorrar
    // recursos
    authService := auth.NewAuthService(logger)
    productService := product.NewProductService(logger)
    // ... 
}
```

## Autowiring
El autowiring es una manera muy concreta de implementar la inyección de dependencias. En el ejemplo anterior vimos
como estas dependencias las instanciabamos y mandabamos manualmente nosotros como desarrollador. El autowiring consiste
en que nuestras dependencias se resuelvan automaticamente. Para esto es necesario usar librerias que nos permitan
configurar y resolver nuestras dependencias. Muchas de estas vienen en los frameworks que usamos a diario como:

- Spring: Java
- ASP.NET: C#
- NestJS: TypeScript

En este caso voy a mencionar una libreria que he creado para Go. Se llama [wiring](https://github.com/4strodev/wiring).
Hay muchas aproximaciones cuando se trata de autowiring:

- Inyección por constructor
- Inyección por contenedor
- Inyección por setters
- Inyección por inspección de atributos
- Inyección por implementación de interfaces
- Inyección por tokens

Muchas veces terminas usando varias estrategias a la vez. Así que vamos directos con un ejemplo. 

```go
package main

import (
    "fmt"
    wiring "github.com/4strodev/wiring/pkg"
)

type Abstraction interface {
    Greet()
}

type Implementation struct {
}

func (i *Implementation) Greet() {
    fmt.Println("Hello world")
}

func main() {
    var err error
	var container = wiring.New()
    // Este resolver solo se ejecuta una vez ya que es un singleton.
    // El contenedor almacenará en cache la instancia de esta abstración
    // la próxima vez que se use, tan solo volvera a usar esa instancia
	container.Singleton(func() (Abstraction, error) {
		fmt.Println("Running resolver")
		return &Implementation{}, nil
	})

    // Resolviendo una dependencia
	var impl Abstraction
	err = container.Resolve(&impl)
    if err != nil {
        panic(err)
    }
	impl.Greet()

    // Esta vez no se volverá a ejecutar el resolver y tan solo
    // volverá a usar la instancia que se hizo anteriormente
	var otherImpl Abstraction
	err = container.Resolve(&otherImpl)
    if err != nil {
        panic(err)
    }
	otherImpl.Greet()
	// Output:
	// Running resolver
	// Hello world
	// Hello world
}
```

Este ejemplo lo he sacado de un proyecto que he estado haciendo para practicar la visualización de datos con Grafana.
Esto es un controlador que tiene como dependencia el servicio que contiene la lógica de negocio. Entonces usando el
método Fill del contenedor resuelvo todas las dependencias de mi controlador.

```go
package auth

import (
	"github.com/4strodev/wiring/pkg"
	"github.com/gofiber/fiber/v3"
)

type AuthController struct {
	Service AuthService
	Router  fiber.Router
}

// Init implements components.Component.
func (a *AuthController) Init(container pkg.Container) error {
    // Aquí estoy resolviendo todas las dependencias de mi controlador
	err := container.Fill(a)
	if err != nil {
		return err
	}

	// Setting routes
	group := a.Router.Group("/auth")
	group.Post("/login", a.Login)
	group.Post("/register", a.Register)

	return nil
}

func (c *AuthController) Register(ctx fiber.Ctx) error {
	var requestBody RegisterRequest
	err := ctx.Bind().Body(&requestBody)
	if err != nil {
		return err
	}

	err = c.Service.Register(requestBody)
	if err != nil {
		return err
	}

	return ctx.JSON(fiber.Map{
		"msg": "user registered",
	})
}

func (c *AuthController) Login(ctx fiber.Ctx) error {
	requestBody := LoginRequest{}
	err := ctx.Bind().Body(&requestBody)
	if err != nil {
		return err
	}
	response, err := c.Service.Login(requestBody)
	if err != nil {
		return err
	}
	return ctx.JSON(fiber.Map{
		"data": response,
		"msg":  "everything okay",
	})
}
```

Recomiendo revisar el repositorio de
