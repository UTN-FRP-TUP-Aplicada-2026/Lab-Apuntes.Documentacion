# Registro — Mapeo de herencia

> **Invocación**: se produce desde `PROMPTs/Analisis/02-Drive-Tema3/01-Analisis-Y-Migracion.md`.
>
> **Overview**: los conceptos del Tema 3 pasados en limpio. Tiene lo **acordado** —lo que se puede usar sin volver a discutirlo— y lo **abierto**. Es el paso intermedio entre el apunte original y el apunte nuevo: acá vive el *por qué* de cada decisión que allá aparece ya tomada.
>
> **Estado**: 2026-09-05, primera pasada.
>
> **Material**: el apunte extractado en [Extraccion-01-Herencia.md](Extraccion-01-Herencia.md), sus defectos en [Hallazgos-01-Herencia.md](Hallazgos-01-Herencia.md), y las fuentes de la §5.

---

## Índice

- **[1. Definiciones](#1-definiciones)**
- **[2. Lo acordado](#2-lo-acordado)** — cinco decisiones para el apunte nuevo
- **[3. El problema que el apunte original no nombra](#3-el-problema-que-el-apunte-original-no-nombra)** — dónde queda la integridad de la jerarquía
- **[4. Lo que aporta la academia: cuatro opciones, no tres](#4-lo-que-aporta-la-academia-cuatro-opciones-no-tres)**
- **[5. Lo que aporta la industria: qué hace EF Core](#5-lo-que-aporta-la-industria-qué-hace-ef-core)**
- **[6. Lo abierto](#6-lo-abierto)**
- **[7. Lo que este registro no cubre](#7-lo-que-este-registro-no-cubre)**

---

## 1. Definiciones

Fijadas para todo el Tema 3.

> **Jerarquía** — una clase base y sus derivadas. En el ejemplo del apunte: `Figura`, con `Rectangulo` y `Circulo`.
>
> **Especialización** — el nombre que la bibliografía académica le da a la relación entre la superclase y sus subclases.
>
> **Discriminador** — columna que indica de qué subclase es cada fila. Elmasri & Navathe lo llaman *type attribute*; EF Core, *discriminator column*.
>
> **Disjunción** — que una instancia pertenezca **a lo sumo** a una subclase. Un rectángulo no es además un círculo.
>
> **Totalidad** — que toda instancia de la superclase pertenezca **al menos** a una subclase. No existen «figuras a secas».
>
> **TPH / TPT / TPC** — los tres nombres de la industria para las estrategias de mapeo: tabla única con discriminador, tabla por tipo, tabla por clase concreta.

**Los dos términos que el apunte original no tiene son «disjunción» y «totalidad»** — y son los que deciden si una estrategia es siquiera aplicable (§4).

---

## 2. Lo acordado

Cinco decisiones ya tomadas para el apunte nuevo. Las tres primeras salen de los hallazgos; las dos últimas, de las fuentes.

### 2.1 Se corrige el SQL siguiendo a las figuras, no al revés

**Decidido el 2026-09-05.** En los tres ejemplos las figuras están bien y el SQL está mal ([Hallazgos §1](Hallazgos-01-Herencia.md)). Las figuras son la intención del autor: `Tipo` no único, `Circulos` con `Radio`, tablas `Figuras_Rectangulos` y `Figuras_Circulos`.

**Qué se descartó:** transcribir el SQL tal cual y anotar los errores al pie. Un apunte cuyo código no compila obliga al alumno a depurar antes de aprender.

### 2.2 El ejemplo sigue siendo `Figura` / `Rectangulo` / `Circulo`

**Decidido el 2026-09-05.** Es minimalista y no realista, y eso es una ventaja: dos subclases con atributos disjuntos hacen visible el problema de los nulos sin ruido de dominio. **Se reusa tal cual**, incluidos los valores `1.0` y `3.1`.

### 2.3 Las tres estrategias se presentan con la misma plantilla

**Decidido el 2026-09-05.** Cada una con: qué hace, el DDL, los datos de ejemplo, la consulta que la caracteriza, y **qué integridad no puede garantizar**. La última parte es la que el apunte original no tiene y la que convierte el tema en una decisión y no en un catálogo.

### 2.4 Se agrega el eje de validez, además del de conveniencia

**Decidido el 2026-09-05, a partir de la §4.** La tabla comparativa del original compara por ventaja/desventaja. Se le suma **cuándo cada estrategia es aplicable** —disjunción y totalidad—, que es criterio de corrección y no de gusto.

### 2.5 Se declara qué queda para el ORM y qué es del modelo

**Decidido el 2026-09-05.** El apunte original cita nueve referencias, todas sobre un mapeador concreto. El apunte nuevo mantiene el nivel relacional: **las estrategias existen aunque no haya ORM**, y lo que un ORM aporta es automatizarlas. Esto lo hereda del criterio ya adoptado en el Tema 2.

---

## 3. El problema que el apunte original no nombra

**La pregunta que ordena todo el tema: ¿dónde queda la integridad cuando la jerarquía se aplana?**

Cada estrategia rompe una garantía distinta, y ninguna las conserva todas:

| Estrategia | Qué **no** puede garantizar el esquema |
| --- | --- |
| **TPH** | Que las columnas nulas se correspondan con el discriminador. Nada impide un rectángulo con `Radio` ([Hallazgos §3.1](Hallazgos-01-Herencia.md)) |
| **TPT** | Que exista exactamente una fila hija por cada fila base. Una `Figura` puede quedar sin subclase, o estar en dos |
| **TPC** | Que los identificadores sean únicos en toda la jerarquía, y que alguien pueda referenciar «una figura cualquiera» ([Hallazgos §3.2](Hallazgos-01-Herencia.md)) |

**Los tres agujeros son de integridad, y por eso el tema pertenece a esta materia y no a una de modelado.** Dos de los tres se pueden tapar con restricciones; el tercero —el de TPT— no del todo.

Esa tabla es el aporte conceptual central del apunte nuevo.

---

## 4. Lo que aporta la academia: cuatro opciones, no tres

Elmasri & Navathe tratan el mismo problema como *mapeo de especialización* y enumeran **cuatro** opciones, con sus condiciones de aplicabilidad —cita literal del capítulo 9:

| Opción | Definición literal | Nombre de industria |
| --- | --- | --- |
| **8A** — *Multiple relations: superclass and subclasses* | «Create a relation L for C … Create a relation Li for each subclass Si … **This option works for any specialization (total or partial, disjoint or overlapping)**» | **TPT** |
| **8B** — *Multiple relations: subclass relations only* | «Create a relation Li for each subclass Si … **This option only works for a specialization whose subclasses are total** (every entity in the superclass must belong to (at least) one of the subclasses)» | **TPC** |
| **8C** — *Single relation with one type attribute* | «Create a single relation L … **The attribute t is called a type (or discriminating) attribute** that indicates the subclass to which each tuple belongs» | **TPH** |
| **8D** — *Single relation with multiple type attributes* | «Create a single relation schema L … **Each ti … is a Boolean type attribute** indicating whether a tuple belongs to the subclass Si» | *(sin nombre en la industria)* |

**Tres cosas que esto agrega y que no están en el apunte original:**

1. **8A/TPT es la única que sirve siempre.** Las otras tienen condiciones. Eso reordena la comparación: TPT no es «la normalizada y costosa», es **la que no exige nada del modelo**.
2. **8B/TPC exige totalidad.** Si puede existir una figura que no sea ni rectángulo ni círculo, TPC no la puede representar: no hay tabla donde ponerla.
3. **Existe una cuarta opción, y la industria no la nombra.** 8D usa un booleano por subclase en vez de un discriminador único, y es la respuesta para **subclases superpuestas** — algo que un discriminador de un solo valor no puede expresar (*razonado: un atributo que guarda un valor solo puede nombrar una subclase*).

**Por qué la industria se quedó con tres:** los ORM asumen jerarquías disjuntas, porque en un lenguaje de objetos un objeto tiene exactamente una clase concreta. **La cuarta opción no le hace falta a un ORM, pero sí al modelo relacional**, que puede representar cosas que el modelo de objetos no.

---

## 5. Lo que aporta la industria: qué hace EF Core

La documentación de EF Core —una de las nueve referencias del apunte original, ahora leída— confirma la terminología y agrega cuatro precisiones útiles. Todas literales:

| Tema | Cita |
| --- | --- |
| **Cuál es la estrategia por omisión** | «*By default, EF maps the inheritance using the table-per-hierarchy (TPH) pattern. TPH uses a single table … and a discriminator column is used to identify which type each row represents*» |
| **Los nulos de TPH** | «*Database columns are automatically made nullable as necessary when using TPH mapping*» |
| **Qué genera para TPT** | `CONSTRAINT [FK_RssBlogs_Blogs_BlogId] FOREIGN KEY ([BlogId]) REFERENCES [Blogs] ([BlogId]) **ON DELETE NO ACTION**` |
| **Cuándo usar TPT** | «*In many cases, TPT shows inferior performance when compared to TPH*» … «***Use TPT only if constrained to do so by external factors***» |
| **Las claves en TPC** | «*EF Core requires that all entities in a hierarchy have a unique key value, even if the entities have different types* … *a simple `Identity` column cannot be used*» — y lo resuelve con una secuencia compartida: `DEFAULT (NEXT VALUE FOR [AnimalSequence])` |
| **Las foráneas en TPC** | «*when using TPC, the primary key for any given animal is stored in the table corresponding to the concrete type* … *This means an FK constraint cannot be created for this relationship*» |
| **TPC y clases abstractas** | «*There are no tables for the `Animal` or `Pet` types, since these are `abstract` in the object model*» |

**Dos tensiones con el apunte original, que hay que resolver en el nuevo:**

### 5.1 «Ejemplo ideal» de TPT

El apunte dice que TPT es ideal para «modelos grandes y normalizados». EF Core dice **«use TPT only if constrained to do so by external factors»**. No se contradicen del todo —uno habla de diseño, el otro de rendimiento medido—, pero **presentar TPT como el caso ideal sin mencionar su costo deja al alumno con media película**.

**Resolución adoptada:** el apunte nuevo distingue los dos ejes explícitamente. TPT gana en **integridad y validez** (§4, opción 8A); pierde en **rendimiento** (cita de EF). Las dos cosas son ciertas al mismo tiempo.

### 5.2 El `CASCADE` de TPT

El apunte declara `ON DELETE CASCADE ON UPDATE CASCADE`; EF Core genera `ON DELETE NO ACTION`.

**Acá el apunte tiene un argumento mejor que la herramienta**, y conviene decirlo: la fila hija de TPT **es** la misma entidad que la fila base, no otra entidad relacionada. Borrar la base y dejar la hija produce una fila que no representa nada. **`ON DELETE CASCADE` es la lectura correcta del modelo**; EF elige `NO ACTION` porque borra las dos filas desde el código y no necesita que el motor lo haga.

*(Y esto engancha con el Tema 2: es exactamente la dependencia de existencia — la fila hija no existe sin la base.)*

---

## 6. Lo abierto

### 6.1 ¿Cuánto `CHECK` poner en TPH?

Se puede declarar la correspondencia entre discriminador y columnas —«si `Tipo = 1` entonces `Radio IS NULL`»— con un `CHECK`. **Queda por decidir si el apunte lo muestra completo o solo lo nombra.** A favor: es la única forma de que TPH tenga integridad real. En contra: con más subclases el `CHECK` crece y ensucia el ejemplo minimalista.

### 6.2 ¿Se muestra la cuarta opción de Elmasri?

8D no tiene nombre de industria y no aparece en ningún ORM. **A favor de incluirla:** explica por qué las otras tres asumen disjunción, y es barata de mostrar. **En contra:** puede confundir a quien después trabaje solo con un ORM.

### 6.3 ¿Cómo se garantiza «exactamente una subclase» en TPT?

Es el agujero de la §3 que no tiene solución declarativa simple: nada impide que una `Figura` tenga fila en `Figuras_Rectangulos` **y** en `Figuras_Circulos`, ni que no tenga en ninguna. Hace falta un `CHECK` que mire otras tablas —que no se puede—, un *trigger*, o vivir con el agujero declarándolo.

### 6.4 ¿El Tema 3 tiene guía propia?

En `Temas.md`, el Tema 3 enlaza como práctica **las mismas guías 2.1 y 2.2 del Tema 2**. O falta la guía de herencia, o el enlace está mal.

---

## 7. Lo que este registro no cubre

| Ausencia | Tipo | Qué corresponde |
| --- | --- | --- |
| Que los scripts corran | **Pendiente** | Nada se ejecutó contra un SQL Server |
| El repositorio de ejemplos del apunte | **Pendiente** | Sin abrir |
| Vistas para reconstruir la jerarquía | **Pendiente** | En TPC y TPT, una vista con `UNION` devuelve «todas las figuras»; no está tratado |
| Herencia múltiple y jerarquías de más de dos niveles | **No aplica acá** | El ejemplo tiene un nivel y dos hojas |
| Rendimiento medido | **Otra herramienta** | Las afirmaciones de costo son de la documentación, no de mediciones propias |
