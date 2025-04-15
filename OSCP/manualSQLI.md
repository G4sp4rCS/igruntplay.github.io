# Manual SQL injection

## Ventajas de la inyección sql a mano
- Es más facil de bypasear los filtros de seguridad como WAF o IDS
- obtenemos más control sobre la inyección

[CheatSheets de inyecciones SQL manuales](https://pentestmonkey.net/category/cheat-sheet)

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