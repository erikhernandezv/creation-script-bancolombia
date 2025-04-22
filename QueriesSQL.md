# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
SELECT 
    c.id_cliente,
    c.nombre,
    COUNT(ct.num_cuenta) AS cantidad_cuentas,
    SUM(ct.saldo) AS saldo_total
FROM 
    Cliente c
JOIN 
    Cuenta ct ON c.id_cliente = ct.id_cliente
GROUP BY 
    c.id_cliente, c.nombre
HAVING 
    COUNT(ct.num_cuenta) > 1
ORDER BY 
    saldo_total DESC;
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
SELECT 
    cl.id_cliente,
    cl.nombre,
    COALESCE(SUM(CASE WHEN d.id_transaccion IS NOT NULL THEN t.monto END), 0) AS total_depositos,
    COALESCE(SUM(CASE WHEN r.id_transaccion IS NOT NULL THEN t.monto END), 0) AS total_retiros
FROM 
    Cliente cl
JOIN 
    Cuenta c ON cl.id_cliente = c.id_cliente
JOIN 
    Transaccion t ON c.num_cuenta = t.num_cuenta
LEFT JOIN 
    Deposito d ON t.id_transaccion = d.id_transaccion
LEFT JOIN 
    Retiro r ON t.id_transaccion = r.id_transaccion
GROUP BY 
    cl.id_cliente, cl.nombre
ORDER BY 
    cl.nombre;
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
SELECT 
    c.num_cuenta,
    c.id_cliente,
    cl.nombre,
    c.saldo
FROM 
    Cuenta c
JOIN 
    Cliente cl ON c.id_cliente = cl.id_cliente
LEFT JOIN 
    Tarjeta t ON c.num_cuenta = t.num_cuenta
WHERE 
    t.num_cuenta IS NULL
ORDER BY 
    c.num_cuenta;
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
SELECT 
    c.tipo_cuenta,
    COUNT(DISTINCT c.num_cuenta) AS cantidad_cuentas,
    AVG(c.saldo) AS saldo_promedio
FROM 
    Cuenta c
JOIN 
    Transaccion t ON c.num_cuenta = t.num_cuenta
WHERE 
    t.fecha >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY 
    c.tipo_cuenta
ORDER BY 
    c.tipo_cuenta;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
SELECT DISTINCT cl.id_cliente, cl.nombre, cl.correo
FROM Cliente cl
JOIN Cuenta c ON cl.id_cliente = c.id_cliente
JOIN Transaccion t ON c.num_cuenta = t.num_cuenta
JOIN Transferencia tf ON t.id_transaccion = tf.id_transaccion
WHERE cl.id_cliente NOT IN (
    SELECT DISTINCT cl2.id_cliente
    FROM Cliente cl2
    JOIN Cuenta c2 ON cl2.id_cliente = c2.id_cliente
    JOIN Transaccion t2 ON c2.num_cuenta = t2.num_cuenta
    JOIN Retiro r ON t2.id_transaccion = r.id_transaccion
    WHERE r.canal = 'cajero'
)
ORDER BY cl.nombre;
```