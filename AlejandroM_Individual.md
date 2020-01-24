# Alumno 4:

## ORACLE:

### Tarea 1
----------------------------------------------------------------------
#### Crea un tablespace de undo e intenta crear una tabla en él.

Vamos a crear un tablespace (espacio de tablas), el cual contendrá la tabla que intentaremos crear.

Este tablespace lo llamaremos `tbs_undo_Tarea1` y será de tipo **undo**. Los tablespace de tipo undo se utilizan para retener los cambios que se realizan sobre los datos en Oracle y asi poder deshacerlos mientras no se hayan sobreescrito o acabado el tiempo de retención.

Ádemas, es necesario crear un `DATAFILE`. Estos son archivos físicos que almacenan los datos de todas las estructuras lógicas en la base de datos.
Nosotros vamos a crear un `DATAFILE` llamado `tbs_undo_Tarea1.f` con un tamaño de 5Mb y que se pueda extender.

Le indicamos la opción `RETENTION GUARANTEE`, que sirve para garantizar que oracle no sobreescribirá los datos, aunque se produzca un fallo.

###### Creamos el tablespace
```sql
CREATE UNDO TABLESPACE tbs_undo_Tarea1
     DATAFILE 'tbs_undo_Tarea1.f'
          SIZE 5M 
          AUTOEXTEND ON
     RETENTION GUARANTEE;
```
Ahora vamos a intentar crear la tabla en el tablespace `tbs_undo_Tarea1`.

###### Creamos la tabla
```sql
CREATE TABLE tab_Tarea1 (
          empno      NUMBER(5) PRIMARY KEY,
          ename      VARCHAR2(15) NOT NULL)
     TABLESPACE tbs_undo_Tarea1;
```
###### Error
```sql
ERROR en línea 1:
ORA-30022: No se pueden crear segmentos en un tablespace de deshacer
```

Al introducir la sentencia de `CREATE TABLE` nos damos cuenta que no se pueden introducir tablas en un tablespace de tipo **undo**. Esto se producé porque no se pueden crear segmentos en este, ya que no es un tablespace normal y no puede almacenar datos de usuarios normales, solo mantiene la información de los cambios de los datos.

##### Permisos necesarios

```sql
GRANT CREATE TABLESPACE TO moralg;
```

### Tarea 2
---------------------------------------------------------------
#### Crea un tablespace temporal TEMP2 y escribe una sentencia SQL que genere un script que haga usar TEMP2 a todos los usuarios que tienen USERS como tablespace por defecto.

En esta tarea vamos a crear un tablespace de tipo **temporary**; este contiene objetos de esquemas, que se almacenan en archivos temporales, que se crean o existen durante una sesión.

Al tablespace temporal lo llamaremos `tbs_temp_Tarea2` y como en el tablespace de la _tarea 1_, tenemos que asignarle un archivo físico pero esta vez será de tipo `TEMPFILE` (archivo fisico temporal).

Le indicamos que ocupe 5Mb y que se pueda extender.

```sql
CREATE TEMPORARY TABLESPACE tbs_temp_Tarea2
     TEMPFILE 'tbs_temp_Tarea2.f'
          SIZE 5M
          AUTOEXTEND ON;
```

Ya tenemos el tablespace temporal, ahora vamos a crear un script para asignar a todos los usuarios que esten utilizando el tablespace `USER` por defecto. 

Para realizar esto, tenemos que consultar en la vista `DBA_USERS`.

```sql
CREATE or REPLACE PROCEDURE ScriptAsignarTbs_temp_Tarea2
IS
     cursor c_Users is
     select USERNAME
     from DBA_USERS
     where DEFAULT_TABLESPACE='USERS';
BEGIN
     dbms_output.put_line(chr(9));
     dbms_output.put_line('---------------------------------------------------');
     for v_Usuario in c_Users loop
          dbms_output.put_line('ALTER USER '||v_Usuario.USERNAME||' TEMPORARY TABLESPACE tbs_temp_Tarea2;');
     end loop;
     dbms_output.put_line('---------------------------------------------------');
     dbms_output.put_line(chr(9));
END;
/
```

##### Prueba

```sql
SQL> exec ScriptAsignarTbs_temp_Tarea2
ALTER USER GSMCATUSER TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER MDDATA TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER SYSBACKUP TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER REMOTE_SCHEDULER_AGENT TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER GSMUSER TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER PRUEBAPASS TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER SYSRAC TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER AUDSYS TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER DIP TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER SYSKM TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER PACO TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER ORACLE_OCM TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER LUIS TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER PRUEBAPALOMA TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER SYSDG TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER SPATIAL_CSW_ADMIN_USR TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER MORALG TEMPORARY TABLESPACE tbs_temp_Tarea2;
ALTER USER JOSEMARI TEMPORARY TABLESPACE tbs_temp_Tarea2;

Procedimiento PL/SQL terminado correctamente.
```

##### Permisos necesarios

```sql
GRANT CREATE TABLESPACE TO moralg;
GRANT ALTER USER TO moralg;
GRANT EXECUTE ON ScriptAsignarTbs_temp_Tarea2 TO moralg;
```
Además en el procedimiento tendríamos que cambiar la vista `DBA_USERS` por `ALL_USERS`.

### Tarea 3
---------------------------------------------------------------

#### Borra todos los tablespaces creados para esta práctica sin que quede rastro de ellos. Realiza las acciones previas que sean necesarias.

Ahora tenemos que borrar los dos tablespaces creados anteriormente, pero hay un problema, que al borrar los tablespaces no se desvinculan los usuarios con los tablespaces elimninados. Para solucionar esto, antes de borrar el tablespace `tbs_temp_Tarea2`, tenemos que asignarle a los usuarios el tablespace temporal por defecto que es `TEMP`.

Vamos a crear un script que asigne el tablespace `TEMP` a los usuarios que tengan el tablespace `tbs_temp_Tarea2`

```sql
CREATE or REPLACE PROCEDURE ScriptAsignarTEMP
IS
     cursor c_Users is
     select USERNAME
     from DBA_USERS
     where TEMPORARY_TABLESPACE='TBS_TEMP_TAREA2';
BEGIN
     dbms_output.put_line(chr(9));
     dbms_output.put_line('---------------------------------------------------');
     for v_Usuario in c_Users loop
          dbms_output.put_line('ALTER USER '||v_Usuario.USERNAME||' TEMPORARY TABLESPACE TEMP;');
     end loop;
     dbms_output.put_line('---------------------------------------------------');
     dbms_output.put_line(chr(9));
END;
/
```

Ya tenemos el tablespace `tbs_temp_Tarea2` desvinculado de cualquier usuario, ahora vamos a eliminar los tablespaces.

Con las siguientes sentencias, eliminamos los tablespaces pero hay que añadir `including contents` para que se borre el contenido y `datafiles` para borrar también el archivo físico correspondiente.

```sql
drop tablespace tbs_temp_Tarea2 including contents and datafiles;
drop tablespace tbs_undo_Tarea1 including contents and datafiles;
```

##### Permisos necesarios

```sql
GRANT DROP TABLESPACE TO moralg;
GRANT ALTER USER TO moralg;
GRANT EXECUTE ON ScriptAsignarTEMP TO moralg;
```
Además en el procedimiento tendríamos que cambiar la vista `DBA_USERS` por `ALL_USERS`.

### Tarea 4
---------------------------------------------------------------

#### Averigua los segmentos existentes para realizar un ROLLBACK y el tamaño de sus extensiones.

Los segmento de rollback se utilizan para deshacer los cambios de las transacciones que no se ha hecho un commit, es decir, para poder volver atrás en caso de un fallo o error en la base de datos.

Las transacciones se asignan de manera automatica a un segmento de tipo rollback llamdo `SYSTEMT`. También se puede asignar a un segmento concreto con la sentencia `SET TRANSACTION USE ROLLBACK SEGMENT nombre_segmento_rollback`.

Para listar los segmentos existentes para realizar un rollback es necesario indicarle que muestre los segmentos de tipo `ROLLBACK` en la vista `DBA_SEGMENTS`.

```sql
select SEGMENT_NAME, EXTENT_ID, BYTES from DBA_EXTENTS where SEGMENT_TYPE='ROLLBACK';

SEGMENT_NAME    EXTENT_ID  BYTES
--------------- ---------- ----------
SYSTEMT         0          65536
SYSTEMT         1          65536
SYSTEMT         2          65536
SYSTEMT         3          65536
SYSTEMT         4          65536
SYSTEMT         5          65536
SYSTEMT         6          65536

7 filas seleccionadas.
```

------------------------------------_CORREGIR

select distinct SEGMENT_NAME from DBA_ROLLBACK_SEGS;


### Tarea 5
---------------------------------------------------------------

#### Queremos cambiar de ubicación un tablespace, pero antes debemos avisar a los usuarios que tienen acceso de lectura o escritura a cualquiera de los objetos almacenados en el mismo. Escribe un procedimiento llamado MostrarUsuariosAccesoTS que obtenga un listado con los nombres de dichos usuarios.

```sql
SELECT DISTINCT OWNER FROM DBA_SEGMENTS WHERE TABLESPACE_NAME='USERS'; 

     OWNER
     ----------
     PACO
     MORALG
```

```sql
select distinct OWNER, TABLE_NAME from DBA_INDEXES where TABLESPACE_NAME='USERS';
select distinct GRANTEE from DBA_TAB_PRIVS where TABLE_NAME='VIVIENDAS';

select distinct OWNER from DBA_CLUSTERS where TABLESPACE_NAME='USERS';

select distinct OWNER from DBA_INDEXES where TABLESPACE_NAME='USERS';
```

### Tarea 6
---------------------------------------------------------------

#### Realiza un procedimiento llamado MostrarInfoTabla que reciba el nombre de una tabla y muestre la siguiente información sobre la misma: propietario, usuarios que pueden leer sus datos, usuarios que pueden cambiar (insertar, modificar o eliminar) sus datos, usuarios que pueden modificar su estructura, usuarios que pueden eliminarla, lista de extensiones y en qué fichero de datos se encuentran.

```sql
set pagesize 999
set linesize 999

CREATE OR REPLACE PROCEDURE MostrarInfoTabla(p_Tabla VARCHAR2)
IS
BEGIN
     dbms_output.put_line(chr(9));
     MostrarPropietario(p_Tabla);
     dbms_output.put_line(chr(9));
     dbms_output.put_line('~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~');
END;
/

CREATE OR REPLACE PROCEDURE MostrarPropietario(p_Tabla VARCHAR2)
IS
     cursor c_Propietarios is
     select distinct OWNER
     from DBA_TABLES 
     where TABLE_NAME=p_Tabla;

     v_Propietario VARCHAR2(50);
BEGIN
     for v_Propietario in c_Propietarios loop
          dbms_output.put_line(chr(9));
          dbms_output.put_line('~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~');
          dbms_output.put_line(chr(9));
          dbms_output.put_line('OWNER: '||v_Propietario.OWNER);
          MostrarUserSelect(p_Tabla, v_Propietario.OWNER);
          MostartUserDML(p_Tabla, v_Propietario.OWNER);
          MostrarUserAlter(p_Tabla, v_Propietario.OWNER);
          MostrarUserDrop(p_Tabla, v_Propietario.OWNER);
          MostrarExt(p_Tabla, v_Propietario.OWNER);
     end loop;
END;
/

CREATE OR REPLACE PROCEDURE MostrarUserSelect(p_Tabla VARCHAR2,
                                              p_Propietario VARCHAR2)
IS
     cursor c_Users is
     select distinct GRANTEE 
     from DBA_TAB_PRIVS 
     where TABLE_NAME=p_Tabla
     and PRIVILEGE='SELECT'
     and OWNER=p_Propietario;
BEGIN
     for v_Usuario in c_Users loop
          dbms_output.put_line(chr(9));
          dbms_output.put_line('USERS PRIVILEGES READ: ');
          dbms_output.put_line(chr(9)||'-'||v_Usuario.GRANTEE);
     end loop;
END; 
/

CREATE OR REPLACE PROCEDURE MostartUserDML(p_Tabla VARCHAR2,
                                           p_Propietario VARCHAR2)
IS
     cursor c_Users is
     select distinct GRANTEE 
     from DBA_TAB_PRIVS 
     where TABLE_NAME=p_Tabla
     and OWNER=p_Propietario 
     and (PRIVILEGE='INSERT' or PRIVILEGE='UPDATE' or PRIVILEGE='DELETE');
BEGIN
     for v_Usuario in c_Users loop
          dbms_output.put_line(chr(9));
          dbms_output.put_line('USERS PRIVILEGES DML:');
          dbms_output.put_line(chr(9)||'-'||v_Usuario.GRANTEE);
     end loop;
END;
/

CREATE OR REPLACE PROCEDURE MostrarUserAlter(p_Tabla VARCHAR2,
                                             p_Propietario VARCHAR2)
IS
     cursor c_Users is
     select distinct GRANTEE 
     from DBA_TAB_PRIVS 
     where TABLE_NAME=p_Tabla 
     and PRIVILEGE='ALTER'
     and OWNER=p_Propietario;
BEGIN
     for v_Usuario in c_Users loop
          dbms_output.put_line(chr(9));
          dbms_output.put_line('USER PRIVILEGES MODIFY:');
          dbms_output.put_line(chr(9)||'-'||v_Usuario.GRANTEE);
     end loop;
END;
/

CREATE OR REPLACE PROCEDURE MostrarUserDrop(p_Tabla VARCHAR2,
                                            p_Propietario VARCHAR2)
IS
     cursor c_Users is
     select distinct GRANTEE 
     from DBA_TAB_PRIVS 
     where TABLE_NAME=p_Tabla 
     and PRIVILEGE='DROP'
     and OWNER=p_Propietario;
BEGIN
     for v_Usuario in c_Users loop
          dbms_output.put_line(chr(9));
          dbms_output.put_line('USER PRIVILEGES REMOVE:');
          dbms_output.put_line(chr(9)||'-'||v_Usuario.GRANTEE);
     end loop;
END;
/

CREATE OR REPLACE PROCEDURE MostrarExt(p_Tabla VARCHAR2,
                                       p_Propietario VARCHAR2)
IS
     v_File VARCHAR2(50);
     cursor c_Extens is
     select seg.EXTENTS, seg.INITIAL_EXTENT, seg.NEXT_EXTENT, seg.MIN_EXTENTS, seg.MAX_EXTENTS, ex.FILE_ID
     from DBA_SEGMENTS seg, DBA_EXTENTS ex
     where seg.SEGMENT_NAME = ex.SEGMENT_NAME 
     and seg.SEGMENT_NAME=p_Tabla
     and seg.OWNER=p_Propietario;
BEGIN
     dbms_output.put_line(chr(9));
     dbms_output.put_line(RPAD('Nº Extensión',13)||' '||RPAD('Inicio',8)||' '||RPAD('Siguiente',10)||' '||RPAD('Mínimo',7)||' '||RPAD('Máximo',13)||' '||RPAD('Fichero',40));
     dbms_output.put_line(RPAD('-------------------------------',13)||' '||RPAD('-------------------------------',8)||' '||RPAD('-------------------------------',10)||' '||RPAD('-------------------------------',7)||' '||RPAD('-------------------------------',13)||' '||RPAD('-------------------------------',40));
     for v_Extension in c_Extens loop
          v_File:=MostrarFicheroExt(v_Extension.FILE_ID);
          dbms_output.put_line(RPAD(v_Extension.EXTENTS,13)||' '||RPAD(v_Extension.INITIAL_EXTENT,8)||' '||RPAD(v_Extension.NEXT_EXTENT,10)||' '||RPAD(v_Extension.MIN_EXTENTS,7)||' '||RPAD(v_Extension.MAX_EXTENTS,13)||' '||RPAD(v_File,40));
     end loop;
     
END;
/

CREATE OR REPLACE FUNCTION MostrarFicheroExt(p_FileID VARCHAR2)
RETURN VARCHAR2
IS
     v_FileName VARCHAR2(50);
BEGIN
     select FILE_NAME into v_FileName
     from DBA_DATA_FILES 
     where FILE_ID=p_FileID;

     return v_FileName;
END;
/
```

## Postgres:

### Tarea 7        
---------------------------------------------------------------

#### Averigua si pueden establecerse claúsulas de almacenamiento para las tablas o los espacios de tablas en Postgres.

PostgreSQL es un gestor de base de datos donde no hay cláusulas de almacenamiento de datos como en Oracle pero hay una función que nos permite controlar el espacio de las tablas e índices. 

La función de la que estoy hablando es `pg_total_relation_size`, con la que se puede controlar el espacio usado por un tabla, incluyendo indices y tablas TOAST. La sintaxis es:

```sql
CREATE VIEW user_disk_usage AS SELECT r.rolname, SUM(pg_total_relation_size(c.oid)) AS total_disk_usage 
FROM pg_class c, pg_roles r 
WHERE c.relkind = 'r' 
AND c.relowner = r.oid 
GROUP BY c.relowner;
```

Como podemos ver, la el proceso de control de espacio de almacenamiento se crea con una vista y utilizando la función `pg_total_relation_size`.

------------------------EXPLICACIÓN DE LA FUNCION

##### Inconvenientes

* Como no pasa en Oracle, las transacciones se abortan si se encuentran con algun fallo durante la ejecución.
* Es muy simple el soporte que aporta PostgreSQL.


## MySQL:

### Tarea 8
---------------------------------------------------------------

#### Averigua si existe el concepto de índice en MySQL y si coincide con el existente en ORACLE. Explica los distintos tipos de índices existentes.



## MongoDB:

### Tarea 9
---------------------------------------------------------------

#### Explica los distintos motores de almacenamiento que ofrece MongoDB, sus características principales y en qué casos es más recomendable utilizar cada uno de ellos.