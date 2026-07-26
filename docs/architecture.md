# Arquitectura de Microservicios — Shortly

## 1. Justificación de la descomposición de servicios

Para este laboratorio decidí no inventar servicios que no tuvieran relación con el código actual de Shortly, sino partir de la separación que ya existe en el monolito. Revisando el proyecto, `Link` y `User` ya están tratados como dos dominios independientes: cada uno tiene su propia entidad, su propio `Service`, su propia interfaz y su propio `Repository`. Eso me hizo sentido como punto de partida natural para trazar el límite de los microservicios, en vez de partir de cero con una arquitectura "ideal" que no tuviera nada que ver con lo que ya está construido.

Así que terminé con dos servicios:

Link Service: se encarga de todo lo relacionado a los links cortos — crearlos, listarlos, y resolver la redirección (`GET /{shortUrl}`). Decidí dejar la redirección dentro del mismo servicio y no separarla en un "Redirect Service" aparte (que en un principio había considerado), porque en el código ambas cosas comparten la misma entidad y el mismo repositorio, y separarlas me habría obligado a resolver problemas de consistencia entre bases de datos que no le aportan nada al alcance de este trabajo.

User Service: maneja registro y login de usuarios. Es más simple que Link Service, pero tiene sentido que sea su propio servicio porque conceptualmente es un dominio totalmente distinto (autenticación vs. gestión de links), y en el código ya está separado así.


## 2. Patrones de comunicación

Toda la comunicación entre los servicios y el resto del sistema es síncrona, vía HTTP/REST. No usé mensajería asíncrona (colas, eventos) porque para el tamaño de este proyecto no se justifica la complejidad extra que eso agrega, habría tenido que definir un broker, manejar reintentos, etc., solo para procesos que en la práctica son rápidos y no necesitan desacoplarse (crear un link, hacer login, incrementar un contador).

El flujo típico es: el cliente (Web App o un consumidor externo de la API) llama al API Gateway, que enruta la petición al servicio correspondiente según el path. Dentro de cada servicio, el endpoint le pasa la petición a la capa de lógica de negocio (el `*Service`), que a su vez usa el repositorio correspondiente para leer o escribir en la base de datos.

Un caso particular es el login: cuando el usuario inicia sesión en User Service, se genera una cookie de sesión cuyo ticket se guarda en un caché distribuido (`IDistributedCache`), y esa misma cookie se valida en cada request posterior que haga el usuario. Esto ya estaba implementado así en el código (`MemoryCacheTicketStore`), así que lo mantuve tal cual en la arquitectura propuesta.

## 3. Propiedad de los datos

Cada servicio es dueño exclusivo de su propia base de datos, y nadie más accede a ella directamente:

- Link Service es dueño de la tabla `links` (Url, ShortUrl, Clicks, UserId, CreatedAt).
- User Service es dueño de la tabla `users` (Email, Password hasheada).

Un detalle que noté al revisar la entidad `Link`: tiene un campo `UserId` que hace referencia a un usuario, pero ese usuario vive en la base de datos de otro servicio. En un monolito esto se resuelve con una foreign key normal, pero al separar en microservicios ya no se puede hacer un join directo entre las dos bases de datos. La solución que propongo es que Link Service guarde solamente el `UserId` (como referencia, sin validarlo contra la base de User Service en cada operación), y que si en algún momento se necesita mostrar información del usuario dueño de un link, sea el cliente (Web App) el que haga dos llamadas separadas — una a cada servicio — y junte la información en la capa de presentación. No es la solución más bonita, pero evita acoplar los dos servicios a nivel de base de datos, que es justamente lo que se busca evitar al separar en microservicios.

## 4. Consideraciones de escalabilidad

De los dos servicios, Link Service es el que más carga recibiría en un escenario real, porque la redirección (`GET /{shortUrl}`) es la operación que más tráfico tiene — cada vez que alguien hace clic en un link corto, no cada vez que alguien crea uno. User Service en cambio se usa mucho menos (solo al hacer login o registrarse), así que no necesitaría tantas instancias corriendo en paralelo.

Esto significa que en un despliegue real, Link Service debería poder escalar horizontalmente de forma independiente a User Service — por ejemplo, tener 3 o 4 instancias de Link Service corriendo detrás del Gateway, pero solo 1 o 2 de User Service. Esto es justamente uno de los beneficios de haberlos separado: si esto siguiera siendo un monolito, no podría escalar solo la parte de redirección sin escalar todo lo demás junto con ella.

## 5. Modos de falla

Si User Service se cae, el impacto es que nadie nuevo puede registrarse ni hacer login, pero los usuarios que ya tienen una sesión activa podrían seguir usando Link Service sin problema, porque la validación de la cookie no depende de que User Service esté funcionando en ese momento.

Si Link Service se cae, es más grave: nadie puede crear links nuevos, y lo más importante, las redirecciones dejan de funcionar que es la funcionalidad principal del sistema. Este sería el punto de falla más crítico de toda la arquitectura, así que en un escenario real sería el primer candidato a tener réplicas y algún mecanismo de failover.

Ninguno de los dos servicios tiene ahora mismo mecanismos de resiliencia como reintentos automáticos o circuit breakers, para el alcance de este trabajo no los implementé, pero los dejo mencionados como algo a considerar si esto se llevara a producción.

## 6. Stack tecnológico propuesto

Como el código base ya está en ASP.NET Core, mantuve la misma tecnología para ambos servicios en vez de proponer algo distinto:

- Framework: ASP.NET Core con Minimal API para los endpoints.
- Base de datos: Se mantiene SQLite.
- Acceso a datos: Entity Framework Core, tal como está en el código actual.
- Autenticación: Cookie Authentication con un ticket store respaldado en caché distribuido, tal como ya está implementado.
- API Gateway: no está implementado en el código actual, pero se podría lograr con algo simple como YARP (el reverse proxy de Microsoft para .NET), ya que mantiene todo dentro del mismo ecosistema tecnológico del resto del proyecto.
