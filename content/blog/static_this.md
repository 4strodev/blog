+++
title = '🤯 ¡No sabia que TypeScript podia hacer esto!'
date = 2024-07-20T22:28:27+02:00
draft = false
tags = ['avanzado', 'genéricos']
+++

## Prerrequisitos
Asumo que conoces como funcionan los constructores en JavaScript y los Genéricos en TypeScript. Además de estar
familiarizado con conceptos como la herencia y el polimorfismo.

## El problema
No es que no supiera que TypeScript no pudiera hacer esto, lo que no sabía es como hacerlo.

No te quiero entretener mucho, pero para que lo entiendas tengo que darte un poco de contexto, solo lee y déjate llevar.

¿Alguna vez te has encontrado con una clase que te permite instanciar objetos usando métodos estáticos? Te pongo
un ejemplo de código que tengo en producción. He quitado la documentación para hacerlo más legible
(con la documentación son 80 líneas).

```typescript
export class UUIDValueObject extends NonEmptyStringValueObject {
  constructor(value: string) {
    super(value);

    if (!this.isValidUUID(value)) {
      throw new InvalidArgumentError('Invalid uuid it should be a uuid v7');
    }
  }

  public static random(): UUIDValueObject {
    return new UUIDValueObject(uuidv7());
  }

  protected isValidUUID(value: string): boolean {
    return uuidValidate(value) && uuidVersion(value) === 7;
  }
}
```

Esta clase es un value object que envuelve un UUID v7. Si te fijas tiene un método estático llamado `random`.
Este devuelve un uuid aleatorio, nada nuevo hasta el momento.

Esta clase es una clase base de la que heredan otras clases como `UserId`. El problema con el que me encontré es que si no
sobreescribía el método `random` lo que me devolvía era una instancia de la clase padre, en este caso `UUIDValueObject`, en
vez de la clase que contenía el método.

Es decir, por cada clase que heredara de `UUIDValueObject` (que no son pocas) tenía que sobreescribir un método
estático que se comportaba exactamente igual, sin que cambiara en absoluto el comportamiento del método.

## La solución
De alguna manera tenía que decirle a TypeScript que el método random devuelve una instancia de la clase que lo ejecuta,
en vez de decirle que devuelve una instancia de UUIDValueObject. Aquí la solución sencilla pero altamente efectiva.

```typescript
export class UUIDValueObject extends NonEmptyStringValueObject {
  constructor(value: string) {
    super(value);

    if (!this.isValidUUID(value)) {
      throw new InvalidArgumentError('Invalid uuid it should be a uuid v7');
    }
  }

  public static random<T extends UUIDValueObject>(
    this: new (value: string) => T,
  ): T {
    return new this(uuidv7());
  }

  protected isValidUUID(value: string): boolean {
    return uuidValidate(value) && uuidVersion(value) === 7;
  }
}
```

Como dijo Jack el destripador, vamos por partes. Lo primero que vamos a ver es que devuelve este método.
Este método devuelve un valor genérico de tipo `T`. Este tipo tiene una restricción i es que tiene que ser
un `UUIDValueObject` o derivado.

### El método
```typescript
// Esto es lo que nos intersa por el momento
public static random<T extends UUIDValueObject>(
//
this: new (value: string) => T,
): T {
return new this(uuidv7());
}
```

Entonces en el caso de tener una clase como `UserId`
```typescript
export class UserId extends UUIDValueObject {}
```
El método random sería capaz de devolver `UserId`, ya que este cumple con la restricción de `T`

> 🤔 ¿Pero cómo puede ser que `UserId` al heredar de `UUIDValueObject` `random` nos devuelva una instancia de `UserId`?

Esto se debe a que estamos usando el constructor implícito `this`. En un contexto no estático, en JavaScript, `this`
hace referencia la instancia actual de nuestro objeto.

> 🤓 En realidad es un poco más complejo debido a que JavaScript por debajo no usa una programación orientiada a objetos
basada en clases si no que funciona por prototipos. Sin embargo, para simplificar la lección vamos a acotar el contexto.

Sin embargo, en un contexto estático, `this` hace referencia al constructor de la clase. Ojo esto en tiempo de ejecución.
Por lo que al momento de ejecutarlo random desde `UserId`. El objeto que nos devuelva el constructor `this`,
será una instancia de `UserId`.


### El valor de retorno
Pero como estamos usando TypeScript, hay que indicarle al compilador cuál es la cabecera de `this`, es decir,
que valores acepta y cuáles devuelve. Con esto TypeScript será capaz de entender qué devuelve `random`.

```typescript
public static random<T extends UUIDValueObject>(
// Estableciendo el tipo de this. Fijate que al poner la palabra new estamos indicando que es un constructor
this: new (value: string) => T,
//
): T {
return new this(uuidv7());
}
```

### Nos falta un último detalle
> ¿Porque cuando usamos random no hay que enviarle ningun parametro?

Esto se debe a que `this` es una palabra que tiene un valor implícito. En este caso entiendo que al ya tener un valor por defecto,
es equivalente a cuando tenemos valores opcionales en nuestros métodos, no es necesario darles ningún valor porque ya lo tienen.

## Lecturas adicionales
Te recomiendo encarecidamente que leas más sobre como funciona la POO en JS. Por lo que aquí te dejo cuatro artículos
que te pueden ayudar.

- [Prototipos MDN](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes)
- [Prototipos W3School](https://www.w3schools.com/js/js_object_prototypes.asp)
- [Constructores](https://www.w3schools.com/js/js_object_constructors.asp)
- [This](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Operators/this)
