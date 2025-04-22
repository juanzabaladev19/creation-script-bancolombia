# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
SELECT 
    Cliente.id_cliente,
    Cliente.nombre,
    COUNT(Cuenta.num_cuenta) AS total_cuentas,
    SUM(Cuenta.saldo) AS saldo_total
FROM 
    Cliente
JOIN 
    Cuenta ON Cliente.id_cliente = Cuenta.id_cliente
GROUP BY 
    Cliente.id_cliente, Cliente.nombre
HAVING 
    COUNT(Cuenta.num_cuenta) > 1
ORDER BY 
    saldo_total DESC;
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
SELECT 
    c.id_cliente,
    c.nombre,
    (SELECT SUM(t1.monto)
     FROM Cuenta ct1
     JOIN Transaccion t1 ON ct1.num_cuenta = t1.num_cuenta
     WHERE ct1.id_cliente = c.id_cliente AND t1.tipo_transaccion = 'deposito') AS total_depositos,
     
    (SELECT SUM(t2.monto)
     FROM Cuenta ct2
     JOIN Transaccion t2 ON ct2.num_cuenta = t2.num_cuenta
     WHERE ct2.id_cliente = c.id_cliente AND t2.tipo_transaccion = 'retiro') AS total_retiros
FROM 
    Cliente c
ORDER BY 
    total_depositos DESC;
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
SELECT 
    ct.num_cuenta,
    ct.id_cliente,
    c.nombre
FROM 
    Cuenta ct
JOIN 
    Cliente c ON ct.id_cliente = c.id_cliente
LEFT JOIN 
    Tarjeta t ON ct.num_cuenta = t.num_cuenta
WHERE 
    t.num_cuenta IS NULL;
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
SELECT 
    ct.tipo_cuenta,
    AVG(ct.saldo) AS saldo_promedio
FROM 
    Cuenta ct
JOIN 
    Transaccion t ON ct.num_cuenta = t.num_cuenta
WHERE 
    t.fecha >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY 
    ct.tipo_cuenta;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
SELECT DISTINCT c.id_cliente, c.nombre, c.correo
FROM Cliente c
JOIN Cuenta ct ON c.id_cliente = ct.id_cliente
JOIN Transaccion t ON ct.num_cuenta = t.num_cuenta
WHERE t.tipo_transaccion = 'transferencia'
  AND c.id_cliente NOT IN (
    SELECT c2.id_cliente
    FROM Cliente c2
    JOIN Cuenta ct2 ON c2.id_cliente = ct2.id_cliente
    JOIN Transaccion t2 ON ct2.num_cuenta = t2.num_cuenta
    JOIN Retiro r ON t2.id_transaccion = r.id_transaccion
    WHERE r.canal = 'cajero'
  );
```
