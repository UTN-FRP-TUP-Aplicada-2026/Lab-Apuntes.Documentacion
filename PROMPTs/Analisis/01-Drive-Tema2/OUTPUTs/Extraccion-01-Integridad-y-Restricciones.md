# Extracción 01 — «Integridad y restricciones» (apunte de Drive)

> **Fuente**: <https://docs.google.com/document/d/1FkZdxlDmhQr-ngvh25fw3_bGXwKcL9nB/preview> — «UTN - FRP - TUP - Programación aplicada - SQL Server - Integridad y restricciones», firmado *Filipuzzi, Fernando Rafael*.
>
> **Fecha de extracción**: 2026-09-05.
>
> **Qué es esto**: la transcripción ordenada del apunte, sin agregados. Lo que acá se afirma, lo afirma la fuente. La discusión sobre estos contenidos vive en [Registro.md](Registro.md); los defectos detectados en la fuente, en [Hallazgos-01-Fuente.md](Hallazgos-01-Fuente.md).
>
> **Repositorio de ejemplos que declara la fuente**: <https://github.com/UTN-FRP-TUP-Aplicada-2026/Ejemplos_integridad_y_restricciones> *(no verificado: no se abrió).*

---

## 1. Integridad en bases de datos

**Definición de la fuente**: «Integridad es un concepto qué nos permite garantizar que los datos son válidos y consistentes. En una base de datos relacional, la integridad significa que los datos cumplen reglas lógicas y de negocio, evitando errores, duplicados o inconsistencias.»

Clasificación en cuatro tipos:

| Tipo | Qué garantiza | Con qué se logra |
| --- | --- | --- |
| **a. De entidad** | Cada fila se identifica de forma única; evita duplicados | `PRIMARY KEY`, `UNIQUE` |
| **b. Referencial** | Las relaciones entre tablas son válidas; evita «datos huérfanos» | `FOREIGN KEY` |
| **c. De dominio** | Los valores respetan tipo, formato, rango o condición | `CHECK`, `NOT NULL`, tipos de datos, valores por defecto |
| **d. De negocio** | Reglas propias de la organización | *(la fuente no lo asocia a un mecanismo)* |

Ejemplo que da la fuente para (d): «que la fecha de entrega no sea anterior a la fecha de pedido».

### 1.1 `PRIMARY KEY` frente a `UNIQUE`, según la fuente

| `PRIMARY KEY` | `UNIQUE` |
| --- | --- |
| Identifica unívocamente cada registro de la tabla | Garantiza que los valores en una columna o combinación no se repitan |
| Solo puede haber una por tabla | Puede haber varias en una misma tabla |
| No permite `NULL` | Sí permite `NULL` |
| No permite valores repetidos | — |
| Puede estar compuesta por una o varias columnas | — |

Sobre los `NULL` en `UNIQUE`, la fuente precisa: «normalmente permite uno solo en SQL Server, pero múltiples en otros motores como PostgreSQL».

---

## 2. Restricciones (*constraints*)

**Definición de la fuente**: «Las restricciones son las reglas físicas o lógicas que el SGBD (SQL Server, PostgreSQL, MySQL, etc.) impone a los datos para garantizar que se cumpla la integridad.»

Clasificadas por alcance:

| De columna | De tabla |
| --- | --- |
| `NOT NULL` — impide valores nulos | `PRIMARY KEY` — asegura la integridad de entidad |
| `CHECK` — valida una condición lógica | `FOREIGN KEY` — asegura la integridad referencial |
| `DEFAULT` — asigna un valor por defecto | `UNIQUE` — cuando es sobre varias columnas |
| `UNIQUE` — cuando se aplica a una sola columna | `CHECK` — cuando involucra varias columnas |

La fuente cierra la sección con una sola frase destacada: **«Las restricciones que ofrece el SGBD aseguran la integridad.»**

---

## 3. Los ejemplos

La fuente advierte por qué elige mostrar datos y no un diagrama: «Si bien existen representaciones gráficas para representar el modelo relacional, en este caso es mejor trabajar con valores para entender la naturaleza del tema.»

El caso es siempre el mismo par de tablas —`Localidades` y `Personas`— y cada ejemplo le agrega una capa de restricción.

### 3.1 Ejemplo 1 — Integridad de entidad (`1_ejemplo1_primary_key.sql`)

```sql
USE master;
DROP DATABASE IF EXISTS Ejemplo_Integridad_DB
GO
CREATE DATABASE Ejemplo_Integridad_DB
GO
USE Ejemplo_Integridad_DB
GO
CREATE TABLE Localidades
(
    Id INT PRIMARY KEY,
    Nombre VARCHAR(100)
)
GO
INSERT INTO Localidades(Id, Nombre)
VALUES(1, 'Parana'),(2, 'La Paz'),(3, 'Hernandarias'),(4, 'Hasenkamp');
GO
CREATE TABLE Personas
(
    Id INT PRIMARY KEY,
    Nombre VARCHAR(100),
    Id_LugarNacimiento INT
)
GO
INSERT INTO Personas(Id, Nombre, Id_LugarNacimiento)
VALUES(1, 'Daniela', 1),(2, 'Andrés', 2),(3, 'Daniela', 2),
      (4, 'Andrés', 1),(5, 'Armando', 1),(6, 'Arturo', 3)
GO
--Consultando el listado de nombres y su lugar de nacimiento
SELECT p.Id, p.Nombre, l.Nombre AS LugarNacimiento
FROM Personas p
INNER JOIN Localidades l ON p.Id_LugarNacimiento = l.Id
ORDER BY L.Nombre
```

En este ejemplo `Id_LugarNacimiento` es una columna `INT` común: **todavía no hay `FOREIGN KEY`**.

### 3.2 Ejemplo 2 — Integridad referencial (`2_ejemplo2_foreing_key`)

La fuente muestra las dos formas de declarar la clave foránea. En línea:

```sql
CREATE TABLE Personas
(
    Id INT PRIMARY KEY,
    Nombre VARCHAR(100),
    Id_LugarNacimiento INT REFERENCES Localidades(Id)
)
```

Y como restricción con nombre propio:

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

En el script completo la variante en línea queda comentada y se conserva la nombrada. Los datos insertados son los mismos del Ejemplo 1.

Referencia que cita la fuente: [Microsoft — `CREATE TABLE`](https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-transact-sql?view=sql-server-ver17).

### 3.3 Ejemplo 3 — Integridad de dominio (`3_ejemplo3_dominio.sql`)

```sql
CREATE TABLE Localidades
(
    Id INT PRIMARY KEY,
    Nombre VARCHAR(100) NOT NULL
)
GO
INSERT INTO Localidades(Id, Nombre)
VALUES(1, 'Paraná'),(2, 'La Paz'),(3, 'Hernandarias'),(4, 'Hasenkamp')
GO
CREATE TABLE Personas
(
    Id INT PRIMARY KEY,
    Nombre VARCHAR(100) NOT NULL,
    Id_LugarNacimiento INT NOT NULL,
    CONSTRAINT FK_Localidades_PERSONAS FOREIGN KEY (Id_LugarNacimiento)
        REFERENCES Localidades(Id)
);
```

Lo que agrega respecto del Ejemplo 2 es `NOT NULL` en tres columnas. **No aparece ningún `CHECK` ni ningún `DEFAULT`** en los ejemplos, aunque la §2 los clasifica.

### 3.4 Ejemplo 4 — Prueba de restricciones

Es el único ejemplo que corre contra la restricción en vez de a favor, y anota el mensaje literal que devuelve el motor:

| Intento | Mensaje que registra la fuente |
| --- | --- |
| `INSERT ... VALUES(1, 'Cecilia', 1)` — `Id` repetido | `Violation of PRIMARY KEY constraint 'PK__Personas__3214EC078DFC3B8A'. Cannot insert duplicate key in object 'dbo.Personas'. The duplicate key value is (1).` |
| `INSERT ... VALUES(7, NULL, 1)` — nombre nulo | `Cannot insert the value NULL into column 'Nombre', table 'Ejemplo_Integridad_DB.dbo.Personas'; column does not allow nulls. INSERT fails.` |
| `INSERT ... VALUES(7, 'Cecilia', 100)` — localidad inexistente | `The INSERT statement conflicted with the FOREIGN KEY constraint "FK_Localidades_PERSONAS". The conflict occurred in database "Ejemplo_Integridad_DB", table "dbo.Localidades", column 'Id'.` |

---

## 4. Dónde se ubica esto en el temario

Según [`/APLICADA/LAB/Drive/Temas.md`](../../../../Drive/Temas.md), este apunte es el material teórico del **Tema 2 — «Integridad referencial y restricciones (constraint) - Mapeo de Composición y Agregación»**, acompañado de dos guías de práctica, y precede al **Tema 3 — «… Mapeo de Herencia»**, que en el archivo figura **sin contenido**.

---

## 5. Lo que esta extracción no cubre

| Ausencia | Tipo | Motivo |
| --- | --- | --- |
| Las figuras 3.1 a 3.7 (imágenes de las tablas con valores) | No aplica a este medio | La lectura del documento devuelve texto; las capturas no se transcriben |
| Las guías de práctica 2.1 y 2.2 | Pendiente | No se leyeron todavía (ver [Hallazgos-01-Fuente.md](Hallazgos-01-Fuente.md) §2 sobre sus enlaces) |
| El repositorio de ejemplos de GitHub | Pendiente | No se abrió; no se puede afirmar que el contenido coincida con el apunte |
| Los demás temas del temario | No aplica | Este prompt entra por el Tema 2 |
