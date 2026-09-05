# Hallazgos 01 — Defectos verificables en las fuentes

> **Qué es esto**: la lista de cosas que **no cierran** en el material leído el 2026-09-05. Cada una con la cita literal que la sostiene y con la distinción entre *error que rompe la ejecución* y *desprolijidad que confunde*.
>
> **Fuentes revisadas**: el apunte [«Integridad y restricciones»](https://docs.google.com/document/d/1FkZdxlDmhQr-ngvh25fw3_bGXwKcL9nB/preview) y [`/APLICADA/LAB/Drive/Temas.md`](../../../../Drive/Temas.md).
>
> **Postura**: ninguno de estos hallazgos se corrigió. Corregir el material didáctico es una decisión de quien lo dicta, no de quien lo extracta.

---

## 1. En el apunte

### 1.1 El `FOREIGN KEY` de la Figura 3.7 no compila

**Rompe la ejecución.** En la Figura 3.7 la columna se declara `Id_LugarNacimiento` y la restricción la referencia con otro nombre:

```sql
Id_LugarNacimiento INT NOT NULL,
CONSTRAINT FK_Personas FOREIGN KEY (Id_LuagarNacimiento) REFERENCES Localidades(Id)
```

`Id_LuagarNacimiento` tiene las vocales invertidas. SQL Server rechaza el `CREATE TABLE` porque la columna no existe.

**Dónde no pasa:** en el script «Todo junto» (`3_ejemplo3_dominio.sql`) el nombre está bien escrito y la restricción se llama `FK_Localidades_PERSONAS`. **Quien copie de la figura falla; quien copie del script anda.**

### 1.2 El fragmento de creación de la base usa antes de crear

**Rompe la ejecución.** El primer bloque de la sección III trae este orden:

```sql
USE master
GO
DROP DATABASE IF EXISTS Ejemplo_Integridad_DB
GO
USE Ejemplo_Integridad_DB;   -- ← la base todavía no existe
GO
CREATE DATABASE Ejemplo_Integridad_DB;
```

El `USE` va antes del `CREATE DATABASE`, y la base acaba de ser borrada por el `DROP` de arriba. Los tres scripts «Todo junto» tienen el orden correcto: `DROP` → `CREATE` → `USE`.

### 1.3 El enunciado del Ejemplo 2 nombra el tipo de integridad equivocado

**Confunde, no rompe.** El Ejemplo 2 se titula «Integridad Referencial» y su enunciado dice: «Aplicar la integridad **de entidad** entre las tablas personas y localidades». Lo que el ejemplo aplica es integridad referencial —una `FOREIGN KEY`—. La integridad de entidad era la del Ejemplo 1.

### 1.4 Los datos cambian entre ejemplos sin que el texto lo señale

**Confunde, no rompe.** Se supone que es el mismo caso a lo largo de los tres ejemplos, pero los valores no son idénticos:

| Dónde | Qué dice | Contra |
| --- | --- | --- |
| Ejemplo 1 y 2 | `(1, 'Parana')` | Ejemplo 3: `(1, 'Paraná')` |
| Figura 3.7 | `(1, 'Daniel', 1)` | Todo el resto: `(1, 'Daniela', 1)` |
| Figura 3.7 | `(2, 'Andres', 2)` | Todo el resto: `(2, 'Andrés', 2)` |

No es cosmético: el par `Parana` / `Paraná` es exactamente el caso que abre la discusión sobre dominio y colación —ver [Registro.md](Registro.md) §5.6—.

### 1.5 `CHECK` y `DEFAULT` se clasifican pero no se ejemplifican

**Vacío de cobertura.** La §2 del apunte lista `CHECK` («valida una condición lógica») y `DEFAULT` («asigna un valor por defecto») entre las restricciones de columna, y `CHECK` otra vez entre las de tabla «cuando involucra varias columnas». Ninguna de las dos aparece en ningún ejemplo: el Ejemplo 3, titulado «Integridad de Dominio», se resuelve entero con `NOT NULL`.

**Por qué importa:** el único ejemplo de integridad de negocio que da el apunte —«que la fecha de entrega no sea anterior a la fecha de pedido»— es precisamente un `CHECK` de varias columnas, y queda sin mostrar.

### 1.6 El apunte no cubre el mapeo que anuncia el temario

**Vacío de cobertura.** El Tema 2 se titula «Integridad referencial y restricciones (constraint) - **Mapeo de Composición y Agregación**». El apunte trata integridad y restricciones, y no menciona composición ni agregación en ninguna parte. Puede ser que el mapeo viva en las guías de práctica —no lo sabemos hasta leerlas—.

### 1.7 `FOREIGN KEY` está clasificada solo como restricción de tabla, y el propio Ejemplo 2 la declara sobre una columna

**Confunde, no rompe.** La §2 del apunte ubica `FOREIGN KEY` entre las «restricciones de tabla (aplican a una o más columnas y relaciones)», y no la menciona entre las de columna. Pero la Figura 3.5 muestra primero esta forma:

```sql
Id_LugarNacimiento INT REFERENCES Localidades(Id)
```

Eso es una clave foránea declarada **dentro de la columna**, y el apunte la presenta como equivalente a la variante `CONSTRAINT … FOREIGN KEY (…)`, que sí es de tabla.

**Por qué importa:** es el tercer caso —después de `UNIQUE` y `CHECK`, que la fuente sí pone en las dos listas— que muestra que el eje «columna / tabla» no clasifica restricciones sino declaraciones. Ver [Registro.md](Registro.md) §3.2.

---

## 2. En el temario (`Temas.md`)

### 2.1 Dos enlaces distintos apuntan al mismo documento, dos veces

**Confunde, no rompe.** Verificado comparando los identificadores de documento:

| Entradas | Identificador compartido |
| --- | --- |
| «Somee.com - hosting sql-server» y «Administración con SSMS - Inicio - Ejemplos básicos de SQL» | `1pJluXDo4JYlc4L1ph_83AWjQ2AtzGCos` |
| «Guia 2.1 …» y «Guia 2.2 …» | `1Wjnpf8ePnEhgrUPCFvDrfDwI5O7qO7E6` |

En los dos casos, dos títulos que prometen contenidos distintos llevan al mismo lado. Uno de cada par está mal enlazado, y **no se puede saber cuál sin abrir el destino**.

### 2.2 El Tema 3 está vacío

**Pendiente.** El archivo termina con el encabezado «## Tema 3. Integridad referencial y restricciones(constraint) - Mapeo de Herencia» y ninguna línea debajo.

---

## 3. Lo que no se revisó

| Ausencia | Tipo | Motivo |
| --- | --- | --- |
| Que los scripts corran de verdad contra un SQL Server | Pendiente | No se ejecutó ninguno. Los hallazgos §1.1 y §1.2 son **razonados** sobre el texto, no verificados corriendo |
| El repositorio de ejemplos de GitHub | Pendiente | No se abrió: puede tener los scripts ya corregidos |
| Las guías 2.1 y 2.2 | Pendiente | Sin leer; de ellas depende el hallazgo §1.6 |
| Los mensajes de error del Ejemplo 4 | Verificado por la fuente | Están citados literalmente en el apunte; no los reprodujimos |
