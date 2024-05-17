# Clonado y Time travel

En el siguiente ejercicio aprenderemos a:

- Clonar una tabla y un esquema.
- Volver al estado anterior de una tabla gracias al *Time Travel* de Snowflake.

En esta práctica seguiremos utilizando los datos que ingestamos en el apartado de Ejercicio 1- Cargando Datos.

# 1. Cloning

## a) Clonado de tabla

Por ejemplo, si queremos clonar la tabla *addresses*  del esquema *bronze* en una tabla con nombre, pongamos, *addresses_clonado*, debemos ejecutar el siguiente comando:

```sql
USE DATABASE dev_curso_brz_db_alumno_<tu numero>;
USE SCHEMA bronze;
CREATE OR REPLACE TABLE addresses_clonado CLONE addresses;
```

Al hacer el clonado podemos observar que ambas tablas son idénticas:

![image](https://github.com/javipo84/Curso_Snowflake/assets/166698078/f0ba487f-ab9d-496a-8ac6-dd69f3093dfe)

## b) Clonado de esquema

De la misma forma, podemos clonar un esquema. Por ejemplo, podemos clonar el esquema *BRONZE*

```sql
CREATE OR REPLACE SCHEMA bronze_clonado CLONE bronze;
```

Usando este comando podemos observar que el esquema nuevo tiene la misma estructura que el original:

![image](https://github.com/javipo84/Curso_Snowflake/assets/166698078/81f08796-6d4f-455e-b513-8c77a67be205)

# 2. Time travel

## a) Time travel por tiempo

El Time Travel en Snowflake es una poderosa funcionalidad que permite a los usuarios acceder a datos históricos dentro de un marco de tiempo específico, lo cual es extremadamente útil para una variedad de casos de uso en el análisis de datos y la gestión de bases de datos. Aquí enseñaremos brevemente cómo podemos usar esta funcionalidad.

Seguiremos usando el schema de “bronze” y la tabla de “addresses” para hacer las pruebas.

Vamos a insertar un nuevo registro en la tabla *addresses* y visualizamos la tabla:

```sql
USE SCHEMA bronze;
INSERT INTO addresses VALUES(10,'Marca',null, CURRENT_TIMESTAMP());
SELECT * FROM addresses;
```

Ahora, borraremos el valor que le acabamos de insertar y mediante el uso de Time travel recuperaremos la versión inicial de la tabla con el registro insertado:

```sql
DELETE FROM addresses WHERE addresses_id=10;
SELECT * FROM addresses;
```
Observamos cómo era la tabla hace 2 minutos:

```sql
SELECT * FROM addresses AT (OFFSET => -60*2);

-- También se puede hacer de esta forma, la cual le resta dos minutos al tiempo actual:
SELECT * FROM addresses AT (TIMESTAMP => TIMESTAMPADD(MINUTE, -2, CURRENT_TIMESTAMP()));
```

Esta versión "antigua" se puede guardar con el siguiente comando:

```sql
CREATE OR REPLACE TABLE addresses_clone CLONE addresses at (TIMESTAMP => TIMESTAMPADD(minute, -5, CURRENT_TIMESTAMP()));
```

*Nota: es posible que hayan pasado los dos minutos y la tabla vuelva a estar como está en la actualidad. Podemos volver a la versión anterior añadiendo más minutos a la función de TIMESTAMPADD(minute, -5, CURRENT_TIMESTAMP()) por ejemplo, si queremos que vuelva a la de hace 10 minutos, la función será TIMESTAMPADD(minute, -10, CURRENT_TIMESTAMP()).*

Siguiendo con el ejercicio, podemos borrar la tabla y recuperarla también con el time travel de Snowflake:

```sql
-- Borramos la tabla
drop table brands;
-- Vemos que nuestra tabla está borrada
select * from brands;
-- Recuperamos la tabla borrada
undrop table brands;
-- Vemos que nuestra tabla ha vuelto
select * from brands;
-- Insertamos de nuevo el valor brand_id=10
insert into brands values(10,'Marca',null, CURRENT_TIMESTAMP());
-- Y podemos seguir usando las versiones anteriores de la tabla.
-- Como se puede ver, no aparece el nuevo registro que acabamos de insertar
-- si hago el time travel a hace 5 minutos:
select * from brands at (TIMESTAMP => TIMESTAMPADD(minute, -5, CURRENT_TIMESTAMP()));

```

## a) Time travel por id de la consulta

También podemos utilizar el time travel con el id de la consulta. 

En ese caso: 

```sql
-- Truncamos los valores para limpiar la tabla pero dejar 
-- la misma estructura que la tabla de brands
TRUNCATE TABLE Addresses_clonado;
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7e0dcde1-5a03-4ce7-a56f-1497c72c368f/e60c8841-dfed-40da-8ea0-23d7d33c314a/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7e0dcde1-5a03-4ce7-a56f-1497c72c368f/f7e9a189-20a8-4cb7-bcb3-0525fde1fcaa/Untitled.png)

```sql
-- Utilizamos el id de la query para volver al estado anterior a esta consulta
CREATE or replace TABLE Addresses_clonado AS SELECT * FROM Addresses_clonado BEFORE (STATEMENT => '01b38ced-0103-a99d-0000-185509e4503e');
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7e0dcde1-5a03-4ce7-a56f-1497c72c368f/74a43f95-f830-47b4-b48b-72a8ac822c9d/Untitled.png)

```sql
select * from Addresses_clonado;
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/7e0dcde1-5a03-4ce7-a56f-1497c72c368f/284e3927-fe18-4f39-b531-5b7f55b694c3/Untitled.png)
