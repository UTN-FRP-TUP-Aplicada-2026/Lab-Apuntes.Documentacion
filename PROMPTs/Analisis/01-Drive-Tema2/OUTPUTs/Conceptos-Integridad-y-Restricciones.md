# Integridad y restricciones — dos miradas sobre el mismo esquema

> **Qué es esto**: la presentación en limpio de los dos conceptos del Tema 2, con sus definiciones y sus clasificaciones. Sale de la discusión registrada en [Registro.md](Registro.md) y del apunte extractado en [Extraccion-01-Integridad-y-Restricciones.md](Extraccion-01-Integridad-y-Restricciones.md).
>
> **Overview**: la tesis del documento es que integridad y restricción no son dos niveles de detalle del mismo tema sino **dos miradas sobre el mismo esquema**, y que por eso cada una trae su propia clasificación y las dos clasificaciones no se superponen. La integridad se clasifica por **lo que preserva** —entidad, referencial, dominio—; las restricciones, por **dónde se escriben** —columna, tabla—. La §5 cruza las dos en una sola tabla, y ahí se ve por qué una `FOREIGN KEY` y un `UNIQUE` viven en el mismo grupo de restricciones sin tener nada que ver entre sí.
>
> **Ejemplo de referencia**: las tablas `Localidades` y `Personas` del apunte, sin cambios.
>
> **Lo que se afirma y con qué**: cada afirmación tiene su fuente en [Respaldo-Bibliografico.md](Respaldo-Bibliografico.md), que dice cuáles están sostenidas por la bibliografía —Elmasri & Navathe, Microsoft Learn, PostgreSQL— y cuáles son razonamiento propio. Los fragmentos de SQL y los mensajes de error son **citas del apunte**. Nada de esto se ejecutó contra un SQL Server (§7).

---

## Índice

- **[1. Las dos definiciones](#1-las-dos-definiciones)** — se declaran, no se discuten
- **[2. Por qué son dos miradas y no dos niveles](#2-por-qué-son-dos-miradas-y-no-dos-niveles)** — el contraste punto por punto y la asimetría entre las dos
- **[3. Clasificación de la integridad — por lo que preserva](#3-clasificación-de-la-integridad--por-lo-que-preserva)** — entidad, referencial, dominio
- **[4. Clasificación de las restricciones — por dónde se escriben](#4-clasificación-de-las-restricciones--por-dónde-se-escriben)** — columna, tabla
- **[5. Las dos clasificaciones, cruzadas](#5-las-dos-clasificaciones-cruzadas)** — la tabla que hace visible que son ejes independientes. **Es el centro del documento**
- **[6. El esquema entero, anotado](#6-el-esquema-entero-anotado)** — cada restricción con su celda
- **[7. Lo que este documento no cubre](#7-lo-que-este-documento-no-cubre)**
- **[8. El criterio, en una línea](#8-el-criterio-en-una-línea)**

El respaldo de cada afirmación —quién la sostiene y qué es razonamiento propio— está en **[Respaldo-Bibliografico.md](Respaldo-Bibliografico.md)**.

---

## 1. Las dos definiciones

Se declaran. Lo que se discute —por qué así y no de otra manera— está en la §2.

> **Restricción** — la **regla que se debe cumplir**, escrita en el esquema. El motor la hace cumplir en cada operación: rechaza toda la que dejaría datos que no la cumplen.
>
> **Integridad** — el **dato que cumple esa regla**. Es el estado en que queda la base cuando las restricciones se cumplen.

La relación entre las dos, que es lo que hay que retener:

> **La restricción es el medio; la integridad es el resultado.**

Y una tercera declaración, que evita el malentendido más caro:

> **Integridad no es verdad.** Un dato íntegro es un dato **posible** según las reglas del esquema. Puede ser falso y la base no lo va a notar. En la fila `(6, 'Arturo', 3)` la base garantiza que la localidad `3` existe; no garantiza que Arturo haya nacido ahí.

---

## 2. Por qué son dos miradas y no dos niveles

### 2.1 ¿Qué mira cada una?

**Respuesta: la restricción se lee en el esquema; la integridad se comprueba en las filas.**

Es la diferencia práctica de la que se derivan todas las demás. Para saber qué restricciones tiene una base, se abre el `CREATE TABLE` —o el catálogo del motor— y se leen. Para saber si la base está íntegra, no alcanza con leer nada: hay que ir a los datos.

| Pregunta | **Restricción** | **Integridad** |
| --- | --- | --- |
| ¿Qué clase de cosa es? | Un objeto declarado en el esquema | Una propiedad del estado de los datos |
| ¿Cómo se dice en una oración? | «La tabla **tiene** cuatro restricciones» — sustantivo | «Los datos **están** íntegros» — adjetivo |
| ¿Dónde se la busca? | En el `CREATE TABLE`, en el catálogo | En las filas |
| ¿Se cuenta? | Sí: se listan una por una | No: no existen «tres integridades» en una base |
| ¿Cómo aparece y cómo se va? | Alguien la agrega o la quita con `ALTER TABLE` | Se conserva o se pierde sola, al operar |
| ¿Qué se nota si falta? | Nada — hasta que entra el dato que hubiera rechazado | Hay datos imposibles conviviendo con los buenos |
| ¿Cómo se clasifica? | Por **dónde se escribe**: columna, tabla | Por **qué preserva**: entidad, referencial, dominio |

Las dos primeras filas alcanzan para no confundirlas nunca más: **un sustantivo no se «conserva»; un adjetivo no se borra con `ALTER TABLE`.**

### 2.2 ¿Se puede tener una sin la otra?

**Respuesta: integridad sin restricción, sí. Restricción sin integridad, no.**

La asimetría es lo que justifica declarar restricciones, y conviene verla con el ejemplo del apunte. En el **Ejemplo 1** la columna `Id_LugarNacimiento` no tiene `FOREIGN KEY`, y sin embargo los seis valores insertados —`1, 2, 2, 1, 1, 3`— existen todos en `Localidades`. **Esa base está referencialmente íntegra sin ninguna restricción que lo asegure.**

| | |
| --- | --- |
| ✅ | «El Ejemplo 1 está íntegro por cómo se cargaron los datos, no porque algo lo impida» |
| ❌ | «El Ejemplo 1 no tiene integridad referencial porque no tiene `FOREIGN KEY`» |

Lo que cambia el Ejemplo 2 al agregar la `FOREIGN KEY` no es el estado de la base —los datos son los mismos—: es que **a partir de ahí ese estado no se puede perder**. La restricción no crea la integridad: **la vuelve inevitable**.

Y al revés no funciona: no existe una base con la restricción declarada y datos que no la cumplan, porque el motor no los habría dejado entrar.

### 2.3 ¿Por qué cada una trae su propia clasificación?

**Respuesta: porque cada mirada solo puede clasificar por lo que ve.**

La mirada de la integridad ve datos, y lo único que puede preguntarles es **qué propiedad conservan**: identidad, referencia, valor. De ahí salen los tres tipos de la §3.

La mirada de la restricción ve texto en el `CREATE TABLE`, y lo único que puede preguntarle es **dónde está escrito**: adentro de una columna o afuera de todas. De ahí salen los dos tipos de la §4.

**Ninguna de las dos clasificaciones se equivocó de criterio: cada una usó el que tenía a mano.** El error es esperar que se correspondan —y es lo que la §5 desarma—.

---

## 3. Clasificación de la integridad — por lo que preserva

Tres tipos. El criterio es **qué propiedad de los datos queda garantizada**.

### 3.1 Integridad de entidad

**Cada fila se identifica de forma única, y esa identificación nunca falta.**

Son dos exigencias, y conviene verlas separadas porque la bibliografía las separa. La **unicidad** viene de que la clave es clave: «no two tuples in any valid relation state will have the same value» (Elmasri & Navathe, *key constraint*). La **integridad de entidad** propiamente dicha es solo la segunda mitad: «the primary key attributes … cannot have null values in any tuple» (Elmasri & Navathe, *entity integrity*).

La `PRIMARY KEY` hace cumplir las dos juntas, y por eso el apunte las presenta como una. **`UNIQUE` hace cumplir solo la primera** — y de ahí que admita un `NULL` donde la primaria no ([Respaldo-Bibliografico.md](Respaldo-Bibliografico.md) §4.1).

```sql
CREATE TABLE Localidades
(
    Id INT PRIMARY KEY,
    Nombre VARCHAR(100)
)
```

Qué pasa cuando falta, con el mensaje literal que registra el apunte al intentar repetir el `Id 1`:

```
Violation of PRIMARY KEY constraint 'PK__Personas__3214EC078DFC3B8A'.
Cannot insert duplicate key in object 'dbo.Personas'. The duplicate key value is (1).
```

**Dónde termina:** garantiza que no haya dos **filas** con la misma identidad, no que no haya dos filas hablando de la misma **cosa**. En `Personas` hay dos `'Daniela'` y dos `'Andrés'` con la `PRIMARY KEY` intacta, y la base no puede decir si son cuatro personas o dos cargadas dos veces.

### 3.2 Integridad referencial

**Toda referencia apunta a algo que existe.**

No hay «datos huérfanos»: si una fila nombra a otra tabla, la fila nombrada está.

```sql
CREATE TABLE Personas
(
    Id INT PRIMARY KEY,
    Nombre VARCHAR(100),
    Id_LugarNacimiento INT,
    CONSTRAINT FK_Personas_Localidades FOREIGN KEY (Id_LugarNacimiento)
        REFERENCES Localidades(Id)
)
```

Qué pasa cuando falta, al intentar insertar una localidad `100` inexistente:

```
The INSERT statement conflicted with the FOREIGN KEY constraint "FK_Localidades_PERSONAS".
The conflict occurred in database "Ejemplo_Integridad_DB", table "dbo.Localidades", column 'Id'.
```

**Dónde termina:** un `NULL` en `Id_LugarNacimiento` **no es un huérfano** —es un «no sé»— y la `FOREIGN KEY` lo acepta. Que la referencia además sea obligatoria es otra decisión, y se toma con `NOT NULL`, que es integridad de dominio (§3.3).

### 3.3 Integridad de dominio

**Cada valor está dentro de lo que su columna admite.**

El tipo, el formato, el rango, la condición. Y también la obligatoriedad: «vacío» no es un valor admitido.

```sql
CREATE TABLE Personas
(
    Id INT PRIMARY KEY,
    Nombre VARCHAR(100) NOT NULL,
    Id_LugarNacimiento INT NOT NULL,
    CONSTRAINT FK_Localidades_PERSONAS FOREIGN KEY (Id_LugarNacimiento)
        REFERENCES Localidades(Id)
);
```

Qué pasa cuando falta, al intentar insertar una persona sin nombre:

```
Cannot insert the value NULL into column 'Nombre', table 'Ejemplo_Integridad_DB.dbo.Personas';
column does not allow nulls. INSERT fails.
```

**Dónde termina:** `VARCHAR(100)` acepta cualquier cadena de hasta cien caracteres. Acepta `'Parana'` y acepta `'Paraná'` —las dos aparecen en el apunte para la misma localidad—, y acepta `'   '`. El tipo de dato **no sabe qué es un nombre de localidad**: define el continente, no el contenido.

### 3.4 ¿Y las reglas de negocio?

**Respuesta: son un grupo aparte, pero definido por exclusión — no por su objeto, como los otros tres.**

La bibliografía lo reconoce con dos nombres. Microsoft lo llama *user-defined integrity*: «lets you define specific business rules that **do not fall into one of the other integrity categories**». Elmasri & Navathe lo llaman *semantic integrity constraints*: «based on application semantics and **cannot be expressed by the model per se**».

Las dos definiciones dicen lo que el grupo **no** es. Los otros tres se definen por qué preservan; este, por dónde no entra. **Por eso no está en el mismo eje** — y por eso no sirve como respuesta a «¿de qué tipo es esta regla?».

Y hay que decir lo que sí separa: **los tres primeros tipos también vienen del negocio.** «Una persona nace en una sola localidad» es una regla del negocio y es integridad referencial. El origen no distingue nada; lo que distingue es si se puede declarar.

Dos casos, para ver dónde está el corte:

```sql
-- Se puede declarar: un CHECK de dos columnas.
CONSTRAINT CK_Pedido_Fechas CHECK (FechaEntrega >= FechaPedido)
```

Esa —el ejemplo que da el apunte— **no es integridad de dominio**, aunque se le parezca: dominio se define por atributo, «la validez de las entradas de una columna determinada» (Microsoft), «every value in a tuple must be from the domain of **its attribute**» (Elmasri & Navathe). Una condición entre dos columnas no entra ahí. **El esquema de tres tipos no tiene casillero para ella**, y forzarla adentro de «dominio» fue un error nuestro, corregido en [Respaldo-Bibliografico.md](Respaldo-Bibliografico.md) §4.2.

El segundo caso ya no se puede declarar: «un cliente no puede tener más de tres pedidos abiertos» habla de un conjunto de filas que la fila entrante no conoce, y no hay `CHECK` que lo exprese. Ahí sí estamos en lo que Elmasri & Navathe llaman semántico, y lo hacen cumplir un *trigger*, una `ASSERTION` o la aplicación.

| | |
| --- | --- |
| ✅ | «Esta regla no es de ninguno de los tres tipos, y está cubierta por `CK_Pedido_Fechas`» |
| ✅ | «Esta regla no se puede declarar; la hace cumplir el procedimiento X» |
| ⚠️ | «Esta regla es de negocio» — no dice ni de qué tipo es ni si está cubierta |
| ❌ | «La integridad de negocio es un cuarto tipo, al lado de los otros tres» |

**Eso no se clasifica: se administra.** Por cada regla enunciada, o hay una restricción que la cubre, o hay una razón declarada de por qué no y el nombre de quién la hace cumplir en su lugar.

---

## 4. Clasificación de las restricciones — por dónde se escriben

Dos tipos. El criterio es **dónde va la declaración dentro del `CREATE TABLE`**, que depende de cuántas columnas nombra.

### 4.1 Restricción de columna

**Se escribe adentro de la declaración de una columna, y solo habla de ella.**

```sql
Id                 INT PRIMARY KEY,
Nombre             VARCHAR(100) NOT NULL,
Id_LugarNacimiento INT REFERENCES Localidades(Id)
```

Las que pueden ir acá: `NOT NULL`, `DEFAULT`, `CHECK` de una columna, `UNIQUE` de una columna, `PRIMARY KEY` de una columna, y `FOREIGN KEY` en su forma `REFERENCES`.

### 4.2 Restricción de tabla

**Se escribe aparte de las columnas, después de todas, y puede nombrar a varias.**

```sql
CONSTRAINT FK_Personas_Localidades FOREIGN KEY (Id_LugarNacimiento)
    REFERENCES Localidades(Id),
CONSTRAINT PK_Detalle PRIMARY KEY (Id_Pedido, Id_Producto),
CONSTRAINT CK_Pedido_Fechas CHECK (FechaEntrega >= FechaPedido)
```

Las que pueden ir acá: `PRIMARY KEY`, `UNIQUE`, `FOREIGN KEY` y `CHECK`.

### 4.3 ¿Por qué casi todas aparecen en las dos listas?

**Respuesta: porque el tipo no es una propiedad de la restricción sino de cómo se la escribió.**

El propio apunte lo dice dos veces, sin sacar la conclusión:

|  | En «restricciones de columna» | En «restricciones de tabla» |
| --- | --- | --- |
| `UNIQUE` | «(cuando se aplica a una sola columna)» | «Cuando es sobre varias columnas» |
| `CHECK` | «Valida una condición lógica» | «Cuando involucra varias columnas» |

Un `UNIQUE` no cambia de naturaleza según dónde se escriba: **cambia cuántas columnas nombra**. Y cuando nombra dos, no hay adentro de qué columna ponerlo — por eso va afuera.

| | |
| --- | --- |
| ✅ | «`UNIQUE` se declara a nivel de columna o de tabla, según cuántas nombre» |
| ⚠️ | «`UNIQUE` es una restricción de columna» — cierto en el caso de una sola, falso en general |
| ❌ | «Hay restricciones de columna y de tabla, y cada una es de una clase distinta» |

**Las dos únicas que no tienen forma de tabla** dentro del `CREATE TABLE` son `NOT NULL` y `DEFAULT`, porque las dos hablan siempre de una columna nombrada y de ninguna otra.

### 4.4 ¿Para qué sirve entonces esta clasificación?

**Respuesta: para escribir el `CREATE TABLE`, y para poder nombrar la restricción.**

Es una regla de escritura, y es útil como tal: **una restricción que nombra dos columnas no entra adentro de ninguna de las dos**. Pero hay una consecuencia menos obvia y más cara, que se ve en el mensaje de error del apunte:

```
Violation of PRIMARY KEY constraint 'PK__Personas__3214EC078DFC3B8A'.
```

La forma de columna —`Id INT PRIMARY KEY`— no deja lugar para el nombre, así que el motor lo genera: `PK__Personas__3214EC078DFC3B8A`, con un sufijo distinto en cada base creada. La forma de tabla lo obliga a estar, y por eso el error de la `FOREIGN KEY` sí se puede leer y buscar: `"FK_Localidades_PERSONAS"`.

**La elección entre columna y tabla no es solo sintáctica: decide si el error va a ser legible.**

---

## 5. Las dos clasificaciones, cruzadas

Es el centro del documento. Las dos clasificaciones no se corresponden, y la única forma de verlo es ponerlas en los dos ejes de una misma tabla.

### 5.1 ¿Cómo se cruzan?

**Respuesta: en una grilla de tres por dos —y las seis celdas tienen contenido.**

| Integridad ↓ / Restricción → | **De columna** | **De tabla** |
| --- | --- | --- |
| **De entidad** | `Id INT PRIMARY KEY` <br> `Email VARCHAR(100) UNIQUE` | `CONSTRAINT PK_Detalle PRIMARY KEY (Id_Pedido, Id_Producto)` <br> `CONSTRAINT UQ_… UNIQUE (Id_Pedido, Id_Producto)` |
| **Referencial** | `Id_LugarNacimiento INT REFERENCES Localidades(Id)` | `CONSTRAINT FK_Personas_Localidades FOREIGN KEY (…) REFERENCES …` |
| **De dominio** | `Nombre VARCHAR(100) NOT NULL` <br> `Edad INT CHECK (Edad >= 0)` <br> `Activo BIT DEFAULT 1` | `CONSTRAINT CK_Pedido_Fechas CHECK (FechaEntrega >= FechaPedido)` |

**Que las seis celdas tengan contenido es la demostración de que los ejes son independientes.** Si una clasificación fuera un detalle de la otra, habría celdas vacías: los tipos de una caerían adentro de los tipos de la otra. No pasa. **Cualquiera de los tres tipos de integridad se puede escribir de las dos maneras.**

### 5.2 ¿Qué se lee en una fila y qué en una columna?

**Respuesta: la fila es homogénea; la columna no.**

Es el hallazgo que ordena todo el documento:

| Se lee | Qué se encuentra | Qué tienen en común |
| --- | --- | --- |
| **Una fila** —«integridad referencial»— | `REFERENCES` en línea y `CONSTRAINT … FOREIGN KEY` | **Todo**: es la misma garantía escrita de dos maneras |
| **Una columna** —«restricción de tabla»— | `FOREIGN KEY`, `PRIMARY KEY`, `UNIQUE`, `CHECK` | **Solo el lugar donde se escriben** |

Puesto en el caso concreto: **una `FOREIGN KEY` de tabla asegura la integridad referencial, y eso se dice de ella con todo sentido. Pero un `UNIQUE`, que está en ese mismo grupo, no tiene nada que ver con la integridad referencial** — asegura integridad de entidad. Lo único que comparte con la `FOREIGN KEY` es que se escribe afuera de las columnas.

| | |
| --- | --- |
| ✅ | «`FOREIGN KEY` asegura la integridad referencial» |
| ✅ | «`UNIQUE` asegura la integridad de entidad» |
| ⚠️ | «`UNIQUE` es una restricción de tabla» — cierto, y no dice nada sobre qué preserva |
| ❌ | «Las restricciones de tabla aseguran la integridad referencial» |

**Por eso «de tabla» nunca puede ser una respuesta a «¿qué garantiza esto?».** Responde a otra pregunta: «¿dónde lo escribo?».

### 5.3 ¿Cuál de las dos miradas se usa primero?

**Respuesta: la de la integridad para diseñar; la de la restricción para escribir.**

Las dos hacen falta, y en este orden:

1. **Qué quiero que sea verdad** de los datos → elijo el tipo de integridad, y de ahí sale **cuál** restricción (§3).
2. **Cuántas columnas nombra** esa restricción → sale **dónde** la escribo, y si le puedo poner nombre (§4).

Invertir el orden es el error típico: empezar por «acá va un `UNIQUE`» sin haber dicho qué se quiere impedir. **La segunda pregunta no se puede contestar mal si la primera se contestó bien** — y no se puede contestar bien si la primera no se contestó.

---

## 6. El esquema entero, anotado

El `CREATE TABLE` del Ejemplo 3 del apunte, con cada restricción ubicada en su celda de la §5.1:

```sql
CREATE TABLE Localidades
(
    Id INT PRIMARY KEY,                 -- entidad  · de columna
    Nombre VARCHAR(100) NOT NULL        -- dominio  · de columna
)
GO
CREATE TABLE Personas
(
    Id INT PRIMARY KEY,                 -- entidad  · de columna
    Nombre VARCHAR(100) NOT NULL,       -- dominio  · de columna
    Id_LugarNacimiento INT NOT NULL,    -- dominio  · de columna
    CONSTRAINT FK_Localidades_PERSONAS  -- referencial · de tabla
        FOREIGN KEY (Id_LugarNacimiento) REFERENCES Localidades(Id)
);
```

Lo que se ve leyendo los comentarios en columna:

- **Cinco restricciones, tres tipos de integridad, dos alcances.** Los números no coinciden porque cuentan cosas distintas.
- **Las tres primeras de `Personas` son de columna y no son del mismo tipo de integridad** —una de entidad, dos de dominio—: la confirmación de la §5.2.
- **La única con nombre propio es la de tabla.** Las otras cuatro van a aparecer en los errores con el nombre que les ponga el motor (§4.4).

---

## 7. Lo que este documento no cubre

| Ausencia | Tipo | Qué corresponde |
| --- | --- | --- |
| Que estos scripts corran | **Pendiente** | Nada de esto se ejecutó contra un SQL Server |
| El texto del estándar ISO/IEC 9075 | **Pendiente** | No se consiguió completo; se usó la documentación de PostgreSQL como sustituto declarado ([Respaldo-Bibliografico.md](Respaldo-Bibliografico.md) §1) |
| `ON DELETE` / `ON UPDATE` | **Pendiente** | Qué pasa al borrar una localidad usada es una decisión, y está abierta en [Registro.md](Registro.md) §5.4 |
| *Triggers* y procedimientos | **Otra herramienta** | Es como se hacen cumplir las reglas de la §3.4 que no se pueden declarar |
| Índices | **Otra herramienta** | `PRIMARY KEY` y `UNIQUE` crean índices, pero eso es rendimiento, no integridad |
| Colación y acentos | **Pendiente** | Si `'Parana'` y `'Paraná'` son el mismo valor lo decide la colación, no el tipo. Abierto en [Registro.md](Registro.md) §5.6 |
| Composición, agregación, herencia | **No aplica acá** | Son el mapeo que anuncia el temario; van en su propio documento |
| `NOT NULL` como objeto del catálogo | **Supuesto** | PostgreSQL lo declara «funcionalmente equivalente a `CHECK (columna IS NOT NULL)`», que no dice cómo lo guarda SQL Server. Sin verificar |

---

## 8. El criterio, en una línea

> **La restricción es la regla que se debe cumplir; la integridad es el dato que la cumple. Una se lee en el esquema, la otra se comprueba en las filas — y por eso una se clasifica por dónde se escribe y la otra por lo que preserva.**
