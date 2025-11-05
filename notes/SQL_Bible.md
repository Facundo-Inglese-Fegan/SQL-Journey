# 📖 La Biblia de SQL

**SQL** significa **Structured Query Language** (Lenguaje de Consulta Estructurado).

**Analogía:** Piensa en una base de datos como un gigantesco almacén (como los de Amazon o IKEA), lleno de millones de estanterías y cajas que contienen tus datos.

> **SQL es el "idioma universal" que hablas con el encargado del almacén** (el motor de la base de datos).

No puedes entrar y coger las cajas tú mismo. Tienes que rellenar un formulario (una *query* de SQL) para pedirle al encargado *exactamente* lo que necesitas.

* `SELECT nombre, precio...` = "Quiero que me traigas el nombre y el precio..."
* `FROM productos...` = "...de la estantería de 'productos'..."
* `WHERE categoria = 'Lácteos'` = "...pero solo aquellos que estén en la caja de 'lácteos'."

---

## 1. Los 4 Pilares de SQL (Las "Leyes")

SQL se divide conceptualmente en sublenguajes. Entenderlos te ayuda a estructurar tu pensamiento:

* **DQL (Data Query Language):** Para consultar datos. Es el 90% del trabajo de un analista.
  * `SELECT`
* **DML (Data Manipulation Language):** Para modificar datos existentes.
  * `INSERT`, `UPDATE`, `DELETE`
* **DDL (Data Definition Language):** Para definir la *estructura* de la base de datos.
  * `CREATE`, `ALTER`, `DROP`, `TRUNCATE`
* **DCL (Data Control Language):** Para gestionar permisos y seguridad.
  * `GRANT`, `REVOKE`
* **TCL (Transaction Control Language):** Para gestionar transacciones (grupos de operaciones).
  * `COMMIT`, `ROLLBACK`, `SAVEPOINT`

---

## 2. DQL: El Arte de Preguntar (`SELECT`)

La estructura fundamental de casi cualquier consulta.

```sql
SELECT
    columna1,
    columna2 AS Alias,
    COUNT(DISTINCT columna3) AS ConteoUnico
FROM
    mi_tabla
WHERE
    condicion_fila = 'valor'
GROUP BY
    columna1, columna2
HAVING
    COUNT(DISTINCT columna3) > 1
ORDER BY
    Alias DESC
LIMIT 100;
```

### 2.1. SELECT y FROM

* SELECT *: Selecciona todas las columnas. (Mala práctica en producción, bueno para explorar).
* SELECT columna1, columna2: Selecciona columnas específicas.
* SELECT DISTINCT pais: Devuelve solo los valores únicos de la columna pais.
* SELECT columna1 AS mi_alias: Aliasing. Renombra una columna en el resultado. Vital para la legibilidad.

### 2.2. WHERE: El Filtro

Filtra filas antes de cualquier agregación.

* Operadores: =, <>, !=, >, <, >=, <=
* Lógicos: AND, OR, NOT
* Especiales:
  * BETWEEN: WHERE edad BETWEEN 18 AND 30 (inclusivo)
  * IN: WHERE pais IN ('España', 'México', 'Argentina') (más eficiente que múltiples OR)
  * LIKE: Para buscar patrones de texto.
  * 'A%': Empieza con "A".
  * '%z': Termina con "z".
  * '%hola%': Contiene "hola".
  * 'Pa_s': El "_" es un comodín para un solo carácter (ej. "País", "Paso").
  * IS NULL / IS NOT NULL: Para buscar valores nulos (no puedes usar = NULL).

### 2.3. ORDER BY: El Orden

Ordena el conjunto de resultados final.

* ORDER BY fecha ASC (Ascendente, por defecto)
* ORDER BY fecha DESC (Descendente)
* ORDER BY pais, ciudad DESC (Ordena primero por país, y dentro de cada país, por ciudad descendente)

---

## 3. JOIN: El Corazón de SQL Relacional

**Analogía:** Piensa en los JOIN como una cremallera. La cláusula ON te dice qué dientes de la cremallera deben encajar (la clave de unión).

```sql
SELECT
    A.pedido_id,
    A.fecha,
    B.nombre_cliente
FROM
    tabla_pedidos AS A
INNER JOIN
    tabla_clientes AS B
ON
    A.cliente_id = B.cliente_id; -- La "clave" que une las tablas
```

* INNER JOIN (La intersección):
  * Solo devuelve filas que tienen una coincidencia en ambas tablas.
  * Cuándo usarlo: "Quiero ver solo los clientes que SÍ han hecho pedidos."

* LEFT JOIN (El "principal" + intersección):
  * Devuelve todas las filas de la tabla izquierda (FROM) y solo las filas coincidentes de la tabla derecha (JOIN).
  * Si no hay coincidencia en la derecha, las columnas de la derecha vendrán como NULL.
  * Cuándo usarlo: "Quiero ver todos mis clientes, y si han hecho un pedido, muéstrame el pedido. Si no, igual muéstrame al cliente."

* RIGHT JOIN:
  * Lo mismo que el LEFT, pero al revés. Devuelve todo de la tabla derecha. (Menos común, generalmente se prefiere reescribir como LEFT JOIN).

* FULL OUTER JOIN:
  * Devuelve todas las filas de ambas tablas. Si hay coincidencia, las une. Si no, rellena con NULL donde falte.
  * Cuándo usarlo: "Quiero un listado maestro de todos mis clientes Y todos mis pedidos, independientemente de si tienen relación o no."

* CROSS JOIN (Producto Cartesiano):
  * Cada fila de la tabla A se une con cada fila de la tabla B.
  * ¡PELIGRO! Si A tiene 1000 filas y B tiene 1000 filas, el resultado es 1,000,000 de filas.
  * Cuándo usarlo: Rara vez. Útil para generar datos de prueba o calendarios maestros.

---

## 4. Agregación: Resumiendo Datos

### 4.1. Funciones de Agregado

Resumen un conjunto de filas en un solo valor.

* COUNT(*): Cuenta el número total de filas.
* COUNT(columna): Cuenta filas donde columna no es NULL.
* COUNT(DISTINCT columna): Cuenta el número de valores únicos.
* SUM(columna): Suma los valores.
* AVG(columna): Calcula el promedio.
* MIN(columna): Devuelve el valor mínimo.
* MAX(columna): Devuelve el valor máximo.

### 4.2. GROUP BY: El Agrupador

Indica a las funciones de agregado cómo "agrupar" los resúmenes.
**Analogía:** Si COUNT() es "contar el dinero", GROUP BY es "contarlo separando por monedas".

```sql
-- Contar cuántos pedidos ha hecho cada cliente
SELECT
    cliente_id,
    COUNT(pedido_id) AS total_pedidos
FROM
    tabla_pedidos
GROUP BY
    cliente_id; -- Agrupamos por cliente
```

> Regla de Oro: Si usas una función de agregado (como COUNT) y también seleccionas una columna normal (como cliente_id), debes incluir esa columna normal en el GROUP BY.

### 4.3. HAVING: El Filtro para Grupos

Es el WHERE de los GROUP BY. Filtra después de que los datos han sido agregados.

* WHERE filtra filas (antes de GROUP BY).
* HAVING filtra grupos (después de GROUP BY).

```sql
-- Mostrar solo los clientes que han hecho MÁS de 5 pedidos
SELECT
    cliente_id,
    COUNT(pedido_id) AS total_pedidos
FROM
    tabla_pedidos
WHERE
    pais = 'España' -- 1. Filtra filas (solo pedidos de España)
GROUP BY
    cliente_id      -- 2. Agrupa por cliente
HAVING
    COUNT(pedido_id) > 5; -- 3. Filtra grupos (solo clientes con >5 pedidos)
```

---

## 5. DML: Manipulando los Datos

* INSERT INTO (Añadir):

```sql
-- Inserta una fila especificando columnas
INSERT INTO clientes (cliente_id, nombre, pais)
VALUES (101, 'Juan Perez', 'México');

-- Inserta múltiples filas
INSERT INTO clientes (cliente_id, nombre, pais)
VALUES
    (102, 'Ana Lopez', 'Argentina'),
    (103, 'Carlos Solis', 'España');
```

* UPDATE (Actualizar):

```sql
-- ¡CUIDADO! Usa siempre un WHERE o actualizarás la tabla entera.
UPDATE clientes
SET
    pais = 'Chile',
    nombre = 'Juan Perez G.'
WHERE
    cliente_id = 101;
```

* DELETE (Borrar):

```sql
-- ¡CUIDADO! Usa siempre un WHERE o borrarás la tabla entera.
DELETE FROM clientes
WHERE cliente_id = 101;
```

---

## 6. DDL: Definiendo la Estructura

* CREATE TABLE (Crear):

```sql
CREATE TABLE empleados (
    empleado_id INT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    departamento VARCHAR(50),
    salario DECIMAL(10, 2),
    fecha_contrato DATE DEFAULT GETDATE() -- Valor por defecto
);
```

> Tipos de Datos Comunes: INT, DECIMAL(precision, escala), VARCHAR(longitud), CHAR(longitud), DATE, DATETIME, BOOLEAN.

* ALTER TABLE (Modificar):

```sql
-- Añadir una columna
ALTER TABLE empleados
ADD email VARCHAR(100);

-- Modificar tipo de dato de una columna
ALTER TABLE empleados
ALTER COLUMN departamento VARCHAR(75);

-- Eliminar una columna
ALTER TABLE empleados
DROP COLUMN salario;
```

* DROP TABLE (Eliminar):

```sql
-- Elimina la tabla entera (estructura y datos)
DROP TABLE empleados;
```

* TRUNCATE TABLE (Vaciar):

```sql
-- Elimina TODAS las filas de la tabla, pero mantiene la estructura.
-- Es mucho más rápido que DELETE FROM sin WHERE, ya que no registra fila por fila.
TRUNCATE TABLE log_eventos;
```

---

## 7. `CASE WHEN`: Lógica Condicional en tu Query

Es el `IF-THEN-ELSE` de SQL. Te permite crear columnas derivadas basadas en reglas de negocio.

**Analogía:** Es como el "Sombrero Seleccionador" de Harry Potter. Examina cada fila y la asigna a un "grupo" (una nueva columna) según las condiciones que le des.

```sql
SELECT
    producto_id,
    precio,
    CASE
        WHEN precio < 20 THEN 'Barato'
        WHEN precio BETWEEN 20 AND 50 THEN 'Rango Medio'
        WHEN precio > 50 AND stock < 10 THEN 'Premium - Stock Bajo'
        ELSE 'Premium'
    END AS categoria_precio
FROM
    productos; 
```

---

## 8. Subconsultas (Consultas Anidadas)

Una consulta dentro de otra consulta. Son potentes pero pueden volverse ilegibles rápidamente.

**Analogía:** Son como las muñecas rusas (Matryoshka). Abres una y hay otra consulta más pequeña dentro.

### 8.1. Subconsulta Escalar (en SELECT o WHERE)

Devuelve un solo valor.

```sql
-- En WHERE (muy común)
-- "Tráeme los empleados que ganan más que el promedio"
SELECT nombre, salario
FROM empleados
WHERE salario > (SELECT AVG(salario) FROM empleados);

-- En SELECT (menos común, puede ser lento)
-- "Tráeme el nombre del empleado y el salario promedio de LA EMPRESA"
SELECT
    nombre,
    salario,
    (SELECT AVG(salario) FROM empleados) AS salario_promedio_total
FROM empleados;
```

### 8.2. Subconsulta de Múltiples Filas (en WHERE con IN o EXISTS)

Devuelve una lista de valores.

```sql
-- "Tráeme los clientes que SÍ han hecho un pedido"
SELECT nombre_cliente
FROM clientes
WHERE cliente_id IN (SELECT DISTINCT cliente_id FROM pedidos);

-- Con EXISTS (a menudo más eficiente que IN)
SELECT nombre_cliente
FROM clientes c
WHERE EXISTS (SELECT 1 FROM pedidos p WHERE p.cliente_id = c.cliente_id);
```

### 8.3. Subconsulta Correlacionada

Una subconsulta que depende de la fila actual de la consulta externa. Es potente pero lenta.

```sql
-- "Tráeme el pedido más reciente de CADA cliente"
-- (Nota: Hoy en día, esto se hace mejor con Funciones de Ventana)
SELECT
    cliente_id,
    pedido_id,
    fecha_pedido
FROM pedidos p_externo
WHERE fecha_pedido = (
    SELECT MAX(fecha_pedido)
    FROM pedidos p_interno
    WHERE p_interno.cliente_id = p_externo.cliente_id -- Correlación
);
```

---

## 9. CTEs (Common Table Expressions) - WITH

Las CTEs son la evolución moderna y legible de las subconsultas. Te permiten definir "tablas temporales" al inicio de tu query.

**Analogía:** Son como "variables" para tablas. Preparas todos tus ingredientes (las CTEs) primero, y luego cocinas tu plato final (el SELECT principal) al final, de forma mucho más limpia.

* Legibilidad: Descomponen un problema complejo en pasos lógicos.
* Reutilización: Puedes referenciar la misma CTE múltiples veces en la misma query.
* Recursividad: Pueden llamarse a sí mismas (útil para jerarquías, como organigramas).

```sql
-- Query compleja: "Ventas por país, pero solo de clientes con más de 5 pedidos"

-- 1. Definimos la CTE 'clientes_frecuentes'
WITH clientes_frecuentes AS (
    SELECT
        cliente_id
    FROM
        pedidos
    GROUP BY
        cliente_id
    HAVING
        COUNT(pedido_id) > 5
),

-- 2. Definimos la CTE 'ventas_clientes'
ventas_clientes AS (
    SELECT
        c.pais,
        p.monto
    FROM
        pedidos p
    INNER JOIN
        clientes c ON p.cliente_id = c.cliente_id
    WHERE
        p.cliente_id IN (SELECT cliente_id FROM clientes_frecuentes) -- Usamos la primera CTE
)

-- 3. Consulta final (limpia y fácil de leer)
SELECT
    pais,
    SUM(monto) AS ventas_totales
FROM
    ventas_clientes
GROUP BY
    pais
ORDER BY
    ventas_totales DESC;
```

---

## 10. Funciones de Ventana (Window Functions) - OVER()
>>
>> EL CONCEPTO MÁS IMPORTANTE DEL ANÁLISIS MODERNO EN SQL.
Permiten realizar cálculos de agregación (como SUM, AVG, RANK) sin colapsar las filas con un GROUP BY.

**Analogía:** GROUP BY es como meter un edificio en una licuadora para saber el peso total. Pierdes todos los pisos. Una Window Function es como un ascensor que visita cada piso (PARTITION BY) y te dice el peso total del edificio sin demolerlo. Ves el detalle (el piso) y el agregado (el total) al mismo tiempo.

La sintaxis clave es OVER (PARTITION BY ... ORDER BY ...):

* PARTITION BY col: Define la "ventana" o grupo (ej. PARTITION BY departamento).
* ORDER BY col: Define el orden dentro de esa ventana (vital para ránkings y acumulados).

### 10.1. Funciones de Agregación de Ventana

```sql
-- "Mostrar el salario de cada empleado, Y el promedio de SU departamento"
SELECT
    nombre,
    departamento,
    salario,
    AVG(salario) OVER (PARTITION BY departamento) AS promedio_depto
FROM
    empleados;
```

### 10.2. Funciones de Ránking

* ROW_NUMBER(): Número único por fila (1, 2, 3, 4).
* RANK(): Ránking con saltos por empates (1, 2, 2, 4).
* DENSE_RANK(): Ránking sin saltos por empates (1, 2, 2, 3).

```sql
-- "Encontrar los 3 empleados con mayor salario por CADA departamento"
WITH ranking_salarios AS (
    SELECT
        nombre,
        departamento,
        salario,
        ROW_NUMBER() OVER(PARTITION BY departamento ORDER BY salario DESC) AS rn
    FROM
        empleados
)
SELECT
    nombre,
    departamento,
    salario
FROM
    ranking_salarios
WHERE
    rn <= 3;
```

### 10.3. Funciones de Valor (Navegación)

* LAG(col): Obtiene el valor de la fila anterior.
* LEAD(col): Obtiene el valor de la fila siguiente.

```sql
-- "Calcular la diferencia de ventas vs. el mes anterior"
SELECT
    fecha_mes,
    ventas,
    LAG(ventas, 1, 0) OVER(ORDER BY fecha_mes) AS ventas_mes_anterior,
    ventas - LAG(ventas, 1, 0) OVER(ORDER BY fecha_mes) AS diferencia_mes_a_mes
FROM
    ventas_mensuales;
```

### 10.4. Acumulados (Running Totals)

```sql
-- "Calcular el acumulado de ventas del año"
SELECT
    fecha_dia,
    ventas_dia,
    SUM(ventas_dia) OVER(ORDER BY fecha_dia) AS ventas_acumuladas
FROM
    ventas_diarias;
```

---

## 11. Combinando Conjuntos de Datos (UNION y UNION ALL)

A diferencia de JOIN (que combina columnas), UNION apila filas.
> Regla: Las consultas deben tener el mismo número de columnas y tipos de datos compatibles.

* UNION: Apila los resultados y elimina duplicados (más lento).
* UNION ALL: Apila los resultados y mantiene duplicados (más rápido).

**Analogía:** UNION ALL es "pegar una lista de invitados debajo de la otra". UNION es "pegar las listas Y tachar los nombres que aparezcan en ambas".

```sql
-- "Crear una lista única de todos los clientes activos e inactivos"
SELECT cliente_id, nombre, 'Activo' AS estado FROM clientes_activos
UNION ALL
SELECT cliente_id, nombre, 'Inactivo' AS estado FROM clientes_inactivos;
```

---

## 12. Vistas (CREATE VIEW)

Una consulta guardada que se comporta como una tabla virtual.

**Analogía:** Es un "acceso directo" en tu escritorio. No es el archivo real, pero te lleva a él. La vista no almacena datos (generalmente), solo almacena la query SELECT.

* Seguridad: Puedes dar acceso a una vista que solo muestre ciertas columnas o filas.
* Simplicidad: Oculta la complejidad de JOINs y lógica de negocio a los usuarios finales (como Power BI).

```sql
-- "Crear una vista simple para el reporte de Power BI"
CREATE VIEW v_Resumen_Ventas_Por_Pais AS
SELECT
    c.pais,
    pr.categoria,
    SUM(p.monto) AS total_ventas,
    COUNT(DISTINCT p.pedido_id) AS total_pedidos
FROM
    pedidos p
JOIN
    clientes c ON p.cliente_id = c.cliente_id
JOIN
    productos pr ON p.producto_id = pr.producto_id
GROUP BY
    c.pais,
    pr.categoria;

-- Ahora, en Power BI, solo necesitas hacer:
SELECT * FROM v_Resumen_Ventas_Por_Pais;
```

---

## 13. Manejo de Nulos (COALESCE e ISNULL)

* ISNULL(columna, valor_si_nulo): Específico de T-SQL (SQL Server). Solo acepta 2 argumentos.
* COALESCE(col1, col2, col3, ...): Estándar ANSI-SQL. Devuelve el primer valor no-nulo que encuentra en la lista. Es más flexible.

```sql
-- Si 'telefono_oficina' es NULL, usa 'telefono_movil'.
-- Si ambos son NULL, usa 'No disponible'.
SELECT
    nombre,
    COALESCE(telefono_oficina, telefono_movil, 'No disponible') AS telefono_contacto
FROM
    empleados;
```

---

## 14. Procedimientos Almacenados (Stored Procedures)

Un Procedimiento Almacenado (o "SP") es un conjunto de una o más sentencias SQL guardadas en la base de datos bajo un nombre.

**Analogía:** Piensa en un SP como una **receta de cocina guardada** o una **macro de Excel**. En lugar de escribir 10 pasos (JOINs, filtros, agregaciones, UPDATEs) cada vez que quieres hacer "Pollo al Curry", simplemente ejecutas `EXEC sp_HacerPolloAlCurry`.

* **Parámetros:** Pueden recibir parámetros de entrada (ej. `@FechaInicio`, `@IDCliente`) y devolver valores de salida.
* **Rendimiento:** El plan de ejecución suele estar "pre-compilado" y cacheado, haciéndolos más rápidos.
* **Seguridad:** Puedes dar permiso a un usuario para *ejecutar el SP* sin darle permiso para *leer las tablas* que usa por debajo.
* **Automatización:** Es la unidad básica para automatizar tareas (ETL/ELT).

### Sintaxis (T-SQL/SQL Server)

```sql
-- 1. CREACIÓN DEL SP
CREATE PROCEDURE sp_ObtenerVentasCliente
    -- Parámetros de entrada
    @ClienteID INT,
    @FechaMin DATE
AS
BEGIN
    -- El 'SET NOCOUNT ON' evita que SQL envíe mensajes de "filas afectadas"
    -- Es una buena práctica para el rendimiento.
    SET NOCOUNT ON;

    -- La lógica de tu query
    SELECT
        p.pedido_id,
        p.fecha_pedido,
        c.nombre_cliente,
        SUM(li.cantidad * li.precio_unitario) AS total_monto
    FROM
        pedidos p
    JOIN
        clientes c ON p.cliente_id = c.cliente_id
    JOIN
        lineas_pedido li ON p.pedido_id = li.pedido_id
    WHERE
        p.cliente_id = @ClienteID
        AND p.fecha_pedido >= @FechaMin
    GROUP BY
        p.pedido_id, p.fecha_pedido, c.nombre_cliente
    ORDER BY
        p.fecha_pedido DESC;

END;
GO

-- 2. EJECUCIÓN DEL SP
EXEC sp_ObtenerVentasCliente @ClienteID = 101, @FechaMin = '2024-01-01';
```

> Uso en Power BI: Puedes llamar a SPs desde Power BI. Es una forma excelente de delegar el procesamiento pesado a la base de datos.

---

## 15. Funciones Definidas por el Usuario (UDFs)

Una UDF es similar a un SP, pero con una diferencia clave: siempre devuelve un valor y (generalmente) no puede modificar datos (no INSERT, UPDATE, DELETE).

**Analogía:** Si un SP es una macro, una UDF es una función personalizada de Excel (como crear tu propia función =CALCULARIVA()).

Tipos de UDFs

* Función Escalar (Scalar): Devuelve un único valor (ej. INT, VARCHAR, DECIMAL).
* Función de Tabla (Table-Valued): Devuelve un conjunto de filas (una tabla).

Sintaxis (T-SQL)

```sql
-- 1. CREACIÓN DE FUNCIÓN ESCALAR
-- "Crear una función que categorice el precio"
CREATE FUNCTION dbo.fn_CategorizarPrecio (@precio DECIMAL(10, 2))
RETURNS VARCHAR(50) -- Lo que devuelve
AS
BEGIN
    DECLARE @categoria VARCHAR(50); -- Variable interna

    IF @precio < 20
        SET @categoria = 'Barato';
    ELSE IF @precio <= 50
        SET @categoria = 'Rango Medio';
    ELSE
        SET @categoria = 'Premium';

    RETURN @categoria; -- El valor de retorno
END;
GO

-- 2. USO DE LA FUNCIÓN EN UN SELECT
SELECT
    producto_id,
    precio,
    dbo.fn_CategorizarPrecio(precio) AS Categoria -- Se usa como una columna más
FROM
    productos;
```

>> ¡ADVERTENCIA! Las funciones escalares pueden ser asesinas del rendimiento. Si usas una UDF escalar en un SELECT o WHERE sobre una tabla de 1 millón de filas, la función se ejecutará 1 millón de veces. Úsalas con extrema precaución.

---

## 16. Transacciones (TCL)

Una transacción es una unidad de trabajo "todo o nada". Garantiza que un bloque de comandos DML (INSERT, UPDATE, DELETE) se ejecute por completo o no se ejecute en absoluto.

**Analogía:** Es un carrito de compras. Agregas 5 productos a tu carrito (INSERT, UPDATE). Si en el último paso tu tarjeta de crédito falla, no te llevas ningún producto (la transacción se revierte). Solo cuando pagas con éxito (COMMIT), la compra es final.

Esto se rige por los principios ACID:

* Atomicidad (Atomicity): O todo se ejecuta, o nada se ejecuta.
* Consistencia (Consistency): La base de datos siempre queda en un estado válido.
* Isolamiento (Isolation): Las transacciones concurrentes no se interfieren entre sí.
* Durabilidad (Durability): Una vez que se hace COMMIT, los cambios son permanentes.

Sintaxis (con manejo de errores)

```sql
BEGIN TRY
    -- 1. Inicia la transacción
    BEGIN TRANSACTION;

    -- Operación 1: Sacar 100 de la cuenta A
    UPDATE cuentas_bancarias
    SET saldo = saldo - 100
    WHERE cuenta_id = 'A';

    -- Operación 2: Poner 100 en la cuenta B
    UPDATE cuentas_bancarias
    SET saldo = saldo + 100
    WHERE cuenta_id = 'B';

    -- 2. Si todo salió bien, confirma los cambios
    COMMIT TRANSACTION;
    PRINT 'Transferencia completada exitosamente.';

END TRY
BEGIN CATCH
    -- 3. Si algo falló, revierte TODOS los cambios
    ROLLBACK TRANSACTION;
    PRINT 'Error durante la transferencia. Cambios revertidos.';
    -- Aquí podrías registrar el error
END CATCH;
```

---

## 17. Índices (Indexes): El Arte de la Velocidad

Un índice es una estructura de datos que mejora la velocidad de las operaciones de búsqueda.

**Analogía:** Es el índice alfabético al final de un libro. Sin él, para encontrar el término "Window Functions" (tu WHERE), tendrías que leer el libro página por página (un Table Scan). Con el índice, vas directo a la página correcta (un Index Seek).

Tipos Principales

* Índice Agrupado (Clustered Index):
  * Define el orden físico en que se almacenan los datos en el disco.
  * **Analogía:** Una guía telefónica ordenada alfabéticamente por apellido. El índice es los datos.
  * Solo puede haber uno por tabla.
  * La PRIMARY KEY (Clave Primaria) es el Clustered Index por defecto.

* Índice No Agrupado (Non-Clustered Index):
  * Es una estructura separada que contiene los valores de la columna indexada y un "puntero" a la fila de datos real.
  * **Analogía:** El índice al final del libro. Está separado del contenido principal.
  * Puede haber muchos por tabla.

* El Costo: Los índices no son gratis.
  * Aceleran: SELECT (especialmente en WHERE y JOIN).
  * Ralentizan: INSERT, UPDATE, DELETE, porque la base de datos no solo tiene que modificar los datos, sino también actualizar todos los índices relacionados.

> Regla de Oro: Indexa las columnas que usas frecuentemente para filtrar (WHERE), unir (JOIN) y ordenar (ORDER BY). No indexes columnas que cambian constantemente o que tienen muy baja cardinalidad (ej. una columna de "Género" con solo 'M' y 'F').

---

## 18. Planes de Ejecución (Execution Plans)

Es el "mapa" que el motor de la base de datos (el Optimizador de Consultas) decide usar para ejecutar tu query.

**Analogía:** Es el "Google Maps" de tu consulta. Le pides ir del Punto A al Punto B (tu SELECT), y el optimizador decide si tomará la autopista (un Index Seek), calles secundarias (un Nested Loop Join) o un desvío por toda la ciudad (un Table Scan).

* Como analista, debes aprender a leer este plan para diagnosticar por qué una consulta es lenta.
  * En tu IDE de SQL (como SQL Server Management Studio), puedes "Mostrar el Plan de Ejecución Estimado".
  * Busca operaciones "costosas" (Coste Alto %):
    * Table Scan / Clustered Index Scan: ¡Alerta! Está leyendo la tabla entera. Probablemente falta un índice en tu WHERE.
    * Key Lookup: Una señal de que tu índice Non-Clustered no "cubre" toda la consulta y tiene que ir a buscar datos adicionales.
    * Sort: Operaciones de ordenamiento costosas. A veces inevitables, pero caras.

---

## 19. PIVOT y UNPIVOT: Rotando Datos

Estos operadores transforman los datos de filas a columnas (PIVOT) o de columnas a filas (UNPIVOT).

>>> Nota : Sinceramente, la sintaxis de PIVOT en SQL es... extraña. Y UNPIVOT es más útil. Dicho esto, Power Query es mil veces mejor, más fácil e intuitivo para estas dos operaciones. A menudo es mejor traer los datos "largos" (sin pivotar) a Power BI y pivotarlos allí.

* PIVOT (Filas a Columnas)

```sql
-- Queremos transformar esto:
-- Producto | Mes | Ventas
-- 'Leche'  | 'Ene' | 100
-- 'Leche'  | 'Feb' | 120
-- 'Pan'    | 'Ene' | 80

-- ...en esto:
-- Producto | Ene | Feb
-- 'Leche'  | 100 | 120
-- 'Pan'    | 80  | NULL

SELECT Producto, [Ene], [Feb]
FROM (
    SELECT Producto, Mes, Ventas
    FROM ventas_mensuales
) AS TablaFuente
PIVOT (
    SUM(Ventas) -- Qué agregar en las nuevas celdas
    FOR Mes IN ([Ene], [Feb]) -- Qué fila se convertirá en columnas
) AS TablaPivot;
```

* UNPIVOT (Columnas a Filas)
Esto es más común para "normalizar" datos que vienen de un Excel feo.

```sql
-- Queremos transformar esto:
-- Producto | Ene | Feb
-- 'Leche'  | 100 | 120

-- ...en esto:
-- Producto | Mes | Ventas
-- 'Leche'  | 'Ene' | 100
-- 'Leche'  | 'Feb' | 120

SELECT Producto, Mes, Ventas
FROM (
    SELECT Producto, Ene, Feb
    FROM ventas_pivotadas
) AS TablaFuente
UNPIVOT (
    Ventas -- Nombre de la nueva columna de valores
    FOR Mes IN (Ene, Feb) -- Nombres de las columnas que quieres "des-pivotar"
) AS TablaUnpivot;
```

---

## 20. El Plano: Modelado de Datos (OLTP vs. OLAP)

No todas las bases de datos se diseñan igual. Una base de datos para una app (como un e-commerce) es diferente a una base de datos para análisis (un Data Warehouse).

**Analogía:** Un **motor de Fórmula 1** (OLTP) está hecho para aceleraciones rápidas y cambios constantes (miles de pequeñas escrituras: "un cliente compró"). Un **motor de camión de carga** (OLAP) está hecho para potencia bruta y constante (una gran lectura: "dame el total de ventas del año").

### OLTP (Online Transaction Processing)

* **Propósito:** Soportar aplicaciones del día a día (CRM, ERP, e-commerce).
* **Prioridad:** Velocidad de **escritura** (`INSERT`, `UPDATE`).
* **Diseño:** **Normalizado (3NF)**.
  * **Normalización:** Es el arte de reducir la redundancia de datos.
  * **Analogía:** Guardas la dirección de un cliente en *una sola tabla* (`Clientes`). La tabla de `Pedidos` solo guarda el `Cliente_ID`. Si el cliente se muda, solo actualizas su dirección en *un* lugar.
  * **Resultado:** Muchas tablas pequeñas y relacionadas. Genial para la integridad, pero lento para los reportes (¡requiere muchos `JOINs`!)

### OLAP (Online Analytical Processing)

* **Propósito:** Soportar análisis de negocio y Business Intelligence (¡Hola, Power BI!).
* **Prioridad:** Velocidad de **lectura** (`SELECT` con agregaciones).
* **Diseño:** **Desnormalizado (Esquema de Estrella)**.

---

## 21. El Esquema de Estrella (Star Schema): Tu Mejor Amigo en Power BI

Este es el diseño de base de datos preferido para el análisis. Power BI está *optimizado* para funcionar con este modelo.

**Analogía:** Piensa en una galaxia. En el centro hay un "sol" masivo (la tabla de hechos) y a su alrededor orbitan planetas (las tablas de dimensiones).

### I. Tabla de Hechos (Fact Table)

* **El "Sol" (El Centro):** Almacena las **métricas** y **eventos** de negocio.
* **Qué contiene:** Números, medidas, KPIs. (Ej. `CantidadVendida`, `Monto`, `Costo`).
* **Claves:** Está llena de claves foráneas (ej. `Producto_ID`, `Cliente_ID`, `Fecha_ID`) que apuntan a las dimensiones.
* **Características:** Es "estrecha" (pocas columnas) y "larga" (millones o billones de filas).

### II. Tablas de Dimensión (Dimension Tables)

* **Los "Planetas" (El Contexto):** Almacenan el **contexto** descriptivo. Son el "quién, qué, dónde, cuándo, por qué".
* **Qué contienen:** Texto, atributos, categorías. (Ej. `NombreCliente`, `CategoriaProducto`, `RegionGeografica`, `NombreMes`).
* **Claves:** Tienen una clave primaria simple (ej. `Cliente_ID`).
* **Características:** Son "anchas" (muchas columnas) y "cortas" (pocas filas, ej. 50 estados, 1000 productos).

**¿Por qué funciona tan bien en Power BI?**

1. **Rendimiento:** Las agregaciones (`SUM`, `AVG`) son rapidísimas en la tabla de hechos.
2. **Simplicidad:** Los filtros y `SLICERS` provienen de las dimensiones (ej. filtrar por `CategoriaProducto`).
3. **Intuición:** Es fácil de entender para un analista. "Quiero ver las *Ventas* (Hecho) por *Región* (Dimensión)".

> **Tu Misión como Data Scientist/Analista:** Tu trabajo con SQL (CTEs, Vistas, SPs) es a menudo **transformar** un modelo OLTP (muchas tablas) en un Esquema de Estrella (OLAP) limpio para que Power BI pueda consumirlo.

---

## 22. SQL en el Mundo Moderno: Manejo de `JSON`

Hoy en día, muchos datos no vienen en tablas perfectas, sino como texto `JSON` desde APIs. Los motores de SQL modernos (SQL Server, PostgreSQL, Snowflake) pueden consultar `JSON` directamente.

**Analogía:** Es como si tu base de datos aprendiera a leer y descomprimir archivos `.zip` (el JSON) sobre la marcha.

### Funciones Clave (Ejemplos de T-SQL)

* `JSON_VALUE`: Extrae un valor **escalar** (texto, número) de un string JSON.

    ```sql
    -- JSON_Data = '{"nombre":"Juan", "edad":30, "ciudad":"Mexico"}'
    SELECT JSON_VALUE(JSON_Data, '$.nombre') AS Nombre -- Devuelve 'Juan'
    FROM logs;
    ```

* `JSON_QUERY`: Extrae un **objeto o array** (otro JSON) de un string JSON.

    ```sql
    -- JSON_Data = '{"user":{"nombre":"Ana"}, "tags":["sql", "pbi"]}'
    SELECT JSON_QUERY(JSON_Data, '$.user') AS Usuario -- Devuelve '{"nombre":"Ana"}'
    SELECT JSON_QUERY(JSON_Data, '$.tags') AS Tags   -- Devuelve '["sql", "pbi"]'
    ```

* `OPENJSON`: Expone un objeto o array JSON como un **conjunto de filas (una tabla)**.

    ```sql
    -- Esta es la más poderosa. Convierte un JSON en una tabla.
    DECLARE @json_doc VARCHAR(MAX) =
    '[
        {"id": 1, "producto": "Leche"},
        {"id": 2, "producto": "Pan"}
    ]';

    SELECT *
    FROM OPENJSON(@json_doc)
    WITH (
        ID_Producto INT '$.id',
        Nombre_Producto VARCHAR(100) '$.producto'
    );
    -- Resultado:
    -- ID_Producto | Nombre_Producto
    -- 1           | 'Leche'
    -- 2           | 'Pan'
    ```

---

## 23. Anti-Patrones: Lo Que NO Debes Hacer

La sabiduría no es solo saber qué hacer, sino qué *evitar*.

1. **Usar `SELECT *` en Producción:**
    * **Por qué no:** Es perezoso, desperdicia ancho de banda y puede romper el código si la tabla cambia. Siempre especifica las columnas (`SELECT col1, col2...`).
2. **Poner Funciones en la Columna (en el `WHERE`):**
    * **Mal:** `WHERE YEAR(fecha_pedido) = 2024`
    * **Por qué:** "Mata" el índice. Obliga a SQL a ejecutar la función `YEAR()` por *cada fila* de la tabla.
    * **Bien:** `WHERE fecha_pedido >= '2024-01-01' AND fecha_pedido < '2025-01-01'`
    * **Esto se llama SARGable** (Search ARGument-able). Permite a SQL usar el índice para "buscar" el rango.
3. **Abusar de los `Cursors`:**
    * **Qué es:** Un `Cursor` procesa los datos **fila por fila**.
    * **Por qué no:** ¡SQL es un lenguaje basado en **conjuntos** (sets)! Es como si para mover una caja de 1000 tornillos, los movieras de uno en uno en lugar de mover la caja entera. Es lentísimo. Casi todo lo que hace un cursor se puede hacer mejor con una `Window Function`.
4. **Usar `NVARCHAR` cuando `VARCHAR` es suficiente:**
    * `NVARCHAR` usa 2 bytes por carácter (para Unicode, ej. 🉐). `VARCHAR` usa 1 byte (para ASCII). Si solo guardas emails o códigos, usar `NVARCHAR` duplica el espacio de almacenamiento innecesariamente.


