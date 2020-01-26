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

###### Creamos el tablespace
```sql
CREATE TEMPORARY TABLESPACE tbs_temp_Tarea2
     TEMPFILE 'tbs_temp_Tarea2.f'
          SIZE 5M
          AUTOEXTEND ON;
```

Ya tenemos el tablespace temporal, ahora vamos a crear un script para asignar a todos los usuarios que esten utilizando el tablespace `USERS` por defecto. 

Para realizar esto, tenemos que consultar en la vista `DBA_USERS`.

###### Script para asignar el tablespace `TBS_TEMP_TAREA2` a los usuario con el tablespace `USERS`
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

###### Script para asignar tablespace por defecto a los usuarios del tablespace `TBS_TEMP_TAREA2` 
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

###### Eliminando los tablespaces
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

Para listar los segmentos existentes para realizar un rollback es necesario llamar vista `DBA_ROLLBACK_SEGS` y para mostrar la información de las extensiones la vista `DBA_EXTENTS`.

###### Consulta de segmentos de tipo rollback y sus extensiones 
```sql
select b.SEGMENT_NAME, a.EXTENT_ID, a.BYTES 
from DBA_EXTENTS a, DBA_ROLLBACK_SEGS b 
where a.SEGMENT_NAME=b.SEGMENT_NAME;
```

###### Nos devulve lo siguiente:
```sql
SEGMENT_NAME			EXTENT_ID  BYTES
------------------------ ---------- ----------
SYSTEM                   0          65536
SYSTEM                   1          65536
SYSTEM                   2          65536
SYSTEM                   3          65536
SYSTEM                   4          65536
SYSTEM                   5          65536
SYSTEM                   6          65536
_SYSSMU1_762089623$      0          65536
_SYSSMU1_762089623$      1          65536
_SYSSMU1_762089623$      2          1048576
_SYSSMU1_762089623$      3          1048576
_SYSSMU1_762089623$      4          1048576
_SYSSMU1_762089623$      5          65536
_SYSSMU1_762089623$      6          65536
_SYSSMU2_3062791661$     0          65536
_SYSSMU2_3062791661$     1          65536
_SYSSMU2_3062791661$     2          1048576
_SYSSMU2_3062791661$     3          1048576
_SYSSMU2_3062791661$     4          65536
_SYSSMU3_1499641855$     0          65536
_SYSSMU3_1499641855$     1          65536
_SYSSMU3_1499641855$     2          1048576
_SYSSMU3_1499641855$     3          1048576
_SYSSMU3_1499641855$     4          1048576
_SYSSMU3_1499641855$     5          1048576
_SYSSMU4_3564003469$     0          65536
_SYSSMU4_3564003469$     1          65536
_SYSSMU4_3564003469$     2          1048576
_SYSSMU4_3564003469$     3          1048576
_SYSSMU4_3564003469$     4          1048576
_SYSSMU4_3564003469$     5          65536
_SYSSMU4_3564003469$     6          65536
_SYSSMU5_1728379857$     0          65536
_SYSSMU5_1728379857$     1          65536
_SYSSMU5_1728379857$     2          1048576
_SYSSMU6_965511687$      0          65536
_SYSSMU6_965511687$      1          65536
_SYSSMU6_965511687$      2          1048576
_SYSSMU6_965511687$      3          1048576
_SYSSMU6_965511687$      4          1048576
_SYSSMU7_2247632671$     0          65536
_SYSSMU7_2247632671$     1          65536
_SYSSMU7_2247632671$     2          1048576
_SYSSMU7_2247632671$     3          1048576
_SYSSMU7_2247632671$     4          1048576
_SYSSMU8_437891266$      0          65536
_SYSSMU8_437891266$      1          65536
_SYSSMU8_437891266$      2          65536
_SYSSMU8_437891266$      3          65536
_SYSSMU8_437891266$      4          65536
_SYSSMU8_437891266$      5          65536
_SYSSMU8_437891266$      6          65536
_SYSSMU8_437891266$      7          65536
_SYSSMU8_437891266$      8          65536
_SYSSMU8_437891266$      9          65536
_SYSSMU8_437891266$      10         65536
_SYSSMU8_437891266$      11         65536
_SYSSMU8_437891266$      12         65536
_SYSSMU8_437891266$      13         65536
_SYSSMU8_437891266$      14         1048576
_SYSSMU8_437891266$      15         65536
_SYSSMU8_437891266$      16         65536
_SYSSMU9_3215744559$     0          65536
_SYSSMU9_3215744559$     1          65536
_SYSSMU9_3215744559$     2          65536
_SYSSMU9_3215744559$     3          1048576
_SYSSMU9_3215744559$     4          1048576
_SYSSMU10_2925533193$    0          65536
_SYSSMU10_2925533193$    1          65536
_SYSSMU10_2925533193$    2          1048576
_SYSSMU10_2925533193$    3          1048576
_SYSSMU10_2925533193$    4          1048576
_SYSSMU11_4115543019$    0          65536
_SYSSMU11_4115543019$    1          65536
_SYSSMU12_2802753883$    0          65536
_SYSSMU12_2802753883$    1          65536
_SYSSMU13_4242710331$    0          65536
_SYSSMU13_4242710331$    1          65536
_SYSSMU14_56631763$      0          65536
_SYSSMU14_56631763$      1          65536
_SYSSMU15_61031811$      0          65536
_SYSSMU15_61031811$      1          65536
_SYSSMU16_4063985861$    0          65536
_SYSSMU16_4063985861$    1          65536
_SYSSMU17_914417578$     0          65536
_SYSSMU17_914417578$     1          65536
_SYSSMU18_3387250440$    0          65536
_SYSSMU18_3387250440$    1          65536
_SYSSMU19_3783430934$    0          65536
_SYSSMU19_3783430934$    1          65536
_SYSSMU20_1894689580$    0          65536
_SYSSMU20_1894689580$    1          65536
_SYSSMU21_1087759902$    0          65536
_SYSSMU21_1087759902$    1          65536
_SYSSMU22_750563217$     0          65536
_SYSSMU22_750563217$     1          65536
_SYSSMU23_3484668115$    0          65536
_SYSSMU23_3484668115$    1          65536
_SYSSMU24_512801220$     0          65536
_SYSSMU24_512801220$     1          65536
_SYSSMU25_665916585$     0          65536
_SYSSMU25_665916585$     1          65536

102 filas seleccionadas.
```

### Tarea 5
---------------------------------------------------------------

#### Queremos cambiar de ubicación un tablespace, pero antes debemos avisar a los usuarios que tienen acceso de lectura o escritura a cualquiera de los objetos almacenados en el mismo. Escribe un procedimiento llamado MostrarUsuariosAccesoTS que obtenga un listado con los nombres de dichos usuarios.

```sql
CREATE OR REPLACE PROCEDURE MostrarUsuariosAccesoTS(p_Tablespace VARCHAR2)
IS
     cursor c_Usuarios is
     select USERNAME
     from DBA_USERS 
     where USERNAME in (select distinct OWNER
                        from DBA_TABLES 
                        where TABLESPACE_NAME=p_Tablespace)
     or USERNAME in (select distinct GRANTEE
                     from DBA_TAB_PRIVS
                     where TABLE_NAME in (select TABLE_NAME
                                          from DBA_TABLES
                                          where TABLESPACE_NAME=p_Tablespace))
     or USERNAME in (select distinct GRANTEE 
                     from DBA_SYS_PRIVS 
                     where PRIVILEGE='ALTER ANY TABLE'
                     or PRIVILEGE='READ ANY TABLE'
                     or PRIVILEGE='SELECT ANY TABLE'
                     or PRIVILEGE='DROP ANY TABLE'
                     or PRIVILEGE='DELETE ANY TABLE'
                     or PRIVILEGE='UPDATE ANY TABLE'
                     or PRIVILEGE='INSERT ANY TABLE')
     or USERNAME in (select distinct GRANTEE
                     from DBA_ROLE_PRIVS
                     where GRANTED_ROLE in (select distinct ROLE
                                            from ROLE_TAB_PRIVS
                                            where TABLE_NAME in (select TABLE_NAME
                                                                 from DBA_TABLES
                                                                 where TABLESPACE_NAME=p_Tablespace))
                     or GRANTED_ROLE in (select distinct ROLE
                                         from ROLE_SYS_PRIVS
                                         where PRIVILEGE='ALTER ANY TABLE'
                                         or PRIVILEGE='READ ANY TABLE'
                                         or PRIVILEGE='SELECT ANY TABLE'
                                         or PRIVILEGE='DROP ANY TABLE'
                                         or PRIVILEGE='DELETE ANY TABLE'
                                         or PRIVILEGE='UPDATE ANY TABLE'
                                         or PRIVILEGE='INSERT ANY TABLE')
                     or GRANTED_ROLE in (select distinct GRANTEE
                                         from DBA_ROLE_PRIVS
                                         start with GRANTED_ROLE in (select distinct ROLE
                                                                     from DBA_ROLES 
                                                                     where ROLE in (select ROLE
                                                                               from ROLE_TAB_PRIVS
                                                                               where TABLE_NAME in (select TABLE_NAME
                                                                                                    from DBA_TABLES
                                                                                                    where TABLESPACE_NAME=p_Tablespace))
                                                                     or ROLE in (select ROLE
                                                                                 from ROLE_SYS_PRIVS
                                                                                 where PRIVILEGE='ALTER ANY TABLE'
                                                                                 or PRIVILEGE='READ ANY TABLE'
                                                                                 or PRIVILEGE='SELECT ANY TABLE'
                                                                                 or PRIVILEGE='DROP ANY TABLE'
                                                                                 or PRIVILEGE='DELETE ANY TABLE'
                                                                                 or PRIVILEGE='UPDATE ANY TABLE'
                                                                                 or PRIVILEGE='INSERT ANY TABLE'))
                                         connect by GRANTED_ROLE = prior GRANTEE))
     or USERNAME in (select OWNER 
                     from DBA_INDEXES 
                     where TABLESPACE_NAME=p_Tablespace)
     or USERNAME in (select distinct OWNER 
                     from DBA_CLUSTERS where 
                     TABLESPACE_NAME=p_Tablespace);
BEGIN
     dbms_output.put_line(chr(9));
     dbms_output.put_line('Hay que informar a los usuarios:');
     for v_Usuarios in c_Usuarios loop
          dbms_output.put_line('-'||v_Usuarios.USERNAME);
     end loop;
END;
/
```

```sql
SQL> exec MostrarUsuariosAccesoTS('USERS');
	
Hay que informar a los usuarios:
-SYS
-SYSTEM
-USUARIOPRUEBA
-GSMUSER
-GGSYS
-GSMADMIN_INTERNAL
-MDSYS
-OLAPSYS
-PACO
-LUIS
-USUARIO1
-WMSYS
-MORALG
-NUEVOUSUARIO

Procedimiento PL/SQL terminado correctamente.
```

### Tarea 6
---------------------------------------------------------------

#### Realiza un procedimiento llamado MostrarInfoTabla que reciba el nombre de una tabla y muestre la siguiente información sobre la misma: propietario, usuarios que pueden leer sus datos, usuarios que pueden cambiar (insertar, modificar o eliminar) sus datos, usuarios que pueden modificar su estructura, usuarios que pueden eliminarla, lista de extensiones y en qué fichero de datos se encuentran.

* `ALTER ANY TABLE`
* `INSERT ANY TABLE`
* `UPDATE ANY TABLE`
* `DELETE ANY TABLE`
* `DROP ANY TABLE`
* `READ ANY TABLE`
* `SELECT ANY TABLE`

```sql
set pagesize 999
set linesize 999

CREATE OR REPLACE PROCEDURE MostrarInfoTabla(p_Tabla VARCHAR2)
IS
     cursor c_Propietarios is
     select distinct OWNER
     from DBA_TABLES 
     where TABLE_NAME=p_Tabla;

     v_Propietario VARCHAR2(50);
BEGIN
     for v_Propietario in c_Propietarios loop
          MostrarPropietario(p_Tabla, v_Propietario.OWNER);
          MostrarUserPrivsREAD(p_Tabla, v_Propietario.OWNER);
          MostartUserDML(p_Tabla, v_Propietario.OWNER);
          MostrarUserAlter(p_Tabla, v_Propietario.OWNER);
          MostrarUserDrop(p_Tabla, v_Propietario.OWNER);
          MostrarExt(p_Tabla, v_Propietario.OWNER);
     end loop;
     dbms_output.put_line(chr(9));
     dbms_output.put_line('~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~');
END;
/

CREATE OR REPLACE PROCEDURE MostrarPropietario(p_Tabla VARCHAR2,
                                               p_Propietario VARCHAR2)
IS
BEGIN
     dbms_output.put_line(chr(9));
     dbms_output.put_line('~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~');
     dbms_output.put_line(chr(9));
     dbms_output.put_line('OWNER: '||p_Propietario);
END;
/

CREATE OR REPLACE PROCEDURE MostrarUserPrivsREAD(p_Tabla VARCHAR2,
                                                 p_Propietario VARCHAR2)
IS
     cursor c_Usuarios is
     select USERNAME
     from DBA_USERS 
     where USERNAME in (select distinct GRANTEE
                        from DBA_TAB_PRIVS
                        where TABLE_NAME in (select TABLE_NAME
                                             from DBA_TABLES
                                             where TABLE_NAME=p_Tabla)
                        and PRIVILEGE='SELECT'
                        and OWNER=p_Propietario)
     or USERNAME in (select distinct GRANTEE 
                from DBA_SYS_PRIVS 
                where PRIVILEGE='READ ANY TABLE'
                or PRIVILEGE='SELECT ANY TABLE')
     or USERNAME in (select distinct GRANTEE
                     from DBA_ROLE_PRIVS
                     where GRANTED_ROLE in (select distinct ROLE
                                            from ROLE_TAB_PRIVS
                                            where TABLE_NAME in (select TABLE_NAME
                                                                 from DBA_TABLES
                                                                 where TABLE_NAME=p_Tabla)
                                            and PRIVILEGE='SELECT'
                                            and OWNER=p_Propietario)
                     or GRANTED_ROLE in (select distinct ROLE
                                         from ROLE_SYS_PRIVS
                                         where PRIVILEGE='READ ANY TABLE'
                                         or PRIVILEGE='SELECT ANY TABLE')
                     or GRANTED_ROLE in (select distinct GRANTEE
                                         from DBA_ROLE_PRIVS
                                         start with GRANTED_ROLE in (select distinct ROLE
                                                                     from DBA_ROLES 
                                                                     where ROLE in (select ROLE
                                                                                    from ROLE_TAB_PRIVS
                                                                                    where TABLE_NAME in (select TABLE_NAME
                                                                                                         from DBA_TABLES
                                                                                                         where TABLE_NAME=p_Tabla)
                                                                                    and PRIVILEGE='SELECT'
                                                                                    and OWNER=p_Propietario)
                                                                     or ROLE in (select ROLE
                                                                                 from ROLE_SYS_PRIVS
                                                                                 where PRIVILEGE='READ ANY TABLE'
                                                                                 or PRIVILEGE='SELECT ANY TABLE'))
                                         connect by GRANTED_ROLE = prior GRANTEE));
BEGIN
     dbms_output.put_line(chr(9));
     dbms_output.put_line('USERS PRIVILEGES READ: ');
     for v_Usuarios in c_Usuarios loop
          dbms_output.put_line('-'||v_Usuarios.USERNAME);
     end loop;
END;
/


CREATE OR REPLACE PROCEDURE MostartUserDML(p_Tabla VARCHAR2,
                                           p_Propietario VARCHAR2)
IS
     cursor c_Usuarios is
     select USERNAME
     from DBA_USERS 
     where USERNAME in (select distinct GRANTEE
                        from DBA_TAB_PRIVS
                        where TABLE_NAME in (select TABLE_NAME
                                             from DBA_TABLES
                                             where TABLE_NAME=p_Tabla)
                        and (PRIVILEGE='INSERT' or PRIVILEGE='UPDATE' or PRIVILEGE='DELETE')
                        and OWNER=p_Propietario)
     or USERNAME in (select distinct GRANTEE 
                from DBA_SYS_PRIVS 
                where PRIVILEGE='DELETE ANY TABLE'
                or PRIVILEGE='UPDATE ANY TABLE'
                or PRIVILEGE='INSERT ANY TABLE')
     or USERNAME in (select distinct GRANTEE
                     from DBA_ROLE_PRIVS
                     where GRANTED_ROLE in (select distinct ROLE
                                            from ROLE_TAB_PRIVS
                                            where TABLE_NAME in (select TABLE_NAME
                                                                 from DBA_TABLES
                                                                 where TABLE_NAME=p_Tabla)
                                            and (PRIVILEGE='INSERT' or PRIVILEGE='UPDATE' or PRIVILEGE='DELETE')
                                            and OWNER=p_Propietario)
                     or GRANTED_ROLE in (select distinct ROLE
                                         from ROLE_SYS_PRIVS
                                         where PRIVILEGE='DELETE ANY TABLE'
                                         or PRIVILEGE='UPDATE ANY TABLE'
                                         or PRIVILEGE='INSERT ANY TABLE')
                     or GRANTED_ROLE in (select distinct GRANTEE
                                         from DBA_ROLE_PRIVS
                                         start with GRANTED_ROLE in (select distinct ROLE
                                                                     from DBA_ROLES 
                                                                     where ROLE in (select ROLE
                                                                                    from ROLE_TAB_PRIVS
                                                                                    where TABLE_NAME in (select TABLE_NAME
                                                                                                         from DBA_TABLES
                                                                                                         where TABLE_NAME=p_Tabla)
                                                                                    and (PRIVILEGE='INSERT' or PRIVILEGE='UPDATE' or PRIVILEGE='DELETE')
                                                                                    and OWNER=p_Propietario)
                                                                     or ROLE in (select ROLE
                                                                                 from ROLE_SYS_PRIVS
                                                                                 where PRIVILEGE='DELETE ANY TABLE'
                                                                                 or PRIVILEGE='UPDATE ANY TABLE'
                                                                                 or PRIVILEGE='INSERT ANY TABLE'))
                                         connect by GRANTED_ROLE = prior GRANTEE));
BEGIN
     dbms_output.put_line(chr(9));
     dbms_output.put_line('USERS PRIVILEGES MODIFY DATAS: ');
     for v_Usuarios in c_Usuarios loop
          dbms_output.put_line('-'||v_Usuarios.USERNAME);
     end loop;
END;
/


CREATE OR REPLACE PROCEDURE MostrarUserAlter(p_Tabla VARCHAR2,
                                             p_Propietario VARCHAR2)
IS
     cursor c_Usuarios is
     select USERNAME
     from DBA_USERS 
     where USERNAME in (select distinct GRANTEE
                        from DBA_TAB_PRIVS
                        where TABLE_NAME in (select TABLE_NAME
                                             from DBA_TABLES
                                             where TABLE_NAME=p_Tabla)
                        and PRIVILEGE='ALTER'
                        and OWNER=p_Propietario)
     or USERNAME in (select distinct GRANTEE 
                from DBA_SYS_PRIVS 
                where PRIVILEGE='ALTER ANY TABLE')
     or USERNAME in (select distinct GRANTEE
                     from DBA_ROLE_PRIVS
                     where GRANTED_ROLE in (select distinct ROLE
                                            from ROLE_TAB_PRIVS
                                            where TABLE_NAME in (select TABLE_NAME
                                                                 from DBA_TABLES
                                                                 where TABLE_NAME=p_Tabla)
                                            and PRIVILEGE='ALTER'
                                            and OWNER=p_Propietario)
                     or GRANTED_ROLE in (select distinct ROLE
                                         from ROLE_SYS_PRIVS
                                         where PRIVILEGE='ALTER ANY TABLE')
                     or GRANTED_ROLE in (select distinct GRANTEE
                                         from DBA_ROLE_PRIVS
                                         start with GRANTED_ROLE in (select distinct ROLE
                                                                     from DBA_ROLES 
                                                                     where ROLE in (select ROLE
                                                                                    from ROLE_TAB_PRIVS
                                                                                    where TABLE_NAME in (select TABLE_NAME
                                                                                                         from DBA_TABLES
                                                                                                         where TABLE_NAME=p_Tabla)
                                                                                    and PRIVILEGE='ALTER'
                                                                                    and OWNER=p_Propietario)
                                                                     or ROLE in (select ROLE
                                                                                 from ROLE_SYS_PRIVS
                                                                                 where PRIVILEGE='ALTER ANY TABLE'))
                                         connect by GRANTED_ROLE = prior GRANTEE));
BEGIN
     dbms_output.put_line(chr(9));
     dbms_output.put_line('USERS PRIVILEGES MODIFY TABLES: ');
     for v_Usuarios in c_Usuarios loop
          dbms_output.put_line('-'||v_Usuarios.USERNAME);
     end loop;
END;
/

CREATE OR REPLACE PROCEDURE MostrarUserDrop(p_Tabla VARCHAR2,
                                            p_Propietario VARCHAR2)
IS
     cursor c_Usuarios is
     select USERNAME
     from DBA_USERS 
     where USERNAME in (select distinct GRANTEE
                        from DBA_TAB_PRIVS
                        where TABLE_NAME in (select TABLE_NAME
                                             from DBA_TABLES
                                             where TABLE_NAME=p_Tabla)
                        and PRIVILEGE='DROP'
                        and OWNER=p_Propietario)
     or USERNAME in (select distinct GRANTEE 
                from DBA_SYS_PRIVS 
                where PRIVILEGE='DROP ANY TABLE')
     or USERNAME in (select distinct GRANTEE
                     from DBA_ROLE_PRIVS
                     where GRANTED_ROLE in (select distinct ROLE
                                            from ROLE_TAB_PRIVS
                                            where TABLE_NAME in (select TABLE_NAME
                                                                 from DBA_TABLES
                                                                 where TABLE_NAME=p_Tabla)
                                            and PRIVILEGE='DROP'
                                            and OWNER=p_Propietario)
                     or GRANTED_ROLE in (select distinct ROLE
                                         from ROLE_SYS_PRIVS
                                         where PRIVILEGE='DROP ANY TABLE')
                     or GRANTED_ROLE in (select distinct GRANTEE
                                         from DBA_ROLE_PRIVS
                                         start with GRANTED_ROLE in (select distinct ROLE
                                                                     from DBA_ROLES 
                                                                     where ROLE in (select ROLE
                                                                                    from ROLE_TAB_PRIVS
                                                                                    where TABLE_NAME in (select TABLE_NAME
                                                                                                         from DBA_TABLES
                                                                                                         where TABLE_NAME=p_Tabla)
                                                                                    and PRIVILEGE='DROP'
                                                                                    and OWNER=p_Propietario)
                                                                     or ROLE in (select ROLE
                                                                                 from ROLE_SYS_PRIVS
                                                                                 where PRIVILEGE='DROP ANY TABLE'))
                                         connect by GRANTED_ROLE = prior GRANTEE));
BEGIN
     dbms_output.put_line(chr(9));
     dbms_output.put_line('USERS PRIVILEGES REMOVE TABLES: ');
     for v_Usuarios in c_Usuarios loop
          dbms_output.put_line('-'||v_Usuarios.USERNAME);
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

##### Prueba

```sql
SQL> exec MostrarInfoTabla('LOCALES');
	
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	
OWNER: PACO
	
USERS PRIVILEGES READ:
-SYS
-SYSTEM
-USUARIOPRUEBA
-GGSYS
-GSMADMIN_INTERNAL
-MDSYS
-OLAPSYS
-LUIS
-USUARIO1
-WMSYS
-NUEVOUSUARIO
	
USERS PRIVILEGES MODIFY DATAS:
-SYS
-SYSTEM
-USUARIOPRUEBA
-GSMADMIN_INTERNAL
-MDSYS
-OLAPSYS
-LUIS
-USUARIO1
-WMSYS
-NUEVOUSUARIO
	
USERS PRIVILEGES MODIFY TABLES:
-SYS
-SYSTEM
-GSMUSER
-GGSYS
-GSMADMIN_INTERNAL
-MDSYS
-USUARIO1
-WMSYS
-NUEVOUSUARIO
	
USERS PRIVILEGES DROP TABLES:
-SYS
-SYSTEM
-GSMUSER
-GSMADMIN_INTERNAL
-OLAPSYS
-USUARIO1
-WMSYS
-NUEVOUSUARIO
	
Nº Extensión  Inicio   Siguiente  Mínimo  Máximo        Fichero
------------- -------- ---------- ------- ------------- -------------------------------
1	      65536    1048576	  1	  2147483645	/opt/oracle/oradata/orcl/users01.dbf
	
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Procedimiento PL/SQL terminado correctamente.
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

Como podemos ver, el proceso de control de espacio de almacenamiento se crea con una vista y utilizando la función `pg_total_relation_size`.

##### Inconvenientes

* Como no pasa en Oracle, las transacciones se abortan si se encuentran con algun fallo durante la ejecución.
* Es muy simple el soporte que aporta PostgreSQL.

## MySQL:

### Tarea 8
---------------------------------------------------------------

#### Averigua si existe el concepto de índice en MySQL y si coincide con el existente en ORACLE. Explica los distintos tipos de índices existentes.

En MySQL existe el concepto de índice y es el mismo concepto que en oracle, el cual agilizan las busquedas en una tabla, para evitar recorrer toda la tabla para obtener los datos solicitados.

Existen cinco tipos de índices en MySQL:

* PRIMARY KEY: Índice sobre un o más campos donde cada valor es único y nunguno de los valores son NULL. 
* KEY o INDEX:
* UNIQUE: Índice que no es `PRIMARY KEY` pero que no permite que los sean iguales.
* FULLTEXT: Índices que se usan en tablas `MyISAM`, y pueden contener uno o más campos del tipo `CHAR`, `VARCHAR` y `TEXT`. Este índice se utiliza para optimizar la busqueda de palabras clave en las tablas que tienen grandes cantidades de infomación en campos de texto.
* SPATIAL: Índice que se utiliza sobre columnas de datos geométricos.


## MongoDB:

### Tarea 9
---------------------------------------------------------------

#### Explica los distintos motores de almacenamiento que ofrece MongoDB, sus características principales y en qué casos es más recomendable utilizar cada uno de ellos.