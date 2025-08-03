# nestjs-react-native

## ¿Cómo gestionas la comunicación entre el backend (NestJS) y el frontend (React Native)?

Utilizo HTTP requests a través de Axios o fetch en React Native para consumir las APIs RESTful expuestas por el backend de NestJS. Cada endpoint está claramente versionado y documentado (por ejemplo, usando Swagger en el backend).

* En React native

  ```js
    import axios from 'axios';

    const API_URL = 'https://api.myapp.com/v1';

    export const loginUser = async (email, password) => {
    return await axios.post(`${API_URL}/auth/login`, { email, password });
    };


  ```

* En Nestjs
 
  ```js
      @Post('login')
      async login(@Body() loginDto: LoginDto) {
          return this.authService.login(loginDto);
      }

  ```
---
## ¿Qué estrategias usas para manejar errores y validaciones entre cliente y servidor?

* En NestJS, uso DTOs con class-validator para validar entradas.

* Capturo errores con filtros globales (@Catch(HttpException)) y manejo centralizado.

* En React Native, muestro mensajes de error amigables para el usuario si hay errores de validación o conexión.


Ejemplo de validación en NestJS:

```js
  import { IsEmail, IsString } from 'class-validator';

  export class LoginDto {
    @IsEmail()
    email: string;

    @IsString()
    password: string;
  }

```

En React Native

```js
  try {
    await loginUser(email, password);
  } catch (error) {
    Alert.alert("Login Error", error.response?.data?.message || "Server error");
  }
```


## ¿Cómo aseguras la autenticación y autorización de los usuarios en una app móvil?

* Uso JWT (JSON Web Tokens) en el backend con Passport.js en NestJS.

* El token se guarda en el Secure Storage en React Native (por seguridad).

* Para rutas protegidas, uso Guards en NestJS (@UseGuards(JwtAuthGuard)).

Ejemplo de flujo:

* El usuario inicia sesión.

* El backend genera un JWT.

* El frontend guarda el token con expo-secure-store o react-native-keychain.

* El token se envía en los headers de futuras peticiones.


En Nestjs

```js

  @UseGuards(JwtAuthGuard)
  @Get('profile')
  getProfile(@Request() req) {
    return req.user;
  }

```

En React Native

```js
  axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
```
---

## ¿Cómo implementas una arquitectura escalable usando NestJS y React Native?

* NestJS: divido en módulos (modular architecture), uso servicios desacoplados, colas con Redis (si es necesario), y organizo el proyecto por dominios.

* React Native: uso navegación estructurada con React Navigation, contextos o Redux para el manejo de estado global, y componentes reutilizables.

Buenas prácticas de escalabilidad:

* Separar responsabilidades (por ejemplo: AuthModule, UserModule, NotificationModule).

* Usar colas con BullMQ para procesos pesados (como envío de correos).

* Dividir la app móvil en pantallas y componentes por dominio: screens/Auth, screens/Profile, components/forms, etc.


---

## ¿Cómo manejas variables de entorno y configuraciones sensibles en ambos entornos?

* En NestJS, uso el paquete @nestjs/config junto con .env files.

* En React Native, uso react-native-dotenv o expo-constants si uso Expo.

* Nunca subo .env al repositorio. Uso servicios como GitHub Secrets o dotenv en CI/CD.

---

## ¿Qué herramientas usas para pruebas (testing) en el backend y frontend?

NestJS (backend):

* Jest para pruebas unitarias y de integración.

* Supertest para probar endpoints HTTP.

* Testing Module de NestJS para inyectar dependencias simuladas.

React Native (frontend):

* Jest y React Native Testing Library para pruebas de componentes.

* Mock Service Worker (MSW) para simular llamadas HTTP si es necesario.

  En Nestjs

```js

  it('should return 200 on login', () => {
    return request(app.getHttpServer())
      .post('/auth/login')
      .send({ email: 'test@example.com', password: '123456' })
      .expect(200);
  });
```

En React Native

```js
    test('renders login screen', () => {
      const { getByText } = render(<LoginScreen />);
      expect(getByText('Login')).toBeTruthy();
  });

```

---

## ¿Qué es un módulo en NestJS y cómo se estructura una aplicación?

Un módulo en NestJS es una clase que organiza el código por funcionalidad. Cada módulo agrupa controladores, servicios, y proveedores relacionados con una parte específica de la aplicación.

La estructura típica incluye:

* AppModule: el módulo raíz.

* Módulos por dominio: UserModule, AuthModule, ProductModule, etc.

```js

  // user.module.ts
  @Module({
    controllers: [UserController],
    providers: [UserService],
    exports: [UserService], // para compartirlo con otros módulos
  })
  export class UserModule {}

```

Y se importa en el módulo principal:

```js
  @Module({
    imports: [UserModule, AuthModule],
  })
  export class AppModule {}

```

---

## ¿Cuál es la diferencia entre un middleware, un guard, y un interceptor en NestJS?
| Tipo            | Cuándo se ejecuta                                   | Para qué se usa                               |
| --------------- | --------------------------------------------------- | --------------------------------------------- |
| **Middleware**  | Antes que el controlador                            | Modificar la request o hacer logs             |
| **Guard**       | Antes que el controlador (y después del middleware) | Controlar acceso (autenticación/autorización) |
| **Interceptor** | Antes y después del controlador                     | Transformar respuestas, logging, timeouts     |


* Middleware (loggear la petición):

```js
  @Injectable()
  export class LoggerMiddleware {
    use(req: Request, res: Response, next: Function) {
      console.log(`${req.method} ${req.url}`);
      next();
    }
  }
```

* Guard (proteger rutas con JWT):
```js
  @Injectable()
  export class JwtAuthGuard implements CanActivate {
    canActivate(context: ExecutionContext): boolean {
      const request = context.switchToHttp().getRequest();
      return request.user !== undefined;
    }
  }
```

* Interceptor (modificar respuesta):
```js
  @Injectable()
  export class TransformInterceptor implements NestInterceptor {
    intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
      return next.handle().pipe(map(data => ({ data, status: 'success' })));
    }
  }
```

---

##  ¿Qué es un proveedor (provider) y cómo funciona la inyección de dependencias?

Un proveedor es cualquier clase que puede ser inyectada en otra clase usando el sistema de inyección de dependencias (DI) de NestJS. Generalmente, un servicio es un proveedor.

NestJS crea una instancia de cada proveedor y la gestiona por ti (singleton por defecto). Esto ayuda a desacoplar el código.

```js
  @Injectable()
  export class UserService {
    getUsers() {
      return ['John', 'Jane'];
    }
  }

```

Este servicio se inyecta en el controlador:

```js

  @Controller('users')
  export class UserController {
    constructor(private readonly userService: UserService) {}

    @Get()
    getUsers() {
      return this.userService.getUsers();
    }
  }
```
---
## ¿Qué es un decorador en NestJS y para qué se usa @Injectable(), @Controller(), @Get()?

Un decorador en NestJS es una función especial de TypeScript que agrega metadatos a clases, métodos o propiedades. NestJS los usa para saber cómo debe comportarse cada parte de la aplicación.

Decoradores comunes:
* @Injectable()
Marca una clase como inyectable, es decir, puede ser utilizada como proveedor.

* @Controller()
Define que la clase es un controlador, y maneja rutas HTTP.

* @Get(), @Post(), @Put()
Asocian métodos a rutas específicas HTTP dentro de un controlador.

```js
  @Injectable()
  export class UserService {
    findAll() {
      return ['user1', 'user2'];
    }
  }

  @Controller('users')
  export class UserController {
    constructor(private readonly userService: UserService) {}

    @Get()
    getAllUsers() {
      return this.userService.findAll();
    }
  }

```

---
## ¿Cómo implementarías autenticación con JWT?

Usaría el módulo @nestjs/jwt y Passport con la estrategia JWT.

Pasos:

* El usuario inicia sesión con email y password.

* Si es válido, NestJS genera un token JWT con una clave secreta.

* El token se envía al cliente.

* En peticiones futuras, el cliente incluye el token en el header Authorization.


```js
  // auth.service.ts
  async login(user: any) {
    const payload = { username: user.username, sub: user.id };
    return {
      access_token: this.jwtService.sign(payload),
    };
  }
```

```js
  // jwt.strategy.ts
  @Injectable()
  export class JwtStrategy extends PassportStrategy(Strategy) {
    constructor() {
      super({ jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(), secretOrKey: 'secret' });
    }

    async validate(payload: any) {
      return { userId: payload.sub, username: payload.username };
    }
  }
```

---
## ¿Cómo proteger rutas usando guards y roles?
* Uso @UseGuards(JwtAuthGuard) para proteger rutas con JWT.

* Creo un RolesGuard personalizado para permitir acceso solo a usuarios con ciertos roles.

```js
  // roles.guard.ts
  @Injectable()
  export class RolesGuard implements CanActivate {
    canActivate(context: ExecutionContext): boolean {
      const request = context.switchToHttp().getRequest();
      const user = request.user;
      return user?.role === 'admin'; // Solo permite admins
    }
  }
```

```js
  @UseGuards(JwtAuthGuard, RolesGuard)
  @Get('admin-data')
  getAdminData() {
    return 'Solo admins';
  }
```
---
## ¿Cómo manejarías la autorización para diferentes tipos de usuarios?
* Uso decoradores personalizados como @Roles('admin').

* El RolesGuard lee esos metadatos y permite o niega acceso.

```js
  // roles.decorator.ts
  export const Roles = (...roles: string[]) => SetMetadata('roles', roles);

  // roles.guard.ts actualizado
  @Injectable()
  export class RolesGuard implements CanActivate {
    canActivate(context: ExecutionContext): boolean {
      const roles = this.reflector.get<string[]>('roles', context.getHandler());
      const user = context.switchToHttp().getRequest().user;
      return roles.includes(user.role);
    }
  }
```

```js
  @Roles('admin', 'moderator')
  @UseGuards(JwtAuthGuard, RolesGuard)
  @Get('dashboard')
  getDashboard() {
    return 'Área protegida';
  }
```

---
## ¿Cómo se integra una base de datos como PostgreSQL con TypeORM o Prisma?

Con TypeORM:
* Instalo @nestjs/typeorm y pg.

* Creo una entidad.

* Uso TypeOrmModule.forRoot() con la configuración.
```js
  // app.module.ts
  TypeOrmModule.forRoot({
    type: 'postgres',
    host: 'localhost',
    port: 5432,
    username: 'postgres',
    password: '123456',
    database: 'mydb',
    entities: [User],
    synchronize: true,
  })
```

Con Prisma:
* Instalo @prisma/client y prisma.

* Creo el esquema y ejecuto npx prisma generate.

* Uso PrismaService para acceder a la base de datos.

---

##  ¿Cómo harías relaciones entre entidades?

Uso relaciones de TypeORM: OneToOne, OneToMany, ManyToOne, etc.

Ejemplo: User -> Posts (One to Many)
```js
  // user.entity.ts
  @OneToMany(() => Post, post => post.user)
  posts: Post[];

  // post.entity.ts
  @ManyToOne(() => User, user => user.posts)
  user: User;

```

Con Prisma sería:

```js
  model User {
    id    Int     @id @default(autoincrement())
    posts Post[]
  }

  model Post {
    id     Int    @id @default(autoincrement())
    user   User   @relation(fields: [userId], references: [id])
    userId Int
  }
```

---
## ¿Cómo escribirías un servicio para consultar datos con paginación?

Uso parámetros limit y offset o page.

```js
  async getUsers(page = 1, limit = 10) {
    const [data, total] = await this.userRepo.findAndCount({
      skip: (page - 1) * limit,
      take: limit,
    });

    return {
      data,
      total,
      page,
      lastPage: Math.ceil(total / limit),
    };
  }
```

---

## ¿Cómo creas una API REST en NestJS?

* Creo controladores con @Controller().

* Uso decoradores HTTP como @Get(), @Post(), etc.

* Uso servicios para separar la lógica de negocio.

```js
  @Controller('users')
  export class UserController {
    constructor(private readonly userService: UserService) {}

    @Get()
    findAll() {
      return this.userService.getAll();
    }

    @Post()
    create(@Body() dto: CreateUserDto) {
      return this.userService.create(dto);
    }
  }
```
---
## ¿Cómo crearías un WebSocket gateway en NestJS para notificaciones en tiempo real?

* Uso @WebSocketGateway() para crear el gateway.

* Manejo eventos con @SubscribeMessage().

* Uso socket.emit o server.emit para enviar mensajes a clientes.
```js
  @WebSocketGateway()
  export class NotificationsGateway {
    @WebSocketServer()
    server: Server;

    @SubscribeMessage('sendMessage')
    handleMessage(client: Socket, payload: any): void {
      this.server.emit('receiveMessage', payload); // broadcast
    }
  }
```
Cliente (React Native con socket.io-client):

```js
  const socket = io('http://localhost:3000');
  socket.on('receiveMessage', data => {
    console.log('Nuevo mensaje:', data);
  });
```

---
##  ¿Cuál es la diferencia entre React y React Native?

* React se usa para construir interfaces web. Utiliza HTML, CSS y el DOM del navegador.

* React Native se usa para crear apps móviles nativas, usando componentes como View, Text, TouchableOpacity, y no usa HTML/CSS ni el DOM.

| React (Web)    | React Native (Mobile)        |
| -------------- | ---------------------------- |
| Usa HTML y CSS | Usa componentes nativos      |
| DOM            | Bridge a componentes móviles |


---

## ¿Qué es el Virtual DOM y cómo funciona en React Native?

* El Virtual DOM es una representación en memoria del DOM real (en React). React lo usa para calcular qué partes necesitan actualizarse y así mejorar el rendimiento.

* En React Native, no hay DOM, pero hay un sistema similar: React crea una representación virtual del árbol de componentes nativos, y cuando hay cambios, solo actualiza los necesarios a través del Bridge nativo.

---

## ¿Qué hooks sueles usar frecuentemente? ¿Puedes explicar useEffect y useState?

* useState → para manejar estado local.

* useEffect → para ejecutar efectos secundarios (como llamadas a APIs).

* useContext, useRef, useCallback, useMemo

```js
  import React, { useState, useEffect } from 'react';

  const Example = () => {
    const [users, setUsers] = useState([]);

    useEffect(() => {
      fetch('https://api.example.com/users')
        .then(res => res.json())
        .then(data => setUsers(data));
    }, []); // solo se ejecuta una vez al montar el componente

    return <Text>Total users: {users.length}</Text>;
  };
```

---
##  ¿Cómo gestionas el diseño responsivo en dispositivos móviles?
* Uso Dimensions, StyleSheet, Flexbox y librerías como react-native-responsive-dimensions.

* También uso SafeAreaView y ScrollView para una mejor experiencia.

```js
  import { Dimensions } from 'react-native';
  const width = Dimensions.get('window').width;

  const styles = StyleSheet.create({
    container: {
      width: width * 0.9,
      padding: 10,
    },
  });
```

---

## ¿Qué librerías usas para la navegación (por ejemplo: @react-navigation/native)?

* Uso `@react-navigation/native` para navegación stack, tab, y drawer.

* También uso `react-navigation/native-stack`, `react-navigation/bottom-tabs`, etc.

`npm install @react-navigation/native @react-navigation/native-stack react-native-screens react-native-safe-area-context`

```js
  import { NavigationContainer } from '@react-navigation/native';
  import { createNativeStackNavigator } from '@react-navigation/native-stack';

  const Stack = createNativeStackNavigator();
```
---
## ¿Cómo trabajarías con formularios y validaciones en React Native?

* Uso react-hook-form junto con yup para validaciones.

* También se puede usar Formik.

  ```js
    const schema = yup.object().shape({
    email: yup.string().email().required(),
  });

  const { control, handleSubmit } = useForm({ resolver: yupResolver(schema) });

```

---
## ¿Cómo consumes APIs desde React Native? ¿Qué librerías usas: fetch, axios, etc.?

* Uso axios por su simplicidad, manejo de interceptores y headers.

* A veces uso fetch si no necesito muchas configuraciones.

```js
  axios.get('https://api.example.com/data')
    .then(res => setData(res.data))
    .catch(err => console.error(err));

```

---
##  ¿Cómo manejarías estados de carga, errores y éxito al consumir una API?

Uso variables de estado como loading, error, y data.

```js
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    axios.get(API_URL)
      .then(res => setData(res.data))
      .catch(err => setError(err.message))
      .finally(() => setLoading(false));
  }, []);
```

---
## ¿Cómo implementarías login social con Google o Facebook?

* Uso `expo-auth-session`, `@react-native-google-signin/google-signin`, o `react-native-fbsdk-next`.

* Luego envío el idToken o accessToken al backend, que lo verifica y genera un JWT.

```js
  import { GoogleSignin } from '@react-native-google-signin/google-signin';

  GoogleSignin.configure({
    webClientId: 'YOUR_GOOGLE_WEB_CLIENT_ID',
  });

  const userInfo = await GoogleSignin.signIn();
```

---
## ¿Cómo almacenarías el token JWT en el dispositivo? ¿Secure Storage, AsyncStorage?

* Para seguridad, uso expo-secure-store o react-native-keychain.

* Evito guardar tokens en AsyncStorage porque no es cifrado.

```js
  import * as SecureStore from 'expo-secure-store';

  await SecureStore.setItemAsync('token', jwt);

```

---
##  ¿Cómo pruebas tus componentes o lógica en React Native?

* Uso Jest para pruebas unitarias.

* Uso `@testing-library/react-native` para pruebas de componentes.

```js
  test('render welcome text', () => {
    const { getByText } = render(<HomeScreen />);
    expect(getByText('Welcome')).toBeTruthy();
  });

```

---

## ¿Qué herramientas usas para pruebas end-to-end (E2E) o unitarias?

* Unitarias: Jest + Testing Library.

* End-to-End (E2E): uso Detox para simular interacciones reales en un dispositivo/emulador.

