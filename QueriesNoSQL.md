# Consultas NoSQL para Sistema Bancario en MongoDB

A continuación se presentan 5 enunciados de consultas basados en las colecciones NoSQL del sistema bancario, junto con las soluciones utilizando operaciones avanzadas de MongoDB.

## 1. Análisis de Saldos por Tipo de Cuenta

**Enunciado:** El departamento financiero necesita un informe que muestre el saldo total, promedio, máximo y mínimo por cada tipo de cuenta (ahorro y corriente) para evaluar la distribución de fondos en el banco.

**Consulta MongoDB:**
```javascript
db.clientes.aggregate([
  { $unwind: "$cuentas" },
  {
    $group: {
      _id: "$cuentas.tipo_cuenta",
      saldo_total: { $sum: "$cuentas.saldo" },
      saldo_promedio: { $avg: "$cuentas.saldo" },
      saldo_maximo: { $max: "$cuentas.saldo" },
      saldo_minimo: { $min: "$cuentas.saldo" }
    }
  },
  {
    $project: {
      tipo_cuenta: "$_id",
      _id: 0,
      saldo_total: 1,
      saldo_promedio: 1,
      saldo_maximo: 1,
      saldo_minimo: 1
    }
  }
]);
```

## 2. Patrones de Transacciones por Cliente

**Enunciado:** El equipo de análisis de comportamiento necesita identificar los patrones de transacciones de cada cliente, mostrando la cantidad y el monto total de transacciones por tipo (depósito, retiro, transferencia) para cada cliente.

**Consulta MongoDB:**
```javascript
db.transacciones.aggregate([
  {
    $group: {
      _id: { cliente: "$cliente_ref", tipo: "$tipo_transaccion" },
      total: { $sum: "$monto" },
      cantidad: { $sum: 1 }
    }
  }
]);
```

## 3. Clientes con Múltiples Tarjetas de Crédito

**Enunciado:** El departamento de riesgo crediticio necesita identificar a los clientes que poseen más de una tarjeta de crédito, mostrando sus datos personales, cantidad de tarjetas y el detalle de cada una.

**Consulta MongoDB:**
```javascript
db.clientes.find({
  "cuentas.tarjetas.tipo_tarjeta": "credito"
},
{
  nombre: 1,
  cedula: 1,
  correo: 1,
  direccion: 1,
  "cuentas.tarjetas.$": 1
});
```

## 4. Análisis de Medios de Pago más Utilizados

**Enunciado:** El departamento de marketing necesita conocer cuáles son los medios de pago más utilizados para depósitos, agrupados por mes, para orientar sus campañas promocionales.

**Consulta MongoDB:**
```javascript
db.transacciones.aggregate([
  { $match: { tipo_transaccion: "deposito" } },
  {
    $project: {
      fecha_real: { $toDate: "$fecha" },
      medio_pago: "$detalles_deposito.medio_pago"
    }
  },
  {
    $project: {
      anio_mes: { 
        $dateToString: { format: "%Y-%m", date: "$fecha_real" }
      },
      medio_pago: 1
    }
  },
  {
    $group: {
      _id: {
        mes: "$anio_mes",
        medio: "$medio_pago"
      },
      cantidad: { $sum: 1 }
    }
  },
  {
    $sort: { "_id.mes": 1, cantidad: -1 }
  }
]);
```

## 5. Detección de Cuentas con Transacciones Sospechosas

**Enunciado:** El departamento de seguridad necesita identificar cuentas con patrones de transacciones sospechosas, definidas como aquellas que tienen más de 3 retiros en un mismo día con un monto total superior a 1,000,000 COP.

**Consulta MongoDB:**
```javascript
db.transacciones.aggregate([
  { 
    $match: { tipo_transaccion: "retiro" } 
  },
  {
    $project: {
      num_cuenta: 1,
      monto: 1,
      fecha_dia: { $substr: ["$fecha", 0, 10] }
    }
  },
  {
    $group: {
      _id: { cuenta: "$num_cuenta", dia: "$fecha_dia" },
      total_monto: { $sum: "$monto" },
      cantidad_retiros: { $sum: 1 }
    }
  },
  {
    $match: {
      cantidad_retiros: { $gt: 3 },
      total_monto: { $gt: 1000000 }
    }
  },
  {
    $sort: { "_id.dia": 1 }
  }
]);
```
