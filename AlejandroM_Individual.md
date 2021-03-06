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

Para listar los segmentos existentes para realizar un rollback es necesario llamar a la vista `DBA_ROLLBACK_SEGS` y para mostrar la información de las extensiones a la vista `DBA_EXTENTS`.

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

Para listar a todos los usuarios que tengan acceso de escritura y lectura de un tablespace tenemos que tener en cuenta varios tipos de privilegios:

* Mostrar los **propietarios de las tablas** del tablespace.
* Mostrar los usuarios y que tienen **privilegios sobre las tablas** del tablespace
* Mostrar los usuarios que tienen **privilegios del sistema** de todas las tablas como:
  
|Privilegios del sistema|
|:---------------------:|
|ALTER ANY TABLE        |
|INSERT ANY TABLE       |
|UPDATE ANY TABLE       |
|DELETE ANY TABLE       |
|DROP ANY TABLE         |
|READ ANY TABLE         |
|SELECT ANY TABLE       |

* Mostrar a los **propietarios de los indexes** del tablespace.
* Mostrar a los **propietarios de los clusters** del tablespace.
* Mostrar los usuarios que tienen un rol, o de manera recursiva tienen un rol, que le otorguen los privilegios del sistema o de las tablas del tablespace.

Sabiendo todo lo que tenemos que mostrar, podemos pasar a realizar el procedimiento:

###### Procedimento que muestra a los usuarios que tienen permiso de lecturas y escritura sobre los objetos de un tablespace
```sql
set pagesize 999
set linesize 999

CREATE OR REPLACE PROCEDURE MostrarUsuariosAccesoTS(p_Tablespace VARCHAR2)
IS
     cursor c_Usuarios is
     select USERNAME 
     from DBA_USERS  -- Utilizamos la vista DBA_USERS para que solo muestre usuarios y no roles.
-- Indicamos que muestre los propietarios de las tablas.
     where USERNAME in (select distinct OWNER
                        from DBA_TABLES 
                        where TABLESPACE_NAME=p_Tablespace)
-- Indicamos que muestre los usuarios con privilegios sobre las tablas.
     or USERNAME in (select distinct GRANTEE
                     from DBA_TAB_PRIVS
                     where TABLE_NAME in (select TABLE_NAME
                                          from DBA_TABLES
                                          where TABLESPACE_NAME=p_Tablespace))
-- Indicamos que muestre los usuarios con privilegios de sistema.
     or USERNAME in (select distinct GRANTEE 
                     from DBA_SYS_PRIVS 
                     where PRIVILEGE='ALTER ANY TABLE'
                     or PRIVILEGE='READ ANY TABLE'
                     or PRIVILEGE='SELECT ANY TABLE'
                     or PRIVILEGE='DROP ANY TABLE'
                     or PRIVILEGE='DELETE ANY TABLE'
                     or PRIVILEGE='UPDATE ANY TABLE'
                     or PRIVILEGE='INSERT ANY TABLE')
-- Indicamos que muestre a los usuarios que tenga un rol con privilegios sobre las tablas o sistema.
     or USERNAME in (select distinct GRANTEE
                     from DBA_ROLE_PRIVS
                     where GRANTED_ROLE in (select distinct ROLE
                                            from ROLE_TAB_PRIVS -- Privilegios sobre de tablas
                                            where TABLE_NAME in (select TABLE_NAME
                                                                 from DBA_TABLES
                                                                 where TABLESPACE_NAME=p_Tablespace))
                     or GRANTED_ROLE in (select distinct ROLE
                                         from ROLE_SYS_PRIVS -- Privilegios del sistema
                                         where PRIVILEGE='ALTER ANY TABLE'
                                         or PRIVILEGE='READ ANY TABLE'
                                         or PRIVILEGE='SELECT ANY TABLE'
                                         or PRIVILEGE='DROP ANY TABLE'
                                         or PRIVILEGE='DELETE ANY TABLE'
                                         or PRIVILEGE='UPDATE ANY TABLE'
                                         or PRIVILEGE='INSERT ANY TABLE')
-- Además indicamos que nos muestre los roles que de manera recursiva tienen roles con privilegios sobre tablas o del sistema.
                     or GRANTED_ROLE in (select distinct GRANTEE
                                         from DBA_ROLE_PRIVS
                                         start with GRANTED_ROLE in (select distinct ROLE
                                                                     from DBA_ROLES 
                                                                     where ROLE in (select ROLE
                                                                                    from ROLE_TAB_PRIVS -- Privilegios sobre tablas.
                                                                                    where TABLE_NAME in (select TABLE_NAME
                                                                                                         from DBA_TABLES
                                                                                                         where TABLESPACE_NAME=p_Tablespace))
                                                                     or ROLE in (select ROLE
                                                                                 from ROLE_SYS_PRIVS -- Privilegios del sistema
                                                                                 where PRIVILEGE='ALTER ANY TABLE'
                                                                                 or PRIVILEGE='READ ANY TABLE'
                                                                                 or PRIVILEGE='SELECT ANY TABLE'
                                                                                 or PRIVILEGE='DROP ANY TABLE'
                                                                                 or PRIVILEGE='DELETE ANY TABLE'
                                                                                 or PRIVILEGE='UPDATE ANY TABLE'
                                                                                 or PRIVILEGE='INSERT ANY TABLE'))
                                         connect by GRANTED_ROLE = prior GRANTEE))
-- Indicamos que muestre los propietarios de los indexes del tablespace. 
     or USERNAME in (select OWNER 
                     from DBA_INDEXES 
                     where TABLESPACE_NAME=p_Tablespace)
-- Indicamos que muestre los propietarios de los clusters del tablespace.
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

######  Nos devuelve lo siguiente
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

Para realizar este procedimiento hay que tener en cuenta varias cosas:

* Puede haber una tabla con mismo nombre, por consiguiente, tenemos que sacar los propietarios que tiene la tabla y luego mostrar la información de cada tabla con sus respectivos propietarios.
* Tenemos que sacar para cada apartado de privilegios:
  * Los privilegios de sistema.
  * Los privilegios sobre tablas.
  * Tener en cuenta los roles que tienen los privilegios como en la Tarea 5.

Sabiendo esto podemos pasar al procedimiento.

###### Procedimiento que muestra el propietario, los usuarios con privilegios, extensiones y ruta del fichero de una determinada tabla
```sql
set pagesize 999
set linesize 999

-- Procedimiento principal, donde agrupamos todos los procedimientos de MOSTRAR.
CREATE OR REPLACE PROCEDURE MostrarInfoTabla(p_Tabla VARCHAR2)
IS
-- Tenemos que enviar el OWNER a todos los procedimientos, como indicamos en los puntos anteriores.
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

-- Procedimiento para mostrar los propietarios.
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

-- Procedimiento para mostrar los usuarios con privilegios de lectura.
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

-- Procedimiento para mostrar los usuarios con privilegios de modificación de registros.
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

-- Procedimiento para mostrar los usuarios con privilegios de modificar la estructura de las tablas.
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

-- Procedimiento para mostrar los usuarios con privilegios de borrar las tablas.
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

-- Procedimiento para mostrar la información de las extensiones.
CREATE OR REPLACE PROCEDURE MostrarExt(p_Tabla VARCHAR2,
                                       p_Propietario VARCHAR2)
IS
-- Realizamos un join con la vista DBA_EXTENTS para obtener el FILE_ID y mandarlo a la función MostrarFicheroExt.
     v_File VARCHAR2(50);
     cursor c_Extens is
     select seg.EXTENTS, seg.INITIAL_EXTENT, seg.NEXT_EXTENT, seg.MIN_EXTENTS, seg.MAX_EXTENTS, ex.FILE_ID
     from DBA_SEGMENTS seg, DBA_EXTENTS ex
     where seg.SEGMENT_NAME = ex.SEGMENT_NAME
     and seg.SEGMENT_NAME=p_Tabla
     and seg.OWNER=p_Propietario;
BEGIN
     dbms_output.put_line(chr(9));
     dbms_output.put_line(RPAD('Nº Extensión',13)||' '||
                          RPAD('Inicio',8)||' '||
                          RPAD('Siguiente',10)||' '||
                          RPAD('Mínimo',7)||' '||
                          RPAD('Máximo',13)||' '||
                          RPAD('Fichero',40));
     dbms_output.put_line(RPAD('-------------------------------',13)||' '||
                          RPAD('-------------------------------',8)||' '||
                          RPAD('-------------------------------',10)||' '||
                          RPAD('-------------------------------',7)||' '||
                          RPAD('-------------------------------',13)||' '||
                          RPAD('-------------------------------',40));
     for v_Extension in c_Extens loop
          v_File:=MostrarFicheroExt(v_Extension.FILE_ID);
          dbms_output.put_line(RPAD(v_Extension.EXTENTS,13)||' '||
                               RPAD(v_Extension.INITIAL_EXTENT,8)||' '||
                               RPAD(v_Extension.NEXT_EXTENT,10)||' '||
                               RPAD(v_Extension.MIN_EXTENTS,7)||' '||
                               RPAD(v_Extension.MAX_EXTENTS,13)||' '||
                               RPAD(v_File,40));
     end loop;
END;
/

-- Función que devuelve la ruta y el nombre del datafile por el ID.
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

###### Nos devuelve lo siguiente

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
1             65536    1048576    1       2147483645    /opt/oracle/oradata/orcl/users01.dbf

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Procedimiento PL/SQL terminado correctamente.
```

## Postgres:

### Tarea 7        
---------------------------------------------------------------

#### Averigua si pueden establecerse claúsulas de almacenamiento para las tablas o los espacios de tablas en Postgres.

PostgreSQL es un gestor de base de datos donde no hay cláusulas de almacenamiento de datos como en Oracle pero hay una función que nos permite controlar el espacio de las tablas e índices. 

La función de la que estoy hablando es `pg_total_relation_size`, con la que se puede controlar el espacio usado por un tabla, incluyendo indices y tablas TOAST. La sintaxis utilizando la función es:

```sql
CREATE VIEW user_disk_usage AS SELECT r.rolname, SUM(pg_total_relation_size(c.oid)) AS total_disk_usage 
FROM pg_class c, pg_roles r 
WHERE c.relkind = 'r' 
AND c.relowner = r.oid 
GROUP BY c.relowner;
```

Como podemos ver, el proceso de control de espacio de almacenamiento se crea con una vista y utilizando la función `pg_total_relation_size`, configuramos una tabla concreto de la base de datos.

Esta vista nos permite ver el almacenamiento ocupado de una tabla pero no se le pueden aplicar clausulas de almacenamiento como asignarle cuotas, crear tablespace o limitar el almacenamiento de una tabla en la base de datos.

##### Inconvenientes

* Como no pasa en Oracle, las transacciones se abortan si se encuentran con algun fallo durante la ejecución.
* Es muy simple el soporte que aporta PostgreSQL.

## MySQL:

### Tarea 8
---------------------------------------------------------------

#### Averigua si existe el concepto de índice en MySQL y si coincide con el existente en ORACLE. Explica los distintos tipos de índices existentes.

En MySQL existe el concepto de índice y es el mismo concepto que en oracle, el cual agilizan las busquedas en una tabla, para evitar recorrer toda la tabla para obtener los datos solicitados.

Existen cuatro tipos de índices en MySQL:

|Índices  | Descripción
|:-------:|:---------------------------------------------------------------------------------------------------------------------------------
|INDEX    | Índice que no único, es decir, índice que admite valores duplicados sin restrinciones, solo para mejorar el tiempo de ejecucción de las consultas.
|UNIQUE   | Índice que restringe el uso valores duplicados, puede haber varios índices `UNIQUE` al contrario que el índice `PRIMARY`.
|PRIMARY  | Índice en el que todas las columnas deben de tener un valor único, igual que el `UNIQUE` pero solo puede haber un índice `PRIMARY` en una de las tablas.
|FULLTEXT | Índices que pueden contener uno o más campos del tipo `CHAR`, `VARCHAR` y `TEXT`. Este índice se utiliza para optimizar la busqueda de palabras clave en las tablas que tienen grandes cantidades de infomación en campos de texto.
|SPATIAL  | Índice que se emplean para realizar búsquedas sobre datos que componen formas geométricas.  

Todos estos índices pueden emplear una o más columnas.

Si lo comparamos con los índices de Oracle, podemos ver un poco de similitud por ejemplo en el caso de los índices `UNIQUE` y `PRIMARY`.

## MongoDB:

### Tarea 9
---------------------------------------------------------------

#### Explica los distintos motores de almacenamiento que ofrece MongoDB, sus características principales y en qué casos es más recomendable utilizar cada uno de ellos.

Los motores de almacenamiento son una de las funciones más interesantes de MongoDB, se añadieron en la versión 3.0. Los motores de almacenamiento ofrecen mejoras significativas en el rendimiento del servidor de MongoDb, como en la eficiencia de la compresión o concurrencia. Además de la escalabilidad al servir como interfaz entre el servidor de MongoDB y el disco físico.

Hay diferentes tipos de moteres de almacenamientos:

|Motor de almacenamiento| Descripción
|:-------------:|:----------------------
|Fusión-io| Es un motor de almacenamiento creado por SanFisk, que omite la capa del sistema de archivos del sistema operativo y escribe directamente en el dispositivo de almacenamiento. Esto mejora significativamente el rendimiento y la eficiencia de los recursos.
|En memoria| Es un tipo de motor de almacenamiento donde todos los datos se almacenan en la memoria volátil (RAM) para realizar lectura y acceso más rápidos. Para solucionar el problema del borrado al reinicio del servidor, podemos configurar réplicas standard para obtener failover automático ( que es el modo de funcionamiento de respaldo en el que las funciones de un componente del sistema, el caso de la RAM, son asumidos por componentes del sistema secundario cuando el componente principal no esta disponible).
|MMAP| Es un motor de almacenamiento que optimiza la lectura de los ficheros grandes al guardalos en una memoria virtual y si tiene que leer una parte pequeña del fichero, este sistema realiza la lectura de manera eficiente. La desventaja es que no se puede procesar dos llamadas de escritura en paralelo para la misma colección.
|Rocas Mongo| Un motor de valor-clave creado para integrarse con RockDB de Facebook.
|TokuMX| Un motor de almacenamiento creado por Pecona que utiliza índices de árboles fractales. Los árboles fractales (fractal tree), es una estructura de datos que mantiene los datos ordenados y permite búsquedas al mismo tiempo que un treeB que suele utlizar Oracle pero con insercciones y eliminaciones que son más rápidas.
|WiredTiger| Un motor de almacenamiento, actualmente el predeterminado de MongoDB, que admite árboles LSM para almacenar índices, estos árboles son mejores en operaciones de escitura donde las grandes cargas de trabajo son de inserciones aleatorias. En WiredTiger no se actualizaran los datos, sino que se crearán nuevo datos y se eliminarán lo anterior. Además dos o más operaciones de escritura no afectarán al mismo documento.