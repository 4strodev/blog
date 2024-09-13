+++
title = 'La POO no es lo que te han contado'
date = 2024-08-23T22:53:34+02:00
draft = false
+++

Si estás en la carrera de ingeniería informática y has logrado sobrevivir al primer año de carrera, es posible que te
hayas tenido que enfrentar o te estás enfrentando a la POO. Si al igual que yo no entendías por qué necesitábamos interfaces
clases abstractas, herencia, polimorfismo... Léete este artículo porque hoy vas a ver con ejemplos prácticos y enfocados
al mundo laboral porque la POO necesita todos estos conceptos para hacer software que sea mantenible y pueda crecer
con el tiempo.

> :warning: Toma el código como lo que és, ejemplos concisos que se centran en una parte de todo el pastel. No
recomiendo bajo ningún concepto coger el código que hay como ejemplo y llevarlo a producción sin hacer las adaptaciones
pertinentes.

## Necesidades de un proyecto
Un proyecto de software independientemente del tamaño de este, normalmente, pasa por diferentes fases durante su desarrollo.

> Aquí no voy a ser riguroso con ninguna metodologia de desarrollo, esto es mas bien a nivel conceptual

- **Desarrollo**: En esta fase se analizan los requisitos de la aplicación a nivel técnico y funcional. Se realizan análisis
y diseños sobre como será la aplicación a nivel arquitectónico, qué componentes de infraestructura necesita y que 
tecnologías se van a usar. En esta fase se desarrollan e implementan nuevas funcionalidades, se testean, pasan un control de calidad y si
cumplen con los requisitos que se han marcado en un inicio o que los usuarios esperan o necesitan, se lanza la aplicación
para que pueda estar a la disposición de los usuarios. Dependiendo de la metodología usada esto se puede volver
un proceso iterativo y que se divide en varias fases o varios objetivos antes de lanzar la aplicación al mercado.

- **Lanzamiento**: En esta fase contamos con que tenemos MVP (Minimal Viable Product). En este paso se preparan todos los componentes
que necesita la aplicación para lanzarse a producción. La base de datos, la infraestructura de red, sistemas de monitoreo, etc.
Aquí ya depende de cada empresa y proyecto si se van a usar servicios en la nube o una infraestructura propia. Durante
el proceso de lanzamiento se realizan lo que se conoce como despliegues. Un despliegue es cuando preparas un entorno
real (se le conoce como producción, el que está a disposición de los usuarios y debe ser resiliente) o similar (por
ejemplo para hacer pruebas internas antes de lanzarlo a producción) y ejecutas tu aplicación en ese entorno. Puede ser
un servidor propio, un servicio en la nube, etc. Durante el ciclo de vida de una aplicación los despliegues son algo
constante y periódico, por lo que se suelen automatizar para agilizar el proceso de desarrollo. Durante un despliegue
puedes someter a tu aplicación bajo una serie de pruebas y comprobaciones antes de lanzarlo a producción para detectar
fallos antes de que la aplicación esté en producción y así evitar la mayor cantidad de problemas en producción.
Finalmente, se prepara la aplicación para lanzarse a producción, se compila, empaqueta, prepara un contenedor de docker...

- **Mantenimiento**: Una vez la aplicación está en el mercado o la usan tus usuarios finales. Es muy probable que aparezcan
bugs, peticiones de nuevas funcionalidades o que en caso de ser un producto exitoso cada vez tenga más usuarios activos
y haya que realizar cambios para que la aplicación no muera de éxito. Esto es el mantenimiento, asegurarse de que tú
producto cumple con las necesidades de los usuarios. Haciendo mejoras y solucionando errores constantemente. Por eso
los despliegues son algo constante en tu aplicación porque no lanzas a producción una sola vez. Si no que a medida
que actualizas tu aplicación tienes que subir esos cambios a producción.


Durante todo este proceso de desarrollo hay varios puntos clave que afectan a tu día a día como desarrollador e incluso
a que tan fácil es trabajar con este proyecto.

## Dependencias
Dependiendo del contexto puede tener significados ligeramente distintos. Pero básicamente se resume en que tengo un, módulo
A que depende de otro módulo B para poder funcionar correctamente. Un módulo dependiendo del contexto puede ser una cosa
u otra:

- Desde una clase que usa otra clase para realizar su trabajo.
- Una parte de tu aplicación que cumple con un propósito concreto dentro de todo lo que tiene que hacer tu aplicación.
- O incluso una aplicación entera que depende de una API como Stripe para poder realizar sus pagos electrónicos. O
las librerías que necesita tu aplicación para poder funcionar.

Independientemente de cuál sea el contexto, la POO siempre tiene un objetivo claro, reducir las dependencias. Los motivos
principales son hacer que nuestra aplicación sea fácil de testear, fácil de extender y por ende mantenible.

A este concepto de reducir las dependencias se le conoce como ✨Desacoplamiento✨.

## Testeabilidad
Para entender por qué reducir las dependencias puede ayudarnos a hacer que nuestra aplicación sea fácil de testear,
imaginemos el siguiente caso de uso. Tenemos una aplicación web que tiene usuarios. Estos usuarios tienen que poder
registrarse. Pongamos que estamos usando TypeScript y NestJS para hacer nuestra aplicación. Este sería nuestro
controlador.

```typescript
// UserController.ts
import { Controller, Post, Body, ConflictException } from '@nestjs/common';
import { v7 as uuidv7 } from 'uuid';
import mysql from 'mysql2/promise';
import { User } from './user';
import { UserResDTO } from './UserResDTO';

export class SaveUserDTO {
    constructor(
        public email: string,
        public name: string,
        public password: string
    ){}
}

@Controller('users')
export class UsersController {
    constructor(@Inject('MYSQL') private pool: mysql.Pool) { }

    @Post()
    async register(@Body() body: SaveUserDTO) {
        const id = uuidv7(); // uuid es un estandard para crear ids unicos
        const user = await User.create(id, body.email, body.password, body.name);

        await this.pool.execute(
            'INSERT INTO users (id, email, password, name) VALUES (?, ?, ?, ?)',
            [user.id, user.email, user.password, user.name]
        );

        return {
            msg: "User created successfully",
            user: new UserResDTO(user),
        }
    }
}
```

```typescript
// User.ts
import * as bcrypt from 'bcrypt';

export class User {
    constructor(
        public readonly id: string,
        public readonly email: string,
        public readonly password: string,
        public readonly name: string,
    ) { }

    public static async create(
        id: string,
        email: string,
        password: string,
        name: string,
    ): Promise<User> {
        const encryptedPassword = await bcrypt.hash(password, 14);
        return new User(id, email, encryptedPassword, name);
    }
}
```

```typescript
// UserResDTO.ts
import { User } from "./user";

export class UserResDTO {
    id: string;
    name: string;
    email: string;

    constructor(user: User) {
        this.id = user.id;
        this.email = user.email;
        this.name = user.name;
    }
}
```

Imaginemos que queremos hacer un test unitario para este controlador, eso va a ser algo complicado. Este está usando directamente
el pool de MySQL. Por lo que en nuestra lógica de negocio queda plasmado que estamos interactuando con mysql. En este caso
para poder realizar el test tendríamos que mandarle una instancia de MySQL. Pool que en vez de interactuar con nuestra base
de datos en producción se conecta a una preparada para hacer pruebas. Además, en este caso concreto es 'fácil' preparar un
test así ya que el controlador importa el pool a través de su constructor, porque si fuera un Singleton apaga y vámonos.
Además, esto hay que prepararlo para que se pueda ejecutar en nuestro pipeline de CI/CD (los despliegues automatizados).

Si da la puñetera casualidad que nuestro pipeline se ejecuta en un entorno en el que no hay conexión a internet o 
estamos capados y no podemos conectarnos a una base de datos. El objetivo es simple, vamos a cambiar nuestro controlador
para que no dependa directamente de MySQL. Crearemos una abstracción de nuestro sistema de persistencia que permita guardar
y obtener usuarios de la base de datos usando una interfaz.

Para ello usaremos el patrón DAO. Este consiste en crear una interfaz que define los métodos para mutar y obtener los
datos de un objeto de la base de datos y luego usar una implementación concreta que tiene la lógica para el motor de
base de datos concreto.

Con esto podemos tener al menos dos implementaciones, una que interactúa con MySQL y otra que guarda los datos en memoria
al guardar los datos en memoria estos no persisten. Lo que nos permite evitar los efectos colaterales de guardar datos
en una base de datos. Perfecto para un test.

Primero vamos a crear la interfaz `UserDAO`.

```typescript
// UserDAO.ts
import { User } from "./user";

// Debido a que typescript es un lenguaje que tiene que ser compatible con JS
// cuando creas una interfaz esta solo se contempla en tiempo de compilación
// ya que JS no tiene interfaces solo clases. Debido a que NestJS trabaja con
// la reflexión para poder inyectar las dependencias a todos los controladores
// y servicios, tendremos que usar una clase abstracta y luego implementar esta
// clase abstracta. Sin embargo en lenguajes como Java o C# podemos usar
// interfaces sin ningun tipo de problema.
export abstract class UserDAO {
    public abstract save(user: User): Promise<void>;
    public abstract update(user: User): Promise<void>;
    public abstract findAll(): Promise<User[]>;
    public abstract find(id: string): Promise<User | undefined>;
    public abstract delete(id: string): Promise<void>;
}
```

Una vez tenemos la interfaz vamos a crear una implementación de UserDAO para que todos aquellos servicios
y controladores que usen `UserDAO` puedan conectarse a MySQL.
```typescript
// MysqlUserDAO.ts
import { Injectable } from "@nestjs/common";
import { User } from "./user";
import { UserDAO } from "./UserDAO";
import mysql from 'mysql2/promise';

@Injectable()
export class MysqlUserDAO implements UserDAO {
    constructor(private pool: mysql.Pool) { }

    public async save(user: User): Promise<void> {
        await this.pool.execute(
            'INSERT INTO users (id, email, password, name) VALUES (?, ?, ?, ?)',
            [user.id, user.email, user.password, user.name]
        );
    }

    public async update(user: User): Promise<void> {
        await this.pool.execute(
            'UPDATE users SET id = ?, email = ?, password = ?, name = ?',
            [user.id, user.email, user.password, user.name]
        );
    }

    public async findAll(): Promise<User[]> {
        const [result] = await this.pool.query<mysql.RowDataPacket[]>('SELECT * FROM users');
        const users: User[] = [];

        for (const r of result) {
            const user = new User(r.id, r.email, r.password, r.name);
            users.push(user);
        }

        return users;
    }

    public async find(searchId: string): Promise<User | undefined> {
        const [result] = await this.pool.query<mysql.RowDataPacket[]>('SELECT * FROM users WHERE id = ?', [searchId]);

        if (result.length === 0) {
            return
        }

        const { id, email, password, name } = result[0];
        const user = new User(id, email, password, name);
        return user;
    }
    public async delete(id: string): Promise<void> {
        await this.pool.execute('DELETE FROM users WHERE id = ?', [id]);
    }

}
```

Ahora vamos a hacer otra implementación que guarde esos datos en memoria. Esta característica de múltiples
implementaciones. Permite modificar el comportamiento de nuestros servicios especificándoles donde queremos
que persistan esos datos. La lógica de negocio sigue igual, pero permitimos que nuestros servicios sé
abstraigan de qué sistema de persistencia estamos usando por debajo haciendo que podamos reutilizarlos
para múltiples sistemas de persistencia. Ya sea en memoria, archivos, bases de datos, etc.
```typescript
// InMemoryUserDAO
import { Injectable } from "@nestjs/common";
import { User } from "./user";
import { UserDAO } from "./UserDAO";

@Injectable()
export class InMemoryUserDAO implements UserDAO {
    private _data: Map<string, User> = new Map();

    public async save(user: User): Promise<void> {
        this._data.set(user.id, user);
    }
    public async update(user: User): Promise<void> {
        this._data.set(user.id, user);
    }
    public async findAll(): Promise<User[]> {
        return Array.from(this._data.values());
    }
    public async find(id: string): Promise<User | undefined> {
        return this._data.get(id);
    }
    public async delete(id: string): Promise<void> {
        this._data.delete(id);
    }
}
```

Ahora UserController en vez de usar el Pool de MySQL va a usar nuestra interfaz. Esto nos permite ejecutar
nuestro controlador en diferentes contextos dependiendo de lo que queramos obtener. Para nuestra aplicación
en producción NestJS al inyectar UserDAO tendrá que recurrir a una implementación de `UserDAO`. Ya que no puedes
instanciar una clase abstracta. Así que configuraremos NestJS para que cada vez que algún servicio necesite
una instancia de `UserDAO` le inyecte una instancia de `MysqlUserDAO`.
```typescript
// UserController.ts
import { Controller, Post, Body } from '@nestjs/common';
import { v7 as uuidv7 } from 'uuid';
import { User } from './user';
import { UserResDTO } from './UserResDTO';
import { UserDAO } from './UserDAO';

export class SaveUserDTO {
    constructor(
        public email: string,
        public name: string,
        public password: string
    ){}
}

@Controller('users')
export class UsersController {
    constructor(private userDao: UserDAO) { }

    // El método register tiene la responsabilidad de obtener los datos de nuestra consulta HTTP
    // crear un nuevo usuario y validar que este sea correcto.
    @Post()
    async register(@Body() body: SaveUserDTO) {
        const id = uuidv7(); // uuid es un estandard para crear ids unicos
        const user = await User.create(id, body.email, body.password, body.name);

        await this.userDao.save(user);

        return {
            msg: "User created successfully",
            user: new UserResDTO(user),
        }
    }
}
```

Prepararemos el test de `UserController`. NestJS funciona por módulos. Creas un módulo de tu aplicación
en el que puedes asignar controladores, proveedores, e importar otros módulos que a su vez tienen proveedores, controladores
y pueden exportar algunos proveedores. En este caso estamos creando un módulo de testing. En este módulo le asignamos un
controlador, `UserController` que es el que queremos testear. La clave de todo esto, está en la sección de providers.
Aquí le estamos especificando que cuando tengamos que inyectar un `UserDAO` NestJS debe inyectar una instancia
de `InMemoryUserDAO`.
```typescript
// UserControllerTest.ts
import { Test, TestingModule } from '@nestjs/testing';
import { SaveUserDTO, UsersController } from './users.controller';
import { UserDAO } from './UserDAO';
import { InMemoryUserDAO } from './InMemoryUserDAO';

describe('UsersController', () => {
    let controller: UsersController;
    let dao: UserDAO;

    beforeEach(async () => {
        const module: TestingModule = await Test.createTestingModule({
            controllers: [UsersController],
            providers: [
                {
                    provide: UserDAO,
                    useClass: InMemoryUserDAO,
                }
            ]
        }).compile();

        controller = module.get<UsersController>(UsersController);
        dao = module.get<UserDAO>(UserDAO);
    });

    // Un test sin transacciones y sin preparar una infraestructura compleja para evitar
    // efectos colaterales
    it('should save user on database with encrypted password', async () => {
        const dto = new SaveUserDTO("john@example.com", "John Doe", "super secure password");
        const response = await controller.register(dto);
        const userId = response.user.id;

        const user = await dao.find(userId);
        expect(user).toBeDefined();

        expect(user.id).toEqual(userId);
        expect(user.email).toEqual(dto.email);
        expect(user.name).toEqual(dto.name);
        expect(user.password).not.toEqual(dto.password);
    });
});
```

## Conclusión

> Los puristas de las clean architectures i las arquitecturas por capas os callais. Ya hablaré de eso en otro momento.
Pero este no es post indicado para ello.

Una de las ventajas de las abstracciones es poder remplazar componentes de nuestros controladores, servicios, etc.
Para modificar su comportamiento. Lo que en el caso del testing nos facilita mucho el trabajo.
También puede ser beneficioso en la mantenibilidad de nuestro proyecto. Sobre todo con las migraciones.

Continuemos con el ejemplo de la persistencia de datos. En node la librería más popular para usar MySQL era el paquete
mysql. Sin embargo, este quedó obsoleto y salió otra versión llamada mysql2 que soportaba la nueva api de async await.

Cuando tienes un proyecto en el que todos tus controladores y servicios usan directamente la librería de mysql. Hacer él
cambio es una tarea titánica. Por lo que como mínimo, aunque no uses interfaces o clases abstractas, la mejor estrategia
es separar la lógica del acceso a los datos en su propia clase. En ese caso por lo menos solo tienes que hacer el cambio
en un solo lugar.

Eso ya es un punto a nuestro favor; sin embargo, la reemplazabilidad puede ser necesaria. Hay casos en los
que o tu aplicación tiene que poder ser compatible con diferentes bases de datos o que por motivos de escalabilidad
te encuentras en la situación de que una parte de tu sistema va a tener que usar otra base de datos por sus características
técnicas. Por lo que hay que reemplazar ese módulo por uno nuevo. En esos casos tus controladores y servicios no se verán
afectados ya que la lógica es la misma y lo único que cambia es el sistema al que se conectan.

Lo mismo aplica para otros sistemas:

- **Event bus**: En el caso de que tengas un sistema de eventos en tu aplicación te puedes encontrar en el que (si el negocio
va bien y lo requiere) tengas que conectar servicios externos a tu aplicación. Te puedes encontrar con la situación de que
haya que empezar a usar message brokers para poder conectar sistemas a través de eventos.

- **Sistemas de notificaciones**: Empiezas usando tu propio servidor de correo electrónico para enviar emails de notificaciones
y campañas de marketing. Pero más adelante decidís cambiar a SendGrid por sus plantillas y su facilidad de uso. Otro
módulo que puede ser reemplazable.

- **Logging**: Es normal que tus aplicaciones escriban logs en algún sitio para poder monitorizar cualquier anomalía o
controlar el rendimiento de las mismas. A veces basta con usar la propia salida estándar, otras veces se envían a sistemas externos.

Por norma general, los módulos más fácilmente reemplazables son aquellos que se encargan de la entrada y salida de datos.
Por lo que si la testeabilidad y mantenibilidad de tus servicios son algo que a tu equipo le concierne. Vale la pena
empezar a ver qué módulos tienes que abstraer para desacoplar la lógica de tu aplicación de tus sistemas externos.

Posiblemente, haya más cosas que añadir a este post así que no te sorprenda si en un futuro ves un post que desarrolla más
este tema.
