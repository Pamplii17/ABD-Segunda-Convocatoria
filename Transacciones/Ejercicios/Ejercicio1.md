Vamos a crear un ejercicio nuevo y diferente, donde la lógica de las transacciones y las operaciones varíe significativamente respecto a los anteriores.

### Ejercicio Nuevo

**Ejercicio 5**

(CONSIDERAREMOS LA INICIALIZACIÓN DE LAS TABLAS COMO UNA TRANSACCIÓN T0)

```sql
CREATE TABLE empleados (
    id NVARCHAR(4) PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    salario DECIMAL(10, 2) NOT NULL
);

INSERT INTO empleados (id, nombre, salario) VALUES
('E01', 'Alice', 3000.00),
('E02', 'Bob', 2500.00),
('E03', 'Charlie', 2000.00);

COMMIT;
```

### Pasos

| Paso | Sesión 1 (serializable)      | Sesión 2 (read committed)  | Sesión 3 (read committed)  |
|------|------------------------------|----------------------------|----------------------------|
| 10   | --T1                         |                            |                            |
|      | SELECT nombre, salario FROM  |                            |                            |
|      | empleados WHERE id='E01';    |                            |                            |
| 11   |                              | --T2                       |                            |
|      |                              | SELECT * FROM              |                            |
|      |                              | empleados WHERE salario < 2500;|                      |
| 12   |                              |                            | --T3                       |
|      |                              |                            | UPDATE empleados           |
|      |                              |                            | SET salario = salario * 1.10 WHERE salario < 2500; |
| 13   | INSERT INTO empleados        |                            |                            |
|      | (id, nombre, salario)        |                            |                            |
|      | VALUES ('E04', 'Diana', 2200.00); |                      |                            |
| 14   | SELECT * FROM                |                            |                            |
|      | empleados WHERE id='E04';    |                            |                            |
| 15   |                              | UPDATE empleados           |                            |
|      |                              | SET salario = 2600.00      |                            |
|      |                              | WHERE id='E02';            |                            |
| 16   | COMMIT;                      |                            |                            |
| 17   |                              |                            | SET TRANSACTION            |
|      |                              |                            | ISOLATION LEVEL            |
|      |                              |                            | READ COMMITTED;            |
| 18   |                              | --T4                       |                            |
|      |                              | DELETE FROM empleados      |                            |
|      |                              | WHERE id = 'E03';          |                            |
| 19   |                              |                            |                            |
|      |                              | COMMIT;                    |                            |
| 20   |                              |                            | SELECT * FROM empleados;   |
| 21   |                              |                            | COMMIT;                    |
| 22   | SELECT * FROM                |                            |                            |
|      | empleados WHERE salario > 2500; |                         |                            |
| 23   | COMMIT;                      |                            |                            |

### Solución

Para analizar la tabla de transacciones y las versiones de la fila existentes en los pasos 15 y 18, sigue esta estructura:

**Paso 15:**

**Transacciones**

| Transacción | Estado | Aislamiento    | ts_inicio | ts_fin |
|-------------|--------|----------------|-----------|--------|
| *T0*        | Commit | No definido    | 0         | 0      |
| T1          | Activa | Serializable   | 10        | --     |
| T2          | Activa | Read-committed | 11        | --     |
| T3          | Activa | Read-committed | 12        | --     |

**Versiones**

| Id  | Nombre  | Salario | ts_inicio | Borrada |
|-----|---------|---------|-----------|---------|
| E01 | Alice   | 3000.00 | 0         | --      |
| E02 | Bob     | 2500.00 | 0         | --      |
| E03 | Charlie | 2000.00 | 0         | --      |
| E04 | Diana   | 2200.00 | 10        | --      |
| E02 | Bob     | 2600.00 | 11        | --      |

**Paso 18:**

**Transacciones**

| Transacción | Estado | Aislamiento    | ts_inicio | ts_fin |
|-------------|--------|----------------|-----------|--------|
| *T0*        | Commit | No definido    | 0         | 0      |
| T1          | Commit | Serializable   | 10        | 16     |
| T2          | Activa | Read-committed | 11        | --     |
| T3          | Activa | Read-committed | 12        | --     |
| T4          | Activa | Read-committed | 18        | - o 18*|

**Versiones**

| Id  | Nombre  | Salario | ts_inicio | Borrada |
|-----|---------|---------|-----------|---------|
| E01 | Alice   | 3000.00 | 0         | --      |
| E02 | Bob     | 2500.00 | 0         | --      |
| E03 | Charlie | 2000.00 | 0         | 18      |
| E04 | Diana   | 2200.00 | 10        | --      |
| E02 | Bob     | 2600.00 | 11        | --      |
| E03 | Charlie | 2200.00 | 12        | --      |

*Nota: El paso 18 se ejecuta exitosamente porque no hay conflictos de escritura. La fila se elimina correctamente.

Voy a convertir este ejercicio en un PDF.

He creado el ejercicio completamente nuevo y lo he convertido en un PDF. Puedes descargar el archivo desde el siguiente enlace:
