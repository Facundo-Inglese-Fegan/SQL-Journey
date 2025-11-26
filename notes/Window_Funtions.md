# Window Funtions

**Son funciones que realizan cálculos a través de un conjunto de filas relacionadas con la fila actual. Se utilizan para comparar la fila actual con totales de grupo, filas anteriores/posteriores, o generar rankings sin necesidad de subqueries complejas o self-joins.**

- **Diferencia clave con** GROUP BY: El GROUP BY colapsa/reduce las filas (pierdes el detalle). Las Window Functions mantienen las filas originales y agregan una columna calculada con contexto del grupo.

## Sintaxis base

Funcion()
    OVER (PARTITION BY [Grupo]
        ORDER BY [Secuencia] [Frame]
    )

- Funcion(): La operación a realizar (SUM, AVG, RANK, LAG). Es obligatorio. Es la herramienta.
- OVER: Abre la "ventana" de cálculo. Es obligatorio. Es el banco de trabajo.
- PARTITION BY: Reinicia el cálculo cuando cambia este valor. Define el "grupo". No es obligatorio. En  caso de no colocarse, los cálculos se hacen sobre toda la tabla. Es el muro de contención que separa "Peras" de "Manzanas". Siempre hay que colocarlo si hay varios usuarios/grupos.
- ORDER BY: Define el orden lógico para procesar las filas dentro de la partición. A veces es obligatorio (Vital para Rank/Lag). Es la fila india.
- ROWS: Define cuántas filas mirar atrás o adelante (Frame). No es obligatorio. Es la mirilla.

## Tipos de funciones

### Funciones de ranking (Jerarquía)

Requieren ORDER BY obligatorio.

- ROW_NUMBER(): Asigna un secuencial único (1, 2, 3...). Si hay empate, decide arbitrariamente (no determinista) a menos que agregues más criterios de orden.
- RANK(): Ranking de competencia (Olímpico). Si hay empate en el puesto 1, el siguiente es el 3. Deja huecos.
- DENSE_RANK(): Ranking compacto. Si hay empate en el puesto 1, el siguiente es el 2. NO deja huecos.
- NTILE(n): Divide la partición en n grupos (cubos) lo más equitativos posible. (Ej: Deciles, Cuartiles).

### Funciones de offset (Desplazamiento/Tiempo)

Permiten acceder a datos de otras filas sin hacer JOIN.

- LAG(col, n, default): Mira n filas hacia atrás. Clave para crecimientos (MoM, YoY).
- LEAD(col, n, default): Mira n filas hacia adelante.
- FIRST_VALUE(col): Devuelve el primer valor de la partición.
- LAST_VALUE(col): Devuelve el último valor (¡Cuidado con el frame por defecto aquí!).

### Funciones de agregación (Matemática)

- SUM(), AVG(), MIN(), MAX(), COUNT().
- Comportamiento clave:
  - Si usas ORDER BY dentro del OVER: Calcula Running Totals (Acumulado hasta la fila actual).
  - Si NO usas ORDER BY: Calcula el Total del Grupo (el mismo valor para todas las filas de la partición).

## El Frame

Controla con precisión quirúrgica qué filas entran en el cálculo.

- UNBOUNDED PRECEDING: Desde el inicio de la partición.
- n PRECEDING: n filas antes de la actual.
- CURRENT ROW: La fila actual.
- n FOLLOWING: n filas después de la actual.
- UNBOUNDED FOLLOWING: Hasta el final de la partición.

## Buenas prácticas

1) El Muro del WHERE: No puedes filtrar por el resultado de una Window Function en la misma consulta (WHERE rank_id = 1 falla).
    - Solución: Debes envolver la query en una CTE o Subquery y filtrar afuera.
2) Partición Fantasma: Si olvidas el PARTITION BY, la función corre sobre TODA la tabla. En tablas de millones de filas, esto puede matar el rendimiento del servidor.
3) Determinismo: En ROW_NUMBER, si ordenas por una columna que tiene duplicados (ej: fecha), el orden de los empatados es aleatorio.
    - Solución: Siempre agrega una columna única al final del ORDER BY (ej: ORDER BY fecha, user_id) para asegurar consistencia.
4) Nulls en LAG/LEAD: Siempre define el tercer argumento (valor por defecto, usualmente 0) para evitar NULLs que rompan restas o divisiones posteriores.

## Ejemplos

**Escenario de Negocio:** "Queremos identificar a los mejores vendedores por región, pero tenemos empates. Muéstrame las diferentes formas de asignarles un puesto."

```sql
SELECT 
    region,
    sales_rep,
    total_sales,

    -- A. ROW_NUMBER: Desempate forzado (útil para paginación o "eliminar duplicados")
    -- Resultado: 1, 2, 3, 4 (Aunque el 2 y 3 tengan las mismas ventas)
    ROW_NUMBER() OVER (
        PARTITION BY region 
        ORDER BY total_sales DESC
    ) as row_num,

    -- B. RANK: Competición Olímpica (salta puestos en empates)
    -- Resultado: 1, 2, 2, 4
    RANK() OVER (
        PARTITION BY region 
        ORDER BY total_sales DESC
    ) as rank_olimpico,

    -- C. DENSE_RANK: Compacto (no salta puestos)
    -- Resultado: 1, 2, 2, 3 (Ideal para "Top 3 mejores montos")
    DENSE_RANK() OVER (
        PARTITION BY region 
        ORDER BY total_sales DESC
    ) as dense_rank,

    -- D. NTILE: Segmentación (Cuartiles/Deciles)
    -- Divide a los vendedores en 4 grupos: 1=Top Performers, 4=Low Performers
    NTILE(4) OVER (
        PARTITION BY region 
        ORDER BY total_sales DESC
    ) as sales_quartile

FROM sales_table;
```

**Escenario de Negocio:** "Calcula el crecimiento mes a mes (MoM) de nuestros ingresos y cuánto tiempo pasa entre una compra y la siguiente."

```sql
SELECT 
    month_date,
    revenue,

    -- A. LAG: Mirar hacia ATRÁS (Previo)
    -- Trae el revenue del mes anterior. '0' es el valor default si no hay mes previo.
    LAG(revenue, 1, 0) OVER (
        ORDER BY month_date
    ) as prev_month_revenue,

    -- CÁLCULO DE CRECIMIENTO (MoM %)
    -- Fórmula: (Actual - Anterior) / Anterior
    (revenue - LAG(revenue, 1, 0) OVER (ORDER BY month_date)) 
    / NULLIF(LAG(revenue, 1, 0) OVER (ORDER BY month_date), 0) as mom_growth_rate,

    -- B. LEAD: Mirar hacia ADELANTE (Futuro)
    -- ¿Cuál es la fecha de la SIGUIENTE compra de este cliente?
    LEAD(month_date, 1) OVER (
        ORDER BY month_date
    ) as next_purchase_date,
    
    -- C. FIRST_VALUE: Comparar contra el inicio
    -- ¿Cuánto ha crecido desde el PRIMER mes registrado en la historia?
    FIRST_VALUE(revenue) OVER (
        ORDER BY month_date
    ) as initial_revenue

FROM monthly_kpis;
```

**Escenario de Negocio:** "Necesito ver la venta de cada transacción, pero comparada contra el total que gastó ese cliente en todo su historial, y también su acumulado hasta el momento."

```sql
SELECT 
    user_id,
    tx_date,
    amount,

    -- A. TOTAL DEL GRUPO (Sin ORDER BY)
    -- "Suma todo lo que este usuario ha gastado en su vida".
    -- El valor se repite idéntico en todas las filas del usuario.
    SUM(amount) OVER (
        PARTITION BY user_id
    ) as lifetime_spend,

    -- B. RUNNING TOTAL (Con ORDER BY)
    -- "Suma acumulada transacción a transacción".
    -- El valor crece en cada fila.
    SUM(amount) OVER (
        PARTITION BY user_id
        ORDER BY tx_date
    ) as running_balance,
    
    -- C. PORCENTAJE DEL TOTAL (% of Total)
    -- ¿Qué peso tiene esta compra sobre el total del cliente?
    amount / SUM(amount) OVER (PARTITION BY user_id) as pct_of_lifetime

FROM transactions;
```

**Escenario de Negocio:** "Los datos diarios tienen mucho ruido. Dame una media móvil de 7 días (la semana actual y los 6 días previos) para ver la tendencia real."

```sql
SELECT 
    date,
    daily_users,

    -- A. MEDIA MÓVIL (Moving Average)
    -- Promedia la fila actual + las 6 anteriores.
    AVG(daily_users) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg_7d,

    -- B. ACUMULADO DESDE INICIO (Explícito)
    -- Es lo mismo que el default, pero escrito explícitamente.
    SUM(daily_users) OVER (
        ORDER BY date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as cumulative_users,
    
    -- C. VENTANA CENTRADA (Raro, pero preguntan)
    -- Promedio de ayer, hoy y mañana.
    AVG(daily_users) OVER (
        ORDER BY date
        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
    ) as centered_avg

FROM traffic_table;
```
