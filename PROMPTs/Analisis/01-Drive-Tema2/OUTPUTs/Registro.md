# Registro — Integridad y restricciones

> **Invocación**: se produce y se actualiza desde `/APLICADA/LAB/Apuntes.Documentacion/PROMPTs/Analisis/Analisis.md`.
>
> **Overview**: este archivo es el pasado en limpio de nuestra discusión sobre integridad y restricciones. Tiene dos mitades que no se mezclan: **lo acordado** —lo que ya podemos usar sin volver a discutirlo— y **lo abierto** —las preguntas puestas sobre la mesa, cada una con la evidencia que la origina y sin respuesta cerrada—. Una pregunta pasa de la segunda mitad a la primera solo cuando la decidimos, y queda anotado quién la decidió y con qué argumento.
>
> **Estado**: 2026-09-05. Dos acuerdos cerrados (§4), tomados de la discusión de la §2 y la §3. Las siete preguntas de la §5 siguen abiertas.
>
> **Respaldo**: cada afirmación con su fuente en [Respaldo-Bibliografico.md](Respaldo-Bibliografico.md) —consultado el 2026-09-05—, que además lista **tres correcciones** a lo que habíamos escrito.
>
> **Producto en limpio**: el documento [Conceptos-Integridad-y-Restricciones.md](Conceptos-Integridad-y-Restricciones.md) presenta los dos conceptos ya decididos. Este registro conserva **por qué** quedaron así y qué sigue abierto.
>
> **Material leído**: el apunte del Tema 2 —extractado en [Extraccion-01-Integridad-y-Restricciones.md](Extraccion-01-Integridad-y-Restricciones.md)—, el temario [`Temas.md`](../../../../Drive/Temas.md), y los defectos que aparecieron en el camino, en [Hallazgos-01-Fuente.md](Hallazgos-01-Fuente.md).

---

## Índice

- **[1. Las definiciones de la fuente](#1-las-definiciones-de-la-fuente)** — transcriptas tal como están, para poder discutirlas
- **[2. La separación propuesta](#2-la-separación-propuesta)** — por qué integridad y restricción no se separan en la fuente, y el contraste punto por punto. **Abierta**
- **[3. Las dos clasificaciones](#3-las-dos-clasificaciones)** — la de la integridad es semántica y la de las restricciones sintáctica; por eso no se corresponden. **Abierta**
- **[4. Lo acordado](#4-lo-acordado)** — las dos definiciones y el cruce de las dos clasificaciones
- **[5. Lo abierto — el debate](#5-lo-abierto--el-debate)** — siete preguntas con su evidencia
- **[6. Lo que este registro no cubre](#6-lo-que-este-registro-no-cubre)**
- **[7. Cómo se actualiza](#7-cómo-se-actualiza)**

---

## 1. Las definiciones de la fuente

Se transcriben tal como están en el apunte. **No son la última palabra: la §2 propone reemplazar las dos definiciones y la §3 discute las dos clasificaciones**, y para eso hace falta tenerlas a la vista.

> **Integridad** — «Integridad es un concepto qué nos permite garantizar que los datos son válidos y consistentes. En una base de datos relacional, la integridad significa que los datos cumplen reglas lógicas y de negocio, evitando errores, duplicados o inconsistencias.»
>
> **Restricción** (*constraint*) — «Las restricciones son las reglas físicas o lógicas que el SGBD (SQL Server, PostgreSQL, MySQL, etc.) impone a los datos para garantizar que se cumpla la integridad.»

Y los cuatro tipos que la fuente distingue:

| Tipo | Qué garantiza | Con qué se logra |
| --- | --- | --- |
| **De entidad** | Cada fila se identifica de forma única | `PRIMARY KEY`, `UNIQUE` |
| **Referencial** | Las relaciones entre tablas son válidas; no hay «datos huérfanos» | `FOREIGN KEY` |
| **De dominio** | Los valores respetan tipo, formato y rango | `CHECK`, `NOT NULL`, tipos, valores por defecto |
| **De negocio** | Reglas propias de la organización | *La fuente no le asigna mecanismo* |

---

## 2. La separación propuesta

*Sección abierta: es una propuesta de redefinición, no un acuerdo. Nace de la observación de que las dos definiciones de la §1 no se separan entre sí.*

### 2.1 ¿Qué problema tienen las dos definiciones de la fuente?

**Respuesta: que las dos dicen «reglas», y por eso no se separan.**

Poné las dos definiciones una al lado de la otra y mirá el sustantivo:

| Término | Lo que dice la fuente | Sustantivo |
| --- | --- | --- |
| **Integridad** | «los datos cumplen **reglas lógicas y de negocio**» | reglas |
| **Restricción** | «las **reglas físicas o lógicas** que el SGBD impone» | reglas |

Lo único que las distingue son los adjetivos —y ni siquiera son disjuntos: **«lógicas» está en las dos**—. Alguien que tenga adelante un enunciado cualquiera y quiera decidir si es «una integridad» o «una restricción» no tiene con qué.

Y hay un segundo problema, más silencioso: **la integridad no es una regla.** Es lo que pasa cuando las reglas se cumplen. Definirla como «reglas» borra la diferencia entre el enunciado y su efecto, que es justamente la diferencia que el tema necesita.

### 2.2 ¿Cuántas cosas hay realmente acá?

**Respuesta: tres, no dos. La regla, la restricción y la integridad.**

| | Qué es | Dónde vive | Cómo se lo interroga |
| --- | --- | --- | --- |
| **Regla de integridad** | Un enunciado sobre el dominio | En el análisis, en la cabeza del negocio | ¿Es verdadera sobre este estado? |
| **Restricción** | Una declaración en el esquema | En el `CREATE TABLE`, en el catálogo del motor | ¿Está declarada? ¿Con qué nombre? |
| **Integridad** | Una propiedad del estado de los datos | En las filas | ¿Se conserva? |

**La prueba de los verbos:** una regla se **enuncia**, una restricción se **declara**, la integridad se **conserva**. Ninguno de los tres verbos sirve para los otros dos —y ese es el indicio de que son tres categorías distintas y no tres nombres de lo mismo—.

### 2.3 ¿Cómo quedarían escritas las tres definiciones?

**Respuesta: cada una definida por su categoría, no por sus adjetivos.**

> **Regla de integridad** — un enunciado, en el lenguaje del negocio, que separa los estados posibles de la base de los imposibles. «Una persona nace en una sola localidad.» Es verdadera o falsa sobre un estado dado.
>
> **Restricción** — la forma declarada de una regla dentro del esquema. El motor la hace cumplir en cada operación: rechaza toda la que dejaría la base en un estado imposible. Tiene nombre, vive en el catálogo y se manifiesta como un mensaje de error.
>
> **Integridad** — la propiedad de un estado de la base: todo lo que contiene es posible según sus reglas. No se enuncia ni se declara; se conserva o se pierde.

Y la frase que las liga:

> **La regla dice qué; la restricción dice cómo; la integridad es lo que queda cuando la segunda alcanza a cubrir a la primera.**

| | |
| --- | --- |
| ✅ | «*Una persona nace en una sola localidad*» → regla |
| ✅ | «`CONSTRAINT FK_Personas_Localidades FOREIGN KEY …`» → restricción |
| ✅ | «Ninguna fila de `Personas` referencia una localidad inexistente» → integridad |
| ❌ | «La `FOREIGN KEY` es una integridad referencial» — confunde el medio con el resultado |

### 2.4 ¿Cómo se contrastan integridad y restricción, punto por punto?

**Respuesta: haciéndoles las mismas preguntas. Donde contestan distinto, ahí está el contraste.**

| Pregunta | **Integridad** | **Restricción** |
| --- | --- | --- |
| ¿Qué clase de cosa es? | Una propiedad del estado de los datos | Un objeto declarado en el esquema |
| ¿Cómo se dice en una oración? | «Los datos **están** íntegros» — adjetivo | «La tabla **tiene** cuatro restricciones» — sustantivo |
| ¿Dónde se la busca? | En las filas | En el `CREATE TABLE` y en el catálogo del motor |
| ¿Se cuenta? | No: no existen «tres integridades» en una base | Sí: se listan y se cuentan una por una |
| ¿Cómo se comprueba? | Corriendo una consulta que busque el estado imposible | Leyendo el esquema — o provocando el error |
| ¿Cómo aparece y cómo se va? | Se conserva o se pierde sola, al operar sobre los datos | Se agrega o se quita con `ALTER TABLE`, porque alguien lo decide |
| ¿Qué se nota si falta? | Hay datos imposibles conviviendo con los buenos | Nada — hasta que entra el dato que hubiera rechazado |
| ¿Cómo se clasifica? | Por **qué preserva**: entidad, referencial, dominio | Por **cuánto abarca su declaración**: columna, tabla |

Las dos primeras filas alcanzan para no confundirlas nunca más: **una es un adjetivo de los datos, la otra es un sustantivo del esquema.** Un adjetivo no se cuenta ni se borra con `ALTER TABLE`; un sustantivo no se «conserva».

La última fila es la que abre la §3, y es la que explica por qué las dos listas de la fuente no se corresponden.

> **La integridad se comprueba mirando las filas; la restricción se lee mirando el esquema.**

### 2.5 ¿La integridad garantiza que los datos sean correctos?

**Respuesta: no. Garantiza que sean posibles.**

En la tabla del apunte, la fila `(6, 'Arturo', 3)` dice que Arturo nació en Hernandarias. La base está íntegra: el `3` existe en `Localidades`. Si Arturo nació en La Paz, el dato es **falso** y la base sigue **íntegra**, porque ninguna restricción sabe dónde nació Arturo.

**La integridad es una propiedad del estado respecto de sus reglas, no respecto del mundo.**

Por qué importa corregirlo: la fuente promete que la integridad evita «errores», y ninguna restricción da eso. Lo que las restricciones eliminan es **una** clase de error —el imposible—. El error posible-pero-falso pasa entero, y es el más caro de encontrar porque la base no protesta. Es el mismo filo de la §5.1.

### 2.6 ¿Quién impone la restricción?

**Respuesta: la declara el diseñador; el SGBD solo la hace cumplir.**

La fuente dice «las reglas … que el SGBD impone a los datos». Leído literal, el motor decidiría las reglas. No decide ninguna: SQL Server aceptó sin protestar la tabla `Personas` del Ejemplo 1 **sin** `FOREIGN KEY`, y aceptó igual la del Ejemplo 2 **con** ella. Lo único que cambió entre un ejemplo y el otro fue lo que el autor escribió.

| | |
| --- | --- |
| ✅ | «El diseñador declara la restricción; el motor la hace cumplir en cada operación» |
| ❌ | «El SGBD impone reglas a los datos» |

**No es una sutileza de redacción.** Si el motor impusiera las reglas, no habría nada que decidir —y todo el debate de la §5 existe porque cada restricción es una decisión que alguien tomó y podría haber tomado distinto—.

### 2.7 ¿Qué queda por decidir de esta propuesta?

**Respuesta: si «regla de integridad» entra como término propio, que es lo que la hace funcionar y también lo que la aparta de la fuente.**

Todo lo anterior —y la §3 entera— depende de admitir un tercer término que el apunte no tiene. Lo que hay que sopesar:

| A favor | En contra |
| --- | --- |
| Sin él, «integridad» tiene que significar el enunciado y el resultado a la vez | Es un término más para sostener, y no aparece en la bibliografía del apunte |
| Permite decir «regla sin restricción que la cubra», que es la deuda real | El alumno lo va a leer en un solo lado |
| Hace evaluable el diseño: se cuentan reglas y se cuentan coberturas | — |

**Lo que no cambia en ningún caso:** la distinción de la §2.2 entre lo que se declara y lo que se conserva. Aunque no adoptemos el tercer término, integridad y restricción tienen que dejar de definirse las dos como «reglas».

---

---

## 3. Las dos clasificaciones

*Sección abierta, como la §2.* La fuente clasifica dos veces: la integridad en cuatro tipos y las restricciones en dos. Puestas una debajo de la otra parecen dos niveles de detalle del mismo tema. No lo son, y de ahí sale buena parte de la confusión.

### 3.1 ¿Por qué las dos clasificaciones no se dejan comparar?

**Respuesta: porque no responden la misma pregunta. Una pregunta para qué; la otra, dónde se escribe.**

| Clasificación | Sus ítems | La pregunta que responde | Qué clase de eje es |
| --- | --- | --- | --- |
| **De la integridad** | de entidad, referencial, de dominio, de negocio | ¿Qué se quiere que sea verdad? | **Semántico** — habla del modelo |
| **De las restricciones** | de columna, de tabla | ¿Sobre cuántas columnas se escribe la declaración? | **Sintáctico** — habla del texto del `CREATE TABLE` |

**Entre «integridad referencial» y «restricción de tabla» no hay ni inclusión ni oposición.** Una `FOREIGN KEY` es las dos cosas a la vez, y saberlo no informa nada sobre ella.

El síntoma de que los ejes están cruzados es que las listas tienen cuatro ítems y dos, y no hay manera de emparejarlos. **No es que a la segunda le falten ítems: es que cuenta otra cosa.**

### 3.2 ¿Qué prueba que el eje columna/tabla es sintáctico?

**Respuesta: que la misma restricción figura en las dos listas, y lo dice la propia fuente.**

|  | En «restricciones de columna» | En «restricciones de tabla» |
| --- | --- | --- |
| `UNIQUE` | «(cuando se aplica a una sola columna)» | «Cuando es sobre varias columnas» |
| `CHECK` | «Valida una condición lógica» | «Cuando involucra varias columnas» |

Un `UNIQUE` no cambia de naturaleza según dónde se lo escriba: **lo que cambia es cuántas columnas nombra**. Por eso el eje no clasifica restricciones — clasifica **declaraciones**.

Y hay un tercer caso, que la fuente contradice sin notarlo: ubica `FOREIGN KEY` solo entre las restricciones de tabla, pero su propio Ejemplo 2 la declara sobre la columna —`Id_LugarNacimiento INT REFERENCES Localidades(Id)`— y muestra al lado la forma de tabla como equivalente ([Hallazgos-01-Fuente.md](Hallazgos-01-Fuente.md) §1.7).

| | |
| --- | --- |
| ✅ | «`UNIQUE` se puede declarar a nivel de columna o de tabla, según cuántas nombre» |
| ⚠️ | «`UNIQUE` es una restricción de columna» — cierto en el caso de una sola, falso en general |
| ❌ | «Hay restricciones de columna y restricciones de tabla, y cada una es de una clase» |

De las seis restricciones que el apunte nombra, **solo dos no tienen forma de tabla**: `NOT NULL` y `DEFAULT`, porque las dos hablan siempre de una columna nombrada y de ninguna otra.

**Para qué sirve igual el eje sintáctico:** para escribir. Una restricción que nombra dos columnas no entra adentro de la declaración de ninguna de las dos y hay que ponerla aparte. Es una regla de escritura útil —**no es una taxonomía del tema**.

### 3.3 ¿Por qué «físicas o lógicas» no clasifica nada?

**Respuesta: porque ninguna restricción es física, y el eje que sí ordena es otro —sobre qué habla—.**

Una `PRIMARY KEY` crea un índice, y el índice sí ocupa disco. Pero eso es una consecuencia de cómo el motor la implementa, no lo que la restricción dice. **Ninguna restricción habla de bytes: todas hablan de valores.** El par «físicas / lógicas» no separa dos grupos, porque el primero está vacío.

El eje que sí las ordena —y que además predice cuánto cuesta verificarlas— es qué necesita mirar el motor para decidir si acepta:

| Sobre qué habla | Qué mira el motor | Restricciones |
| --- | --- | --- |
| **Un valor** | La columna que entra | Tipo de dato, `NOT NULL`, `DEFAULT`, `CHECK` de una columna |
| **Una fila entera** | Esa fila, completa | `CHECK` de varias columnas |
| **El conjunto de filas de la tabla** | Todas las demás filas | `PRIMARY KEY`, `UNIQUE` |
| **La relación con otra tabla** | Otra tabla | `FOREIGN KEY` |

Esta tabla hace un trabajo que la de la fuente no hace: **explica por qué el catálogo de restricciones termina donde termina.** Todo lo que necesite mirar algo que no está en esas cuatro filas —el estado anterior, otra transacción, el reloj— no se puede declarar, y por eso cae en la brecha de la §3.5.

### 3.4 ¿Cómo se conectan entonces las dos listas?

**Respuesta: por el propósito de cada restricción —que la propia fuente ya anota—, no por su alcance.**

El puente está escrito en la tabla de restricciones de tabla del apunte: «`PRIMARY KEY` — asegura la integridad de entidad», «`FOREIGN KEY` — asegura la integridad referencial». Extendido a todo el catálogo, y cruzado con el eje de la §3.3:

| Integridad que asegura | Qué necesita mirar el motor | Restricciones | Alcance de la declaración |
| --- | --- | --- | --- |
| **De dominio** | El valor que entra, o la fila entera | Tipo de dato, `NOT NULL`, `DEFAULT`, `CHECK` | Columna; de tabla si nombra varias |
| **De entidad** | Las demás filas de la tabla | `PRIMARY KEY`, `UNIQUE` | Tabla; de columna si es una sola |
| **Referencial** | Otra tabla | `FOREIGN KEY` | Tabla; de columna con `REFERENCES` en línea |
| **De negocio** | Lo que no entra en las tres de arriba | *Ninguna declarativa* | — |

Esta tabla dice tres cosas que ninguna de las dos listas de la fuente dice sola:

1. **La columna del medio es la que ordena.** El propósito no se elige: se deduce de qué tiene que mirar el motor para decidir.
2. **La última columna cambia dentro de una misma fila.** Es la confirmación de la §3.2 —el alcance no es una propiedad de la restricción, es una propiedad de cómo se la escribió—.
3. **La última fila está vacía del lado de las restricciones.** Ese vacío no es un olvido de la tabla: es la brecha, y es el tema de la §3.5.

### 3.5 ¿Y entonces qué es la «integridad de negocio»?

**Respuesta: no es un cuarto tipo. Es el nombre de lo que quedó sin restricción que lo cubra.**

La clasificación de la fuente pone cuatro ítems en una misma lista, pero responden a dos preguntas distintas:

| Tipo | Responde a |
| --- | --- |
| De entidad, referencial, de dominio | ¿**Sobre qué** habla la regla? |
| De negocio | ¿**De dónde viene** la regla? |

Son ejes cruzados, y por eso se solapan. Las tres primeras **también** vienen del negocio: que una persona nazca en una sola localidad es una regla de negocio, y se declara con una `FOREIGN KEY`. Y al revés, el ejemplo que la fuente da para el cuarto tipo —«que la fecha de entrega no sea anterior a la fecha de pedido»— es integridad de dominio sobre una fila, y se declara con el `CHECK` de varias columnas que la propia fuente clasifica en su §2 y no ejemplifica nunca ([Hallazgos-01-Fuente.md](Hallazgos-01-Fuente.md) §1.5).

Lo que sí existe y merece un nombre propio es **la brecha**: las reglas del negocio que ninguna restricción declarativa alcanza a cubrir. Por ejemplo, «un cliente no puede tener más de tres pedidos abiertos»: habla de un conjunto de filas que la fila entrante no conoce, y no hay `CHECK` que lo exprese.

| | |
| --- | --- |
| ✅ | «Esta regla está cubierta por la restricción `CK_Pedido_Fechas`» |
| ⚠️ | «Esta regla es de negocio» — cierto, pero no dice si está cubierta ni quién la hace cumplir |
| ❌ | «La integridad de negocio es otro tipo de integridad» |

**La brecha se administra, no se clasifica:** por cada regla enunciada, o hay una restricción que la cubre, o hay una razón declarada de por qué no y el nombre de quién la hace cumplir en su lugar. Eso es lo que está en discusión en la §5.7.

## 4. Lo acordado

Dos acuerdos, ya volcados al documento [Conceptos-Integridad-y-Restricciones.md](Conceptos-Integridad-y-Restricciones.md). Ninguna de las preguntas de la §5 fue decidida todavía.

### 4.1 La restricción es la regla que se debe cumplir; la integridad es el dato que la cumple

**Decidido el 2026-09-05, a partir de la §2.** La formulación es del debate y se adopta con sus palabras. Resuelve el solapamiento que abría la §2.1: las dos definiciones de la fuente decían «reglas», y ahora solo una lo dice —la otra pasó a ser el dato que la cumple—. De ahí se sigue el par **medio → resultado**.

**Qué se descartó:** definir la integridad como «reglas lógicas y de negocio» (la fuente) y definirla como «una propiedad del estado» (mi primera propuesta, §2.3). La segunda es correcta pero no contrasta: no dice contra qué se define.

**Qué queda en pie de la §2:** que la integridad garantiza que los datos sean **posibles**, no ciertos (§2.5), y que la restricción la declara el diseñador, no el SGBD (§2.6). Sigue sin decidirse si «regla de integridad» entra como tercer término (§2.7).

### 4.2 Las dos clasificaciones son dos miradas, y se cruzan en una grilla

**Decidido el 2026-09-05, a partir de la §3.** La integridad se clasifica por **lo que preserva** —entidad, referencial, dominio—; las restricciones, por **dónde se escriben** —columna, tabla—. No se corresponden, y la prueba es que la grilla de tres por dos tiene **las seis celdas ocupadas**: cualquiera de los tres tipos de integridad se puede escribir de las dos maneras.

**El caso que lo hace evidente**, aportado en el debate: una `FOREIGN KEY` de tabla asegura la integridad referencial y decirlo tiene sentido; un `UNIQUE`, que está en el mismo grupo de restricciones, no tiene nada que ver con esa integridad —asegura la de entidad—. **La fila es homogénea en propósito; la columna solo comparte el lugar donde se escribe.**

**Qué se descartó:** tratar «de negocio» como un cuarto tipo **en el mismo eje** que los otros tres. Las tres primeras también vienen del negocio; el cuarto grupo lo definen por exclusión tanto Microsoft —«business rules that do not fall into one of the other integrity categories»— como Elmasri & Navathe —«cannot be expressed by the model per se»—.

**Corregido el 2026-09-05 contra la bibliografía** ([Respaldo-Bibliografico.md](Respaldo-Bibliografico.md) §4): habíamos dicho que el cuarto tipo era un error del apunte —no lo es, la clasificación en cuatro es de Microsoft palabra por palabra—; y habíamos clasificado «la fecha de entrega no puede ser anterior a la de pedido» como integridad de dominio —no lo es: dominio se define por atributo, y esa condición cruza dos columnas—.

**Qué queda abierto:** si en el aula conviene **reemplazar** el eje columna/tabla por el del propósito, o conservarlo como lo que es —una regla de escritura, con la consecuencia de la §4.4 del documento: la forma de columna no deja poner nombre a la restricción, y el error se vuelve ilegible—.

Formato con el que entrará cada acuerdo, cuando lo haya:

```markdown
### 4.n <La afirmación acordada, en una línea>

**Decidido el <fecha>, a partir de la §5.<n>.** <El argumento que la sostuvo y qué alternativa se descartó.>
```

---

## 5. Lo abierto — el debate

Siete preguntas. Cada una nace de algo que **está** en el material, no de una hipótesis: donde hay una cita, es literal.

### 5.1 ¿La `PRIMARY KEY` alcanza para evitar duplicados?

**Mi posición: no, y los propios datos del apunte lo muestran.**

El apunte dice que la integridad de entidad «evita duplicados y asegura identificación inequívoca». Ahora mirá la tabla `Personas` de los tres ejemplos:

```sql
VALUES(1, 'Daniela', 1), (2, 'Andrés', 2), (3, 'Daniela', 2),
      (4, 'Andrés', 1),  (5, 'Armando', 1), (6, 'Arturo', 3)
```

Hay dos `Daniela` y dos `Andrés`. La `PRIMARY KEY` está cumpliendo perfectamente y aun así **no sabemos si son cuatro personas o dos personas cargadas dos veces**.

| | |
| --- | --- |
| ✅ | «La `PRIMARY KEY` garantiza que no haya dos *filas* con la misma identidad» |
| ⚠️ | «La `PRIMARY KEY` evita duplicados» — cierto en la tabla, falso en el mundo |
| ❌ | «Con `PRIMARY KEY` ya está resuelta la identificación de la persona» |

**Lo que pongo en discusión:** si «duplicado» significa *dos filas con la misma clave* o *dos filas que hablan de la misma cosa real*, y en el segundo caso qué restricción lo evita —¿un `UNIQUE` sobre qué columnas? ¿DNI? ¿nombre + fecha de nacimiento?—. Esa elección es lo que el modelo relacional llama *clave natural*, y el apunte no la nombra.

### 5.2 ¿`Id INT PRIMARY KEY` o `Id INT IDENTITY(1,1) PRIMARY KEY`?

**Mi posición: la pregunta no es cuál es mejor, sino quién genera el identificador.**

En los tres ejemplos, la clave se escribe a mano en el `INSERT`. Eso permite ver los valores —que es exactamente lo que el apunte quería, «trabajar con valores para entender la naturaleza del tema»— pero deja fuera la pregunta de producción: **con varios usuarios insertando a la vez, ¿quién decide el próximo `Id`?**

| Quién genera | Consecuencia |
| --- | --- |
| La aplicación | Hay que resolver la concurrencia afuera del motor |
| `IDENTITY` / `SEQUENCE` | La resuelve el motor, y los valores dejan de ser predecibles |

**Lo que pongo en discusión:** si conviene que el ejemplo didáctico mienta sobre esto para poder mostrarlo, o si la mentira después cuesta caro.

### 5.3 ¿Una clave foránea que admite `NULL` viola la integridad referencial?

**Mi posición: no la viola, y esa es la parte interesante.**

El Ejemplo 2 declara la `FOREIGN KEY` sobre una columna que sigue siendo nulable. Recién el Ejemplo 3 le agrega `NOT NULL`. **Entre un ejemplo y el otro cambia el significado de la relación**, y el apunte no lo comenta:

| Declaración | Qué dice del negocio |
| --- | --- |
| `Id_LugarNacimiento INT REFERENCES Localidades(Id)` | Toda persona *puede* tener lugar de nacimiento; si lo tiene, tiene que existir |
| `Id_LugarNacimiento INT NOT NULL REFERENCES Localidades(Id)` | Toda persona *debe* tener lugar de nacimiento |

Un `NULL` ahí **no es un huérfano**: es un «no sé». Un huérfano sería un `100` que no está en `Localidades` —el tercer intento del Ejemplo 4—.

**Lo que pongo en discusión:** si «desconocido» y «no aplica» merecen distinguirse en la base, y cómo, porque `NULL` los confunde en uno solo.

### 5.4 ¿Qué debe pasar cuando se borra una localidad que tiene personas nacidas ahí?

**Mi posición: es la decisión más importante del tema y el apunte no la toca.**

Ninguna de las dos formas que muestra el Ejemplo 2 declara `ON DELETE` ni `ON UPDATE`. SQL Server aplica entonces su comportamiento por defecto —`NO ACTION`: el `DELETE` sobre `Localidades` falla—. Es una decisión tomada por omisión, y hay al menos tres alternativas:

| Opción | Qué significa para el negocio |
| --- | --- |
| `NO ACTION` *(el que queda por defecto)* | «No se puede borrar una localidad usada» |
| `ON DELETE CASCADE` | «Borrar la localidad borra a las personas nacidas ahí» |
| `ON DELETE SET NULL` | «Se pierde el dato del lugar, la persona queda» |

**El `CASCADE` sobre este caso es absurdo** —nadie deja de existir porque se borre una localidad— y por eso el ejemplo sirve: muestra que la opción correcta se deduce del significado de la relación, no de una preferencia técnica.

**Lo que pongo en discusión:** si conviene declarar `NO ACTION` explícitamente aunque sea el valor por defecto, para que se lea que fue una decisión.

### 5.5 ¿Conviene nombrar las restricciones o dejar que el motor las nombre?

**Mi posición: nombrarlas, y la prueba está en el mensaje de error del propio apunte.**

El Ejemplo 4 registra literalmente lo que devolvió el motor al insertar un `Id` repetido:

```
Violation of PRIMARY KEY constraint 'PK__Personas__3214EC078DFC3B8A'.
Cannot insert duplicate key in object 'dbo.Personas'.
```

Comparalo con el mensaje del tercer intento, donde la restricción sí tenía nombre puesto por el autor:

```
The INSERT statement conflicted with the FOREIGN KEY constraint "FK_Localidades_PERSONAS".
```

`PK__Personas__3214EC078DFC3B8A` **cambia en cada base creada**: el sufijo es generado. Un mensaje de error de producción con ese nombre no se puede buscar en el código ni comparar entre entornos.

| | |
| --- | --- |
| ✅ | `CONSTRAINT PK_Personas PRIMARY KEY (Id)` — el error se lee y se busca |
| ⚠️ | `Id INT PRIMARY KEY` — anda, pero el error queda ilegible |

**Lo que pongo en discusión:** el nombre `FK_Localidades_PERSONAS` del Ejemplo 3 y el `FK_Personas_Localidades` del Ejemplo 2 son la misma relación con el orden invertido. Si vamos a nombrarlas, hace falta una convención: **¿el nombre va de la tabla que referencia hacia la referenciada, o al revés?**

### 5.6 ¿`VARCHAR(100)` es una restricción de dominio suficiente para un nombre?

**Mi posición: el propio apunte se tropieza con esto sin notarlo.**

La localidad 1 aparece como `'Parana'` en los Ejemplos 1 y 2, y como `'Paraná'` en el Ejemplo 3 (ver [Hallazgos-01-Fuente.md](Hallazgos-01-Fuente.md) §1.4). Las dos entran en `VARCHAR(100)` sin protestar. El tipo de dato acepta cualquier cadena de hasta cien caracteres: **no sabe qué es un nombre de localidad.**

Y hay una capa más, que el apunte no menciona: si `Parana` y `Paraná` se consideran o no el mismo valor lo decide la **colación** de la columna, no el tipo. Con una colación acento-insensible, un `UNIQUE` sobre `Nombre` rechazaría la segunda; con una acento-sensible, la aceptaría.

**Lo que pongo en discusión:** cuánto del dominio se puede realmente declarar —`VARCHAR(100) NOT NULL` más un `CHECK (LEN(TRIM(Nombre)) > 0)`, por ejemplo— y dónde empieza lo que solo puede validar la aplicación o la carga de datos.

### 5.7 ¿La regla de negocio vive en la base o en la aplicación?

**Mi posición: es la única de las siete preguntas que no tiene respuesta única, y hay que decidirla por caso.**

El apunte clasifica la integridad de negocio como un cuarto tipo —«reglas específicas de la organización»— y le da un ejemplo: «que la fecha de entrega no sea anterior a la fecha de pedido». Pero, a diferencia de los otros tres tipos, **no le asigna ningún mecanismo**. Y sin embargo esa regla es expresable como restricción:

```sql
CONSTRAINT CK_Pedido_Fechas CHECK (FechaEntrega >= FechaPedido)
```

Es exactamente el «`CHECK` cuando involucra varias columnas» que el apunte clasifica en su §2 y no ejemplifica nunca ([Hallazgos-01-Fuente.md](Hallazgos-01-Fuente.md) §1.5).

La tensión, planteada honestamente:

| A favor de ponerla en la base | A favor de ponerla en la aplicación |
| --- | --- |
| Se cumple **siempre**, venga el dato de donde venga | Da un mensaje de error que el usuario entiende |
| Nadie la puede saltear con un `INSERT` manual | Se cambia sin tocar el esquema ni migrar datos |
| Es declarativa: se lee en el `CREATE TABLE` | Puede depender de datos que la fila no tiene |

**El argumento que me parece decisivo:** una regla en la aplicación protege *a los que entran por la aplicación*. La base tiene más de una puerta.

**Lo que pongo en discusión:** si la respuesta es «las dos» —validar en la aplicación por el mensaje, restringir en la base por la garantía— o si duplicar la regla en dos lugares es el camino seguro a que las dos copias diverjan.

---

## 6. Lo que este registro no cubre

| Ausencia | Tipo | Qué corresponde |
| --- | --- | --- |
| Composición y agregación | **Pendiente** | El temario los pone en el Tema 2 y el apunte no los trata; probablemente estén en las guías 2.1 y 2.2 |
| Mapeo de herencia | No aplica *todavía* | Es el Tema 3, y en `Temas.md` está vacío |
| Índices | Otra herramienta | `PRIMARY KEY` y `UNIQUE` crean índices, pero el rendimiento es un tema aparte del de integridad |
| *Triggers* y procedimientos como vía de integridad | **Pendiente** | Es la continuación natural de la §5.7 y de la brecha de la §3.5, y no está abierta acá |
| Que los scripts del apunte corran | **Pendiente** | No se ejecutó ninguno; dos de ellos no compilarían tal como están ([Hallazgos-01-Fuente.md](Hallazgos-01-Fuente.md) §1.1 y §1.2) |

---

## 7. Cómo se actualiza

Este archivo se pasa en limpio, no se apila. La mecánica:

1. **Una pregunta decidida se mueve** de la §5 a la §4, con la fecha y el argumento. No se deja la versión vieja abajo.
2. **Una pregunta nueva entra en la §5** con la misma forma: posición, evidencia literal, y qué queda en discusión.
3. **Toda afirmación nueva declara su registro** —verificado / razonado / supuesto—, como pide [el estilo](../../../../../../IA/PROMPTs/IA.Prompts/Base/Estilo-Redaccion-Explicativo.md) en su §5.1.
4. **Las extracciones de fuentes nuevas van a archivos aparte** de esta misma carpeta, y este registro las enlaza.

---

> **Una restricción no es una traba: es la única parte de la regla que se cumple sola.**
