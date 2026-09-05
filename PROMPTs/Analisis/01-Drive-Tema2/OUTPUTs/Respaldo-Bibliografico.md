# Respaldo bibliográfico — qué está sostenido y por quién

> **Por qué existe este documento**: hasta el 2026-09-05, todo lo escrito en [Conceptos-Integridad-y-Restricciones.md](Conceptos-Integridad-y-Restricciones.md) salía del apunte más razonamiento propio. La única referencia externa era la de Microsoft que ya traía el apunte, **y no se había abierto**. Este documento cierra esa deuda: toma cada afirmación del documento de conceptos y dice quién la sostiene.
>
> **Fecha de consulta de las fuentes**: 2026-09-05.
>
> **Lo que apareció al hacerlo**: tres correcciones a lo que habíamos escrito, en la §4. Van primero en importancia aunque estén al final: **una de ellas invalida un ejemplo que dimos**.

---

## Índice

- **[1. Las fuentes consultadas](#1-las-fuentes-consultadas)**
- **[2. Lo que queda confirmado](#2-lo-que-queda-confirmado)** — afirmación por afirmación, con la cita literal
- **[3. Lo que queda como razonamiento propio](#3-lo-que-queda-como-razonamiento-propio)** — sostenido por las fuentes pero no dicho por ellas
- **[4. Lo que hay que corregir](#4-lo-que-hay-que-corregir)** — tres correcciones, una de ellas de fondo
- **[5. Lo que sigue sin verificar](#5-lo-que-sigue-sin-verificar)**

---

## 1. Las fuentes consultadas

| Sigla | Fuente | Qué clase de autoridad es |
| --- | --- | --- |
| **[E&N]** | Elmasri & Navathe, *Fundamentals of Database Systems*, capítulo 5 «The Relational Data Model and Relational Database Constraints», material de cátedra publicado, © 2016 — [PDF](https://www.cs.purdue.edu/homes/bb/cs448_Fall2017/lpdf/Chapter05.pdf) | **Académica**. Es el libro de texto estándar del tema |
| **[MS-CT]** | Microsoft Learn, [`CREATE TABLE (Transact-SQL)`](https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql?view=sql-server-ver17) | **De producto**. Es la referencia que el propio apunte cita |
| **[MS-UC]** | Microsoft Learn, [Unique constraints and check constraints](https://learn.microsoft.com/en-us/sql/relational-databases/tables/unique-constraints-and-check-constraints?view=sql-server-ver17) | **De producto** |
| **[MS-DI]** | Microsoft Learn, [Data Integrity](https://learn.microsoft.com/en-us/previous-versions/sql/sql-server-2008-r2/ms184276(v=sql.105)) *(documentación archivada de SQL Server 2008 R2)* | **De producto**. Es el origen de la clasificación en cuatro tipos |
| **[PG]** | PostgreSQL, [Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html) | **De producto**, y la que más se ajusta al estándar SQL |
| **[Codd]** | E. F. Codd, «A Relational Model of Data for Large Shared Data Banks», *CACM* 13(6), 1970 | **Académica, fundacional** |

**Lo que no se consiguió:** el texto normativo de ISO/IEC 9075 (el estándar SQL). Las copias públicas que se encontraron están truncadas antes de las cláusulas que interesaban. Donde hacía falta el estándar, se usó **[PG]**, que documenta explícitamente su apego a él — y eso queda dicho, no disimulado.

---

## 2. Lo que queda confirmado

### 2.1 La restricción es la condición; la integridad es el estado que la satisface

**Es la definición de [E&N], casi con las mismas palabras.**

> «Constraints are conditions that must hold on all valid relation states.» — [E&N], slide 5-22

> «A relational database state DB of S is a set of relation states … such that the ri relation states **satisfy the integrity constraints** specified in IC. … **A database state that does not meet the constraints is an invalid state**.» — [E&N], slide 5-29

La academia usa exactamente el par que decidimos: **la restricción es una condición; el estado que la satisface es el estado válido.** No hay «reglas» de los dos lados. Y confirma la §2.1 del documento de conceptos: la integridad se predica de un **estado**, y por eso se comprueba en las filas.

También lo dice Microsoft, con el verbo correcto:

> «Constraints are **rules that the SQL Server Database Engine enforces for you**.» — [MS-UC]

*Enforces*, no *imposes*: el motor **hace cumplir** la regla, no la decide. Es lo que sostiene la §2.6 del [Registro](Registro.md).

### 2.2 Una restricción de columna se puede escribir como restricción de tabla; al revés no siempre

**Es la afirmación central de la §4.3 del documento de conceptos, y [PG] la dice literal.**

> «**Column constraints can also be written as table constraints, while the reverse is not necessarily possible**, since a column constraint is supposed to refer to only the column it is attached to.» — [PG]

Es exactamente lo que sosteníamos: **el eje columna/tabla clasifica declaraciones, no restricciones.** Una misma restricción se puede escribir de las dos maneras salvo que nombre más de una columna.

Y [MS-CT] lo confirma del lado de SQL Server, con la gramática: `<column_constraint>` admite `PRIMARY KEY | UNIQUE`, `FOREIGN KEY … REFERENCES` y `CHECK`; `<table_constraint>` admite los mismos cuatro. **`DEFAULT` y `NOT NULL` solo existen a nivel de columna** — que era nuestra afirmación de la §4.3, ahora verificada contra la gramática del producto.

Esto además confirma el [Hallazgo §1.7](Hallazgos-01-Fuente.md): la `FOREIGN KEY` **sí** puede declararse a nivel de columna, contra lo que dice la clasificación del apunte y de acuerdo con lo que muestra su propio Ejemplo 2.

### 2.3 Las seis celdas de la grilla tienen contenido

**Verificado contra la gramática de [MS-CT].** `PRIMARY KEY`, `UNIQUE`, `FOREIGN KEY` y `CHECK` aparecen las cuatro en `<column_constraint>` **y** en `<table_constraint>`. Las tres integridades se pueden escribir de las dos formas, que es lo que sostiene la §5.1 del documento de conceptos.

### 2.4 Un `NULL` en la clave foránea no es un huérfano

**Confirmado por las dos fuentes, académica y de producto.**

> «The value in the foreign key column (or columns) FK … can be either: 1) a value of an existing primary key value … **or (2) a null**.» — [E&N], slide 5-34

> «Normally, a referencing row **need not satisfy the foreign key constraint if any of its referencing columns are null**.» — [PG]

Sostiene la §3.2 del documento de conceptos y la §5.3 del [Registro](Registro.md).

### 2.5 `UNIQUE` admite un solo `NULL` en SQL Server y varios en PostgreSQL

**La afirmación del apunte era correcta, y ahora está verificada de los dos lados.**

> «Unlike `PRIMARY KEY` constraints, `UNIQUE` constraints allow for the value `NULL`. However … **only one null value is allowed per column**.» — [MS-UC]

> «By default, two null values are not considered equal in this comparison. That means **even in the presence of a unique constraint it is possible to store duplicate rows that contain a null value** in at least one of the constrained columns.» — [PG]

### 2.6 `NOT NULL` es integridad de dominio

**Confirmado, y [PG] da la razón exacta:**

> «A not-null constraint is **functionally equivalent to creating a check constraint `CHECK (column_name IS NOT NULL)`**» — [PG]

Si `NOT NULL` es un `CHECK` sobre una columna, entonces es del mismo tipo que un `CHECK`: restringe qué valores admite esa columna. Eso es integridad de dominio, como la clasifica el apunte y como la clasifica [MS-DI].

### 2.7 `CHECK` asegura integridad de dominio, `UNIQUE` asegura integridad de entidad

**Microsoft lo dice de las dos, y es lo que sostiene la §5.2 del documento de conceptos.**

> «`CHECK` constraints **enforce domain integrity** by limiting the values that are accepted by one or more columns.» — [MS-UC]

> «[`UNIQUE`:] A constraint that **provides entity integrity** for a specified column or columns through a unique index.» — [MS-CT]

Esto es la confirmación del caso que abrió el debate: **`UNIQUE` y `FOREIGN KEY` conviven en el grupo «restricción de tabla» asegurando integridades distintas** —de entidad una, referencial la otra—. El grupo no dice nada sobre qué preserva cada una.

---

## 3. Lo que queda como razonamiento propio

Sostenido por las fuentes, pero **no afirmado por ellas**. Se marca para que se pueda discutir sin confundirlo con una cita.

| Afirmación nuestra | En qué se apoya | Qué le agregamos |
| --- | --- | --- |
| «Son dos miradas, no dos niveles» | [E&N] clasifica integridades; [MS-CT] y [PG] clasifican declaraciones | **La grilla que las cruza es nuestra.** Ninguna fuente la presenta así |
| «La fila es homogénea, la columna no» (§5.2) | Se deduce de la §2.7 de acá | Es una lectura de la grilla, no una cita |
| «La restricción no crea la integridad: la vuelve inevitable» (§2.2) | Se deduce de [E&N]: un estado puede ser válido sin que exista la restricción que lo obligue | La formulación es nuestra |
| «La forma de columna no deja poner nombre y por eso el error se vuelve ilegible» (§4.4) | [MS-CT]: «If *constraint_name* isn't supplied, **a system-generated name is assigned** … The constraint name appears in any error message about constraint violations» | La consecuencia práctica —«el error se vuelve ilegible»— la sacamos nosotros |
| «Integridad no es verdad» (§1) | [E&N] define validez respecto de las restricciones, nunca respecto del mundo | La formulación y el ejemplo de Arturo son nuestros |

---

## 4. Lo que hay que corregir

Tres cosas. Se anotan acá y **se corrigen en el documento de conceptos**, no se arreglan en silencio.

### 4.1 «Integridad de entidad» no significa lo que dice el apunte — significa menos

**Corrección de fondo.** El apunte —y nosotros detrás— definimos integridad de entidad como «cada fila se identifica de forma única». En [E&N] son **dos restricciones distintas**:

> **Key constraint** — «Superkey of R: … **No two tuples in any valid relation state r(R) will have the same value for SK**» — slide 5-23
>
> **Entity Integrity** — «The primary key attributes PK of each relation schema R in S **cannot have null values** in any tuple of r(R).» — slide 5-32

Es decir: **la unicidad viene de que la clave es clave; la integridad de entidad es solo la parte del «no puede ser nulo».** Lo que el apunte presenta como una sola propiedad de la `PRIMARY KEY` —«no permite `NULL`, no permite valores repetidos»— son dos reglas de origen distinto que la `PRIMARY KEY` hace cumplir juntas.

**Qué cambia en la práctica:** nada del SQL. Cambia lo que se puede decir con precisión. Y explica por qué `UNIQUE` sí admite un `NULL` (§2.5): cumple la restricción de clave, no la de integridad de entidad — que solo se le exige a la primaria.

### 4.2 El ejemplo de la fecha de entrega **no** es integridad de dominio

**La corrección más importante, porque afecta a un ejemplo que dimos como resuelto.** En la §3.4 del documento de conceptos escribimos que «la fecha de entrega no puede ser anterior a la fecha de pedido» es «integridad de dominio sobre una fila». Ninguna fuente lo respalda:

> «**Domain constraint**: Every value in a tuple must be from the domain of **its attribute**» — [E&N], slide 5-22

> «**Domain Integrity** is the validity of entries for **a specific column**.» — [MS-DI]

Las dos definen dominio **por atributo**, y una condición entre dos columnas no lo es. Bajo la taxonomía de Microsoft, esa regla cae en **user-defined integrity**; en [E&N] es una restricción de tupla que se expresa con `CHECK` pero que no entra en ninguno de los tres tipos clásicos.

**Lo correcto es decir que el esquema de tres tipos no tiene casillero para ella**, y no forzarla adentro de «dominio».

### 4.3 «Integridad de negocio» sí es una categoría de la industria — pero definida por exclusión

**Corrección de tono, no de fondo.** Dijimos que «de negocio» no era un cuarto tipo sino un error de categoría. Eso fue injusto con el apunte: la clasificación en cuatro es de Microsoft, palabra por palabra:

> «**User-Defined Integrity** lets you define specific business rules that **do not fall into one of the other integrity categories**.» — [MS-DI]

Y la academia tiene su equivalente, con otro nombre y otro criterio:

> «**Semantic Integrity Constraints**: based on application semantics and **cannot be expressed by the model per se** … SQL-99 allows `CREATE TRIGGER` and `CREATE ASSERTION` to express some of these» — [E&N], slide 5-38

**Lo que sobrevive de nuestro argumento, y ahora con respaldo:** las dos fuentes definen el cuarto grupo **por lo que no es** —Microsoft «lo que no cae en las otras», [E&N] «lo que el modelo no puede expresar»—, mientras que a los tres primeros los definen por su objeto. **El cuarto no está en el mismo eje**, que era la observación. Lo que hay que retirar es haberlo llamado un error del apunte: el apunte está repitiendo la clasificación del fabricante.

Y hay un premio: la academia le da nombre propio a lo que nosotros llamamos «la brecha» — **restricciones de integridad semántica**, y las define exactamente por no ser declarables.

---

## 5. Lo que sigue sin verificar

| Ausencia | Tipo | Estado |
| --- | --- | --- |
| El texto de ISO/IEC 9075 | **Pendiente** | Las copias públicas encontradas están truncadas. Se usó [PG] como sustituto declarado |
| `NOT NULL` como objeto del catálogo de SQL Server | **Pendiente** | Sigue siendo un supuesto nuestro. [PG] dice que es *funcionalmente equivalente* a un `CHECK`, que no es lo mismo que decir cómo lo guarda SQL Server |
| Que los scripts del apunte corran | **Pendiente** | Nada se ejecutó todavía contra un motor |
| El texto original de [Codd] 1970 | **Pendiente** | La atribución de «entity integrity» y «referential integrity» a Codd se tomó de fuentes secundarias y de [E&N]; no se leyó el paper |
| El repositorio de ejemplos del apunte | **Pendiente** | Sin abrir |

---

> **Una afirmación sin fuente no es falsa: es una que todavía no sabemos si podemos usar.**
