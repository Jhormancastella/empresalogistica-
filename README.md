## creacion de tablas 
```sql
-- Crear tabla de países
CREATE TABLE paises (
    pais_id INT PRIMARY KEY,
    nombre_pais VARCHAR(100)
);

-- Crear tabla de ciudades
CREATE TABLE ciudades (
    ciudad_id INT PRIMARY KEY,
    nombre_ciudad VARCHAR(100),
    pais_id INT,
    FOREIGN KEY (pais_id) REFERENCES paises(pais_id)
);

-- Crear tabla de sucursales
CREATE TABLE sucursales (
    sucursal_id INT PRIMARY KEY,
    nombre_sucursal VARCHAR(100),
    direccion VARCHAR(255),
    ciudad_id INT,
    FOREIGN KEY (ciudad_id) REFERENCES ciudades(ciudad_id)
);

-- Crear tabla de conductores
CREATE TABLE conductores (
    conductor_id INT PRIMARY KEY,
    nombre_conductor VARCHAR(100),
    telefono VARCHAR(15),
    sucursal_id INT,
    FOREIGN KEY (sucursal_id) REFERENCES sucursales(sucursal_id)
);

-- Crear tabla de auxiliares
CREATE TABLE auxiliares (
    auxiliar_id INT PRIMARY KEY,
    nombre_auxiliar VARCHAR(100),
    telefono VARCHAR(15),
    sucursal_id INT,
    FOREIGN KEY (sucursal_id) REFERENCES sucursales(sucursal_id)
);

-- Crear tabla de vehículos
CREATE TABLE vehiculos (
    vehiculo_id INT PRIMARY KEY,
    placa VARCHAR(20),
    modelo VARCHAR(100),
    sucursal_id INT,
    FOREIGN KEY (sucursal_id) REFERENCES sucursales(sucursal_id)
);

-- Crear tabla de clientes
CREATE TABLE clientes (
    cliente_id INT PRIMARY KEY,
    nombre_cliente VARCHAR(100),
    correo VARCHAR(100),
    direccion VARCHAR(255),
    telefono VARCHAR(15)
);

-- Crear tabla de paquetes
CREATE TABLE paquetes (
    paquete_id INT PRIMARY KEY,
    numero_seguimiento VARCHAR(50),
    peso DECIMAL(10,2),
    dimensiones VARCHAR(50),
    contenido VARCHAR(255),
    valor_declarado DECIMAL(10,2),
    destino VARCHAR(100),
    tipo_servicio ENUM('Nacional', 'Internacional', 'Express', 'Estandar'),
    sucursal_id INT,
    FOREIGN KEY (sucursal_id) REFERENCES sucursales(sucursal_id)
);

-- Crear tabla de envíos
CREATE TABLE envios (
    envio_id INT PRIMARY KEY,
    cliente_id INT,
    paquete_id INT,
    sucursal_id INT,
    fecha_envio DATETIME,
    estado ENUM('Recibido', 'En tránsito', 'Entregado', 'Retenido en aduana'),
    ruta_id INT,
    numero_seguimiento VARCHAR(50),
    FOREIGN KEY (cliente_id) REFERENCES clientes(cliente_id),
    FOREIGN KEY (paquete_id) REFERENCES paquetes(paquete_id),
    FOREIGN KEY (sucursal_id) REFERENCES sucursales(sucursal_id)
);

-- Crear tabla de rutas
CREATE TABLE rutas (
    ruta_id INT PRIMARY KEY,
    nombre_ruta VARCHAR(100),
    conductor_id INT,
    vehiculo_id INT,
    FOREIGN KEY (conductor_id) REFERENCES conductores(conductor_id),
    FOREIGN KEY (vehiculo_id) REFERENCES vehiculos(vehiculo_id)
);

-- Crear tabla de seguimiento
CREATE TABLE seguimiento (
    seguimiento_id INT PRIMARY KEY,
    paquete_id INT,
    ubicacion VARCHAR(255),
    fecha_evento DATETIME,
    estado ENUM('Recibido', 'En tránsito', 'Entregado', 'Retenido en aduana'),
    FOREIGN KEY (paquete_id) REFERENCES paquetes(paquete_id)
);

-- Crear tabla de asignaciones_rutas
CREATE TABLE asignaciones_rutas (
    asignacion_id INT PRIMARY KEY,
    ruta_id INT,
    auxiliar_id INT,
    FOREIGN KEY (ruta_id) REFERENCES rutas(ruta_id),
    FOREIGN KEY (auxiliar_id) REFERENCES auxiliares(auxiliar_id)
);

-- Crear tabla de empleados
CREATE TABLE empleados (
    id INT PRIMARY KEY,
    nombre VARCHAR(100),
    salario DECIMAL(10,2),
    departamento_id INT,
    puesto VARCHAR(50),
    sucursal_id INT,
    FOREIGN KEY (sucursal_id) REFERENCES sucursales(sucursal_id)
);

-- Crear tabla de ventas
CREATE TABLE ventas (
    id INT PRIMARY KEY,
    vendedor_id INT,
    producto VARCHAR(100),
    total_venta DECIMAL(10,2),
    fecha DATETIME,
    sucursal_id INT,
    FOREIGN KEY (sucursal_id) REFERENCES sucursales(sucursal_id)
);
```
## consultas 

```sql
-- 1. Consulta básica para ver todos los envíos de un cliente específico
SELECT 
    e.envio_id,
    e.numero_seguimiento,
    p.contenido,
    e.fecha_envio,
    e.estado
FROM envios e
JOIN clientes c ON e.cliente_id = c.cliente_id
JOIN paquetes p ON e.paquete_id = p.paquete_id
WHERE c.nombre_cliente = 'Nombre del Cliente';

-- 2. Rastrear un paquete por número de seguimiento
SELECT 
    s.fecha_evento,
    s.ubicacion,
    s.estado
FROM seguimiento s
JOIN paquetes p ON s.paquete_id = p.paquete_id
WHERE p.numero_seguimiento = 'ABC123'
ORDER BY s.fecha_evento DESC;

-- 3. Reporte de envíos por sucursal
SELECT 
    suc.nombre_sucursal,
    COUNT(e.envio_id) as total_envios,
    SUM(p.valor_declarado) as valor_total_envios
FROM sucursales suc
LEFT JOIN envios e ON suc.sucursal_id = e.sucursal_id
LEFT JOIN paquetes p ON e.paquete_id = p.paquete_id
GROUP BY suc.sucursal_id, suc.nombre_sucursal;

-- 4. Buscar conductores y sus rutas asignadas
SELECT 
    c.nombre_conductor,
    c.telefono,
    r.nombre_ruta,
    v.placa as vehiculo
FROM conductores c
LEFT JOIN rutas r ON c.conductor_id = r.conductor_id
LEFT JOIN vehiculos v ON r.vehiculo_id = v.vehiculo_id;

-- 5. Encontrar paquetes retenidos en aduana
SELECT 
    p.numero_seguimiento,
    p.contenido,
    p.valor_declarado,
    c.nombre_cliente,
    c.telefono
FROM paquetes p
JOIN envios e ON p.paquete_id = e.paquete_id
JOIN clientes c ON e.cliente_id = c.cliente_id
WHERE e.estado = 'Retenido en aduana';

-- 6. Reporte de ventas por sucursal y fecha
SELECT 
    s.nombre_sucursal,
    DATE(v.fecha) as fecha,
    COUNT(v.id) as total_ventas,
    SUM(v.total_venta) as monto_total
FROM ventas v
JOIN sucursales s ON v.sucursal_id = s.sucursal_id
GROUP BY s.sucursal_id, s.nombre_sucursal, DATE(v.fecha)
ORDER BY fecha DESC;

-- 7. Rendimiento de empleados por sucursal
SELECT 
    s.nombre_sucursal,
    e.nombre as empleado,
    e.puesto,
    COUNT(v.id) as total_ventas,
    SUM(v.total_venta) as ventas_totales
FROM empleados e
LEFT JOIN ventas v ON e.id = v.vendedor_id
JOIN sucursales s ON e.sucursal_id = s.sucursal_id
GROUP BY s.sucursal_id, s.nombre_sucursal, e.id, e.nombre, e.puesto;

-- 8. Consulta de paquetes por tipo de servicio
SELECT 
    tipo_servicio,
    COUNT(*) as cantidad_paquetes,
    AVG(peso) as peso_promedio,
    SUM(valor_declarado) as valor_total
FROM paquetes
GROUP BY tipo_servicio;

-- 9. Encontrar clientes frecuentes (más de 5 envíos)
SELECT 
    c.nombre_cliente,
    c.correo,
    COUNT(e.envio_id) as total_envios,
    SUM(p.valor_declarado) as valor_total_enviado
FROM clientes c
JOIN envios e ON c.cliente_id = e.cliente_id
JOIN paquetes p ON e.paquete_id = p.paquete_id
GROUP BY c.cliente_id, c.nombre_cliente, c.correo
HAVING COUNT(e.envio_id) > 5
ORDER BY total_envios DESC;

-- 10. Consulta de eficiencia de rutas
SELECT 
    r.nombre_ruta,
    c.nombre_conductor,
    COUNT(DISTINCT e.envio_id) as total_envios,
    COUNT(DISTINCT ar.auxiliar_id) as total_auxiliares
FROM rutas r
JOIN conductores c ON r.conductor_id = c.conductor_id
LEFT JOIN asignaciones_rutas ar ON r.ruta_id = ar.ruta_id
LEFT JOIN envios e ON r.ruta_id = e.ruta_id
GROUP BY r.ruta_id, r.nombre_ruta, c.nombre_conductor;

-- 11. Consulta avanzada de seguimiento con información completa
SELECT 
    e.numero_seguimiento,
    c.nombre_cliente,
    c.telefono,
    p.contenido,
    p.peso,
    p.tipo_servicio,
    s.fecha_evento,
    s.ubicacion,
    s.estado,
    suc.nombre_sucursal as sucursal_origen
FROM envios e
JOIN clientes c ON e.cliente_id = c.cliente_id
JOIN paquetes p ON e.paquete_id = p.paquete_id
JOIN seguimiento s ON p.paquete_id = s.paquete_id
JOIN sucursales suc ON e.sucursal_id = suc.sucursal_id
WHERE e.numero_seguimiento = 'ABC123'
ORDER BY s.fecha_evento DESC;
```
