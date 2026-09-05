# Hallazgos 01 — Defectos verificables en el apunte de herencia

> **Qué es esto**: lo que no cierra en el [apunte de mapeo de herencia](https://docs.google.com/document/d/1ZkB2ylhtA6OHwBaDriGWs_FJ4-voSNAG/edit), revisado el 2026-09-05. Cada hallazgo con su evidencia y con la distinción entre **rompe la ejecución**, **contradice a la figura** y **vacío conceptual**.
>
> **Postura**: no se corrigió nada en el original. La transcripción fiel está en [Extraccion-01-Herencia.md](Extraccion-01-Herencia.md); las correcciones se aplican en el apunte nuevo, y allí se declaran.
>
> **El patrón**: en los tres ejemplos, **las figuras están bien y el SQL está mal**. Eso importa para decidir qué se corrige: el modelo del autor es el de las figuras.

---

## 1. En el SQL

### 1.1 El discriminador de TPH está declarado `UNIQUE`

**Rompe el modelo, y en silencio.**

```sql
Tipo INT UNIQUE NOT NULL,
```

`UNIQUE` sobre el discriminador significa que **no puede haber dos filas del mismo tipo**: un solo rectángulo, un solo círculo, y la tabla se acabó. Es exactamente lo contrario de lo que TPH necesita — el discriminador se repite tantas veces como instancias haya de cada subclase.

**La figura lo desmiente:** el diagrama de la tabla `Figuras` rotula `Tipo` únicamente como «No nulo», en rojo. No dice único.

**Por qué no salta a la vista:** los datos de ejemplo tienen una fila por tipo —un rectángulo y un círculo—, así que **el `UNIQUE` se cumple y nadie lo nota**. El error aparece recién al insertar el segundo rectángulo.

| | |
| --- | --- |
| ✅ | `Tipo INT NOT NULL` — se repite, como corresponde a un discriminador |
| ❌ | `Tipo INT UNIQUE NOT NULL` — una fila por tipo |

### 1.2 `PRIMARY KEY` y `FOREIGN KEY` sin paréntesis

**Rompe la ejecución.** En los ejemplos de TPT y TPC:

```sql
CONSTRAINT PK_Rectangulos PRIMARY KEY Id,
CONSTRAINT FK_Rectangulos_Figuras FOREIGN KEY Id REFERENCES Figuras(Id)
```

La sintaxis de restricción de tabla exige la lista de columnas entre paréntesis: `PRIMARY KEY (Id)`, `FOREIGN KEY (Id)`. Tal como está, ninguno de los dos `CREATE TABLE` compila.

### 1.3 `Circulos` tiene `Ancho` y `Largo` en vez de `Radio`

**Contradice a la figura**, y en los dos ejemplos: en TPT y en TPC.

```sql
CREATE TABLE Circulos ( Id INT, Ancho DECIMAL(18,2), Largo DECIMAL(18,2), ... );
```

Las figuras `Figuras_Circulos` y `Circulos` muestran **`Radio`**, y la clase `Circulo` del modelo de objetos declara `public double Radio`. **El único lugar donde el SQL le da `Radio` al círculo es la consulta `getById`** de la sección TPT — que además usa los nombres de tabla de las figuras.

### 1.4 Una clave foránea llamada `PK_…`

**Confunde, no rompe.** En `Circulos` (TPT):

```sql
CONSTRAINT PK_Rectangulos_Figuras FOREIGN KEY Id REFERENCES Figuras(Id)
```

Dos problemas en un solo nombre: dice `PK_` siendo una clave foránea, y dice `Rectangulos` estando en la tabla de círculos. Es una copia sin adaptar.

### 1.5 El C# no compila en dos de las tres celdas

**Rompe la ejecución.** En TPT y en TPC, la clase base aparece así:

```csharp
public double Area;{get; set;}
```

El punto y coma cierra la declaración de campo, y lo que sigue queda suelto. **En la celda de TPH la misma clase está bien escrita** —`public double Area{get; set;}`—, lo que confirma que es un error de copia y no una variante intencional.

### 1.6 En TPC, la clave primaria se declara dos veces

**Rompe la ejecución.**

```sql
CREATE TABLE Rectangulos
(
    Id INT PRIMARY KEY IDENTITY(1,1),      -- primera
    ...
    CONSTRAINT PK_Rectangulos PRIMARY KEY Id   -- segunda
);
```

Una tabla admite una sola clave primaria. Con las dos declaraciones el `CREATE TABLE` falla, y falla en las dos tablas del ejemplo.

---

## 2. En la estructura del documento

### 2.1 Hay dos secciones «II»

**Confunde.** El índice y los encabezados numeran: `I. Introducción`, `II. Tabla por Jerarquía`, **`II. Tabla por Subclase`**, `III. Tabla por Clase Concreta`, `IV. Referencias`. La segunda debería ser `III`, y las siguientes correrse.

### 2.2 La tabla comparativa está incompleta

**Vacío de cobertura.** Sus últimas tres filas no tienen contenido en todas las columnas:

| Fila | Qué falta |
| --- | --- |
| «Requiere un campo adicional, discriminador» | Solo la columna TPH tiene valor; TPT y TPC quedan vacías —siendo que la respuesta es «no» en las dos— |
| «Modelo orientado a objetos» | Una sola imagen para las tres columnas, sin aclarar que el modelo de objetos es el mismo |
| «Modelo relacional» | Las imágenes están, pero sin epígrafe que diga qué tabla es cada una |

### 2.3 El encabezado dice «2025»

**Confunde.** El título es «Programación aplicada **2025**», mientras que el temario y el resto de los documentos del curso son de 2026.

---

## 3. Vacíos conceptuales

No son errores: son cosas que el modelo no puede garantizar y el documento no menciona. **Son las que más valor tienen para el apunte nuevo.**

### 3.1 TPH no impide un rectángulo con radio

Nada en el esquema evita `INSERT INTO Figuras(Tipo, Area, Ancho, Largo, Radio) VALUES (1, 1.0, 1.0, 1.0, 5.0)` — un rectángulo con radio, o un círculo con ancho. **La correspondencia entre el discriminador y las columnas que deben ser nulas no está declarada**, y se puede declarar con un `CHECK`.

Es el precio real de TPH, y el documento solo menciona «muchas columnas nulas», que es el precio de almacenamiento — no el de integridad.

### 3.2 TPC no puede garantizar identificadores únicos en la jerarquía

Las dos tablas usan `IDENTITY(1,1)` **por separado**, así que las dos generan `1`, `2`, `3`… **Un rectángulo y un círculo pueden tener el mismo `Id`.** En las figuras eso no se ve porque el rectángulo es el `1` y el círculo el `2`, pero nada lo garantiza.

Y no es un detalle de implementación: si algún día otra tabla necesita referenciar «una figura cualquiera», no hay a qué apuntar. **Está confirmado por la documentación de EF Core**, que resuelve el mismo problema con una secuencia compartida:

> «*EF Core requires that all entities in a hierarchy have a unique key value, even if the entities have different types* … *this means a simple `Identity` column cannot be used*.»

### 3.3 Falta la condición que decide si una estrategia es siquiera válida

El documento compara las tres estrategias por **ventajas, desventajas y ejemplo ideal** — todos criterios de conveniencia. La bibliografía académica agrega criterios de **validez**: hay especializaciones para las que una estrategia directamente no sirve. Está desarrollado en [Registor.md](Registor.md) §4.

### 3.4 `ON UPDATE CASCADE` sobre una clave `IDENTITY`

**Razonado, no verificado.** El TPT del apunte declara `ON DELETE CASCADE ON UPDATE CASCADE` sobre una clave foránea que apunta a `Figuras.Id`, que es `IDENTITY(1,1)`. Un valor `IDENTITY` no se actualiza por las vías normales, así que **la cláusula `ON UPDATE` no tiene ocasión de dispararse**: es inerte.

*(Como contraste: para el mismo patrón, EF Core genera `ON DELETE NO ACTION` y ninguna cláusula de update — ver [Registor.md](Registor.md) §5.)*

---

## 4. Lo que no se revisó

| Ausencia | Tipo | Motivo |
| --- | --- | --- |
| Que los scripts corran | Pendiente | No se ejecutó ninguno. Los hallazgos §1.2, §1.5 y §1.6 son **razonados sobre la sintaxis** |
| El repositorio de ejemplos | Pendiente | No se abrió; puede tener los scripts ya corregidos |
| Las guías de práctica del Tema 3 | Pendiente | El temario enlaza las guías del Tema 2 en el lugar de las del Tema 3 |
