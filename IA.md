# IA.md — Registro de uso de IA

Herramienta utilizada: **Claude**

A continuación se listan los prompts utilizados durante el desarrollo del Laboratorio 4, junto con una breve descripción de para qué sirvió cada uno y qué se generó a partir de él.

---

**Prompt:**
> tengo que hacer este taller, explicame en que consiste, que se esta haciendo, como se hace, lo que se debe hacer etc, para lograr comprender el contenido

**Uso:** Se pidió una explicación general del laboratorio antes de empezar, para entender qué es el modelo C4, sus 4 niveles, y qué se esperaba en cada uno según la rúbrica.

**Resultado:** No se generó código; fue una explicación conceptual del enunciado.

---

**Prompt:**
> que diferencia hay entre un system context diagram y uno de contenedores, se supone que van en el mismo archivo o separados?

**Uso:** Se preguntó por la diferencia conceptual entre los niveles del modelo C4 antes de empezar a dibujar.

**Resultado:** Se explicó el criterio de "zoom" entre niveles y se sugirió mantener un archivo `.puml` por nivel.

---

**Prompt:**
> por que se dice que cada microservicio deberia tener su propia base de datos, que problema evita eso exactamente?

**Uso:** Se preguntó por el principio de propiedad exclusiva de datos, para entender por qué era un criterio evaluado en la rúbrica y no solo una preferencia de estilo.

**Resultado:** Se explicó el problema de acoplamiento de esquema entre servicios y por qué el acceso cruzado a otra base de datos rompe la independencia que se busca al separar en microservicios.

---

**Prompt:**
> que diferencia hay entre comunicacion sincrona y asincrona entre microservicios, cuando conviene usar cada una?

**Uso:** Se preguntó por los patrones de comunicación antes de decidir cómo iban a interactuar Link Service y User Service en el diagrama de contenedores.

**Resultado:** Se explicaron los trade-offs de cada enfoque (latencia, acoplamiento temporal, complejidad de infraestructura) para justificar la elección de comunicación síncrona vía HTTP/REST en el documento de arquitectura.

---

**Prompt:**
> que es un api gateway y por que no puedo simplemente dejar que cada servicio reciba las peticiones directo?

**Uso:** Se preguntó por el rol del API Gateway antes de incluirlo en el diagrama de Nivel 2.

**Resultado:** Se explicó su función como punto de entrada único (enrutamiento, y potencialmente autenticación/rate limiting centralizado) y se justificó su inclusión en el diagrama de Contenedores.

---

**Prompt:**
> considerando esa gran arquitectura siento que es demasiado para lo que yo podria, se podria hacer mas simple?

**Uso:** Se pidió una versión simplificada del diagrama de Contenedores, sin Redis, colas de mensajes ni servicios adicionales, acorde al alcance de un proyecto individual de curso.

**Resultado:** Versión simplificada con 2 servicios de dominio en vez de 4-5.

---

**Prompt:**
> el nombre del contenedor en el diagrama tiene que coincidir exacto con el namespace de c# o puede ser mas descriptivo?

**Uso:** Se preguntó por convención de nomenclatura al pasar de nombres de clases reales a nombres de componentes en el diagrama.

**Resultado:** Se explicó que el nombre del componente puede ser descriptivo, y que el nombre técnico real se deja en la descripción del componente.

---

**Prompt:**
> en el punto de data ownership tengo que explicar como se resuelve el userid si esta en otra base de datos o eso no importa a este nivel?

**Uso:** Se preguntó si el problema de la foreign key cruzada entre `Link` y `User` debía abordarse explícitamente en el documento de arquitectura.

**Resultado:** Se agregó un párrafo específico en la sección de "Propiedad de los datos" explicando el manejo del `UserId` como referencia sin join directo entre bases de datos.

---

**Prompt:**
> structurizr dsl y plantuml con extension c4 hacen lo mismo o hay diferencias importantes entre usar uno u otro?

**Uso:** Se preguntó por las opciones de herramientas mencionadas en el enunciado antes de decidir con cuál trabajar.

**Resultado:** Se explicaron las diferencias (Structurizr como DSL propio orientado a C4, PlantUML como lenguaje general con una librería de extensión C4) y se optó por PlantUML por ser más simple de integrar y visualizar sin herramientas adicionales.

---

**Prompt:**
> me puedes ayudar a revisar la redaccion y ortografia del documento de arquitectura y completar las ideas que me quedaron cortas en cada punto?

**Uso:** Se pidió apoyo para mejorar la redacción y ortografía de `architecture.md`, y ayudar a completar/desarrollar mejor las ideas de cada sección (rationale, comunicación, data ownership, escalabilidad, failure modes, stack tecnológico).

**Resultado:** Versión revisada de `architecture.md` con correcciones de redacción y ortografía, y las ideas de cada punto desarrolladas con mayor detalle.