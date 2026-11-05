# 🛒 Análisis de caída de ingresos en un E-commerce de moda (SQL Server + Power BI)

Proyecto de análisis de datos de punta a punta: a partir de una base transaccional en **SQL Server** se investiga por qué los **ingresos netos de septiembre 2026 cayeron cerca de 37 % respecto a agosto**, y se presentan los resultados en un **dashboard de Power BI**.

> **Nota:** el conjunto de datos es **sintético**. Fue generado para simular la operación de una tienda online peruana de ropa (moneda: soles, S/, con IGV incluido). No contiene información real de personas ni de empresas.

---

## 📌 Contexto del negocio

*ModaPerú* es una tienda 100 % online que vende polos, pantalones, calzado, accesorios y casacas. El equipo de gerencia plantea la situación como un encargo de análisis:

> Los ingresos netos de septiembre cayeron respecto a agosto. ¿Qué pasó, cuánto pesa cada causa y qué hacemos al respecto?

Pistas iniciales del negocio: agosto tuvo una promoción en la primera quincena, marketing ajustó su presupuesto en septiembre, operaciones mencionó "algunos problemas" a fin de mes y atención al cliente reportó más quejas.

## 🎯 Objetivos

1. Confirmar la caída con una definición clara de ingreso neto.
2. Ubicar dónde se concentra: canal, categoría, método de pago, dispositivo.
3. Cuantificar en soles el efecto de cada causa identificada.
4. Proponer recomendaciones priorizadas.

## 🧰 Herramientas

| Herramienta | Uso |
|---|---|
| SQL Server (T-SQL) | Modelo de datos, validaciones y consultas de análisis |
| Power BI | Dashboard de 3 páginas (DAX para las métricas) |
| Excel | Ejercicio inicial de análisis por canal |

---

## 🗃️ Conjunto de datos

Periodo: **1 de enero al 30 de septiembre de 2026**.

| Tabla | Descripción | Filas |
|---|---|---|
| `customers` | Clientes, región y canal de adquisición | 6,796 |
| `products` | Catálogo (60 productos, 5 categorías) con costo y precio | 60 |
| `orders` | Cabecera de órdenes: canal, dispositivo, método de pago, cupón, estado | 10,940 |
| `order_items` | Detalle de productos por orden | 18,339 |
| `payments` | Libro de transacciones: intentos de pago, comisiones y reembolsos (en negativo) | 12,404 |
| `returns` | Devoluciones a nivel de ítem, con motivo y monto reembolsado | 993 |
| `daily_traffic` | Embudo diario por canal: sesiones, carritos, checkout | 1,365 |
| `marketing_spend` | Inversión diaria por canal, impresiones y clics | 619 |
| `inventory_snapshots` | Foto semanal (lunes) del stock por producto | 2,340 |

### Modelo de datos

```mermaid
erDiagram
    customers ||--o{ orders : realiza
    orders ||--|{ order_items : contiene
    products ||--o{ order_items : "se vende en"
    orders ||--o{ payments : "se paga con"
    orders ||--o{ returns : origina
    order_items ||--o{ returns : "devuelve"
    products ||--o{ inventory_snapshots : "tiene stock"
```

Las tablas `daily_traffic` y `marketing_spend` se relacionan con `orders` por **fecha y canal**.

### Estados de una orden

- `Completed`: orden pagada y procesada.
- `Cancelled`: cancelada (motivo en `status_reason`).
- `Payment Failed`: el pago nunca se aprobó.

---

## ⚙️ Cómo reproducir el proyecto

1. Abre **SQL Server Management Studio** (o Azure Data Studio).
2. Ejecuta `sql/01_crear_esquema.sql`: crea la base `ModaPeru_Ecommerce` y las tablas.
3. Ejecuta `sql/02_cargar_datos.sql` (~4 MB): carga los datos y devuelve un conteo de filas por tabla para validar.
4. Abre el dashboard en Power BI y actualiza la conexión a tu servidor local.

---

## 📐 Definiciones de métricas

| Métrica | Definición |
|---|---|
| Total de órdenes | Órdenes creadas, sin importar su estado |
| Ventas completadas | Suma de `total_amount` de órdenes `Completed` |
| Reembolsos | Suma de `refund_amount` de la tabla `returns` (por mes de la compra) |
| Gasto de marketing | Suma de `spend_pen` de `marketing_spend` |
| Ingreso neto *(versión actual del dashboard)* | Ventas completadas − reembolsos − gasto de marketing |

**Notas y mejoras pendientes**

- El "ingreso neto" actual descuenta el gasto de marketing, que es un gasto operativo. La definición más estricta sería *ventas completadas − reembolsos*, dejando el resultado después de marketing como una métrica aparte.
- Las devoluciones ocurren entre 6 y 25 días después de la compra. Los reembolsos de las últimas semanas de septiembre están **incompletos**, por lo que septiembre se ve ligeramente mejor de lo que será.
- Un `JOIN` entre `orders` y `order_items` duplica `total_amount` en órdenes con varios ítems. Los ingresos por categoría deben calcularse desde `order_items`.

---

## 📊 Dashboard (Power BI)

El dashboard tiene tres páginas:

1. **Dato general:** KPIs (órdenes, ventas completadas, ingreso neto, reembolsos, gasto de marketing) y evolución mensual.
2. **Ingreso neto:** evolución y variación porcentual contra el mes anterior.
3. **Concentración:** detalle de septiembre por categoría, método de pago, estado del pago, dispositivo, gasto por canal y stock de productos.

<!-- Agrega aquí tus capturas, por ejemplo:
![Dato general](images/dashboard_general.png)
![Ingreso neto](images/dashboard_ingreso_neto.png)
![Concentración](images/dashboard_concentracion.png)
-->

---

## 🔎 Hallazgos

### Confirmados

- **Septiembre registra una caída de −36.95 %** en el ingreso neto frente a agosto (de S/ 341.98 mil a S/ 215.62 mil), la mayor variación negativa del año.
- **Cuatro productos de calzado quedaron sin stock** (Zapatilla Casual Beige, Classic Gris, Running Negra y Urbana Blanca), según las fotos semanales de inventario.
- El gasto de marketing bajó de S/ 35.88 mil a S/ 31.09 mil, y el canal Email muestra una inversión muy baja en septiembre.

### En investigación

- Magnitud en soles del quiebre de stock respecto a la caída total de ventas completadas.
- Patrón de **pagos fallidos con tarjeta** en septiembre, cruzado por dispositivo y fecha de inicio.
- Descomposición de la caída en **número de órdenes** vs. **ticket promedio**.
- Efecto del fin de la promoción de agosto y del cambio de mezcla de categorías.

### Tabla de causas *(por completar)*

| Causa | Ingreso perdido (S/) | % de la caída |
|---|---|---|
| Quiebre de stock | _pendiente_ | _pendiente_ |
| Pagos fallidos | _pendiente_ | _pendiente_ |
| Menor tráfico / marketing | _pendiente_ | _pendiente_ |
| Otros (promoción, mezcla de productos) | _pendiente_ | _pendiente_ |

---

## 💡 Recomendaciones *(por completar tras cuantificar)*

1. _Reabastecer los productos de mayor rotación y alertar antes de llegar a stock cero._
2. _Revisar la integración con la pasarela de pagos en tarjeta, especialmente en móvil._
3. _Reevaluar el recorte de inversión en canales con mejor retorno._

---

## 🧾 Consultas SQL de ejemplo

**Órdenes no completadas por mes y motivo**

```sql
SELECT
    MONTH(order_datetime) AS mes,
    status,
    status_reason,
    COUNT(*) AS total
FROM orders
WHERE status <> 'Completed'
GROUP BY MONTH(order_datetime), status, status_reason
ORDER BY mes, status;
```

**Validación de la lógica de reembolsos** (descuento prorrateado por ítem)

```sql
SELECT
    o.order_id,
    oi.order_item_id,
    r.refund_amount,
    ROUND(oi.quantity * oi.unit_price * (1 - o.discount_amount / o.subtotal), 2) AS reembolso_calculado
FROM orders o
JOIN order_items oi ON oi.order_id = o.order_id
JOIN returns r      ON r.order_item_id = oi.order_item_id;
```

**Órdenes y ticket promedio por mes**

```sql
SELECT
    MONTH(order_datetime) AS mes,
    COUNT(*) AS ordenes_creadas,
    SUM(CASE WHEN status = 'Completed' THEN total_amount ELSE 0 END) AS ventas_completadas,
    AVG(CASE WHEN status = 'Completed' THEN total_amount END)        AS ticket_promedio
FROM orders
GROUP BY MONTH(order_datetime)
ORDER BY mes;
```

---

## 📁 Estructura del repositorio

```
ecommerce-analisis-ventas/
├── README.md
├── sql/
│   ├── 01_crear_esquema.sql
│   ├── 02_cargar_datos.sql
│   └── 03_analisis.sql
├── dashboard/
│   └── ecommerce_dashboard.pbix
├── excel/
│   └── caso_ecommerce_trafico.xlsx
└── images/
    └── (capturas del dashboard)
```

## 🧠 Habilidades demostradas

- Modelado relacional y validación de integridad entre tablas.
- T-SQL: agregaciones, `JOIN`, `CASE`, segmentación y descomposición de métricas.
- Pensamiento analítico: formular hipótesis, segmentar antes de concluir y cuantificar el impacto.
- Visualización y storytelling de datos con Power BI.
- Documentación de supuestos y limitaciones de los datos.

---

## 👤 Autor

**[Tu nombre]** · [LinkedIn](https://www.linkedin.com/) · [correo@ejemplo.com](mailto:correo@ejemplo.com)
