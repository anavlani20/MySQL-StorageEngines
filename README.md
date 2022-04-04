**UF2. Instal·lació i ajustament de SGBD corporatiu**

# Activitat 3 Storage Engines MySQL
**Aryan Navlani**  
*Aquesta pràctica s'ha fet des d'un Centos 7 amb MySQL 5.7*  

![image](https://user-images.githubusercontent.com/61285257/160308243-d0ef56c5-cc02-4da9-92d4-f11fa7dccfa9.png)

<a name="index"></a>
## ÍNDEX
* [Activitat 1](#Ac1)
* [Activitat 2](#Ac2)
* [Activitat 3](#Ac3)
* [Activitat 4](#Ac4)
* [Activitat 5](#Ac5)
* [Activitat 6](#Ac6)
* [Activitat 7](#Ac7)
* [Activitat 8](#Ac8)

<a name="Ac1"></a>
## Activitat 1. REALITZA I/O RESPON ELS SEGÜENTS APARTATS
* [Tornar al ÍNDEX](#index)

### Indica quins són els motors d’emmagatzematge que pots utilitzar (quins estan actius)? Mostra al comanda utilitzada i el resultat d’aquesta.
       
![image](https://user-images.githubusercontent.com/61285257/159655511-e49d9faf-1d81-4ad0-8088-373fb37ca9f5.png)
Bàsicament podem observar els motors disponibles, i informació extra, com per exemple quin és el motor per defecte, savepoints, etc.

````mysql
SHOW STORAGE ENGINGES;
SELECT * FROM INFORMATION_SCHEMA.ENGINES;
````
He utilitzat la primera, però també podem utilitzar la segona consulta.
Donen exactament els mateixos resultats, ja que les comandes són diferents però busquen al mateix lloc.
     

### Com puc saber quin és el motor d’emmagatzematge per defecte. Mostra com canviar aquest paràmetre de tal manera que les noves taules que creem a la BD per defecte utilitzin el motor MyISAM?

En l'anterior punt, quan llistem tots els storages, ja es veu quin és el default, perquè òbviament només n'hi ha un.
Però per només buscar un resultat específic de la taula, en aquest cas Suport, podem executar la següent query.

![image](https://user-images.githubusercontent.com/61285257/159659397-e72f0fe4-f927-42e3-a74e-7dee7cdcb62b.png)


````mysql
SELECT * FROM INFORMATION_SCHEMA.ENGINES 
WHERE SUPPORT="DEFAULT";
````

Per canviar el motor d'emmagatzematge per defecte, ho farem afegint la línia corresponent en l'arxiu de configuració my.cnf, previament de reiniciar el servei. Ho podríem fer també canviant la variable a partir de la consulta; 
````mysql
SET default_storage_engine=MyISAM;
````
Però només seria per la sessió actual, per tant, millor que es guardi permanentment.
  
Per tant, anem al my.cnf i afegirem la següent línia corresponent.

````mysql
[mysqld]
default-storage-engine = MyISAM
````  
![image](https://user-images.githubusercontent.com/61285257/159663426-2528962c-ec76-470f-9ac0-a0514be76794.png)


Per aplicar els canvis, reiniciarem el servei mysql.

````bash  
 systemctl restart mysqld
````
![image](https://user-images.githubusercontent.com/61285257/159663980-c73eff63-dab4-49d0-8ea9-e7baee0463b1.png)

Ara dins de mysql, si tornem a buscar els storage engines, podrem veure que ara per defecte és el MyISAM.

![image](https://user-images.githubusercontent.com/61285257/159664399-b7b6a720-47b1-4fd3-9f39-235cae4b5312.png)


### Explica els passos per instal·lar i activar l'ENGINE MyRocks. MyRocks és un motor d'emmagatzematge per MySQL basat en RocksDB (SGBD incrustat de tipus clau-valor). Aquest tipus d’emmagatzematge està optimitzat per ser molt eficient en les escriptures amb lectures acceptables.
    
Primer, instal·larem el My Rocks a la nostra màquina(percona), amb el gestor de paquets corresponent.
Si tenim la versió del percona més recent, només indicarem percona-server-rocksdb, però si tenim una versió anterior, l'hem d'especificar, per exemple si tinc la 5.7,percona-server-rocksdb-57.

````bash
 sudo yum install Percona-Server-rocksdb-57
````
![image](https://user-images.githubusercontent.com/61285257/159667143-5c38e7c8-3357-4500-afad-80ea7fca9b14.png)

     
Per activar-lo, ho farem via execució del paràmetre ps-admin.


````bash
 ps-admin --enable-rocksdb -u <mysql_admin_user> -p[mysql_admin_pass]
````
En el meu cas, ho fare amb l'usuari root.

![image](https://user-images.githubusercontent.com/61285257/159668998-66c2b564-73a1-4119-a6fc-d24b29ea3628.png)

Podem veure que els missatges ens diuen la funcionalitat de la comanda.

Ara si consultem un altre vegada els storage engines,dins del mysql,ens surt l'engine MyRocks,el primer de la taula.

![image](https://user-images.githubusercontent.com/61285257/159669476-5fcb7f7a-1f58-4819-95d6-7445a07b645c.png)

**Checkpoint: Mostra al professor que està instal·lat i posa un exemple de com funciona.**
    
### Importa la BD Sakila com a taules MyISAM. Fes els canvis necessaris per importar la BD Sakila perquè totes les taules siguin de tipus MyISAM.
    
Primer de tot, quan tinguem el fitxer sakila-schema.sql, per importar aquesta base de dades com a taules MyISAM, el que farem és canviar totes les referències InnoDB(per defecte)dins del fitxer a MyISAM. Això ho podem fer de diverses maneres, per exemple una vegada dins de la màquina centenars podem utilitzar la comanda; 
sed -i 's/ENGINE=InnoDB/ENGINE=MyISAM/' sakila-schema.sql 
Perquè faci aquests canvis dins del fitxer. Però en el meu cas editaré el fitxer en un gestor de textos a la màquina real(Windows), per després passar aquest arxiu ja canviat al sistema Redhat Centos.

Podem veure en aquesta imatge com amb l'eina cercar i reemplaçar canvis els engines de la taula.

![image](https://user-images.githubusercontent.com/61285257/159679219-455d3697-e0e1-4011-aeff-149b88439413.png)

Quan passem el fitxer al servidor percona, comprovarem el contingut, amb un cat.

![image](https://user-images.githubusercontent.com/61285257/159683933-849d745c-9dc8-461d-a305-52d91df4611a.png)

Ara dins de mysql, hauríem d'importar aquesta base de dades, com podem veure anteriorment, la vaig desar dins de /var/lib/mysql. 
L'importació la farem amb un source.

````mysql
SOURCE /var/lib/mysql/sakila-schema.sql
````
![image](https://user-images.githubusercontent.com/61285257/159684169-732e2ef8-5a87-4c84-96da-97390aca6f67.png)

Podem comprovar que la base de dades ha estat importada correctament i també podem veure que les taules han estat importades amb el motor MyISAM.

![image](https://user-images.githubusercontent.com/61285257/159684511-edd9d154-d339-4960-a232-2e264695ff71.png)

         
### Mira quins són els fitxers físics que ha creat, quan ocupen i quines són les seves extensions. Mostra'n una captura de pantalla i indica què conté cada fitxer.**
        
Aquests fitxers es troben al directori /var/lib/mysql/sakila.
        
El directori conté els següents fitxers:
*He de comentar que utilitzo la comanda ls -l -S -h, per filtrar,hi han moltes maneres*

![image](https://user-images.githubusercontent.com/61285257/159690385-da443af5-4d8e-491a-9698-f8b1d616a901.png)

**Mida dels fitxers**  
Com podem veure, tot el directori té una mida de 304K.  
Els fitxers **.frm** tenen una mida mínima de 1,7K fins al màxim que és 9,0K.  
Els fitxers **.MYI** tenen una mida aproximada de 1,0K.  
Els fitxers **.TRN** tenen una mida de mitjana d'uns 50 bytes aproximadament.  
Els fitxers **.MYD** tenen una mida inicial de 0.  

**Informació dels fitxers**  
- Format file (.frm): Conté la definició de l'estructura de la taula. (Definir camps)
- Data File (.MYD): Conté el contingut de la taula, bàsicament les dades de la taula, el que són files i dades.
- Index file (.MYI): Guarda els índexs de les taules MyISAM.
- Backup file (.TRN): Els fitxers TRN s'utilitzen per restaurar la base de dades i es poden utilitzar amb altres fitxers TRN per retrocedir en cadena a qualsevol estat anterior de la base de dades.

#### Pràcticament no es pot veure el contingut de cap fitxer perquè estan encryptats.
    

### Un cop fet això torna a deixar el motor InnoDB per defecte.
Comentem o esborrem el paràmetre definit anteriorment a la configuració.

![image](https://user-images.githubusercontent.com/61285257/159692952-d03e8662-2cef-4a5b-b072-33396d4b2832.png)

Reiniciem el servei.

````bash  
 systemctl restart mysqld
````
Si consultem els motors podem veure que InnoDB és el motor per defecte.

![image](https://user-images.githubusercontent.com/61285257/159693027-563496d7-77fa-40eb-93fb-c23e4af863f2.png)

### A partir de MySQL apareixen els schemas de metadades i informació guardats amb InnoDB. Busca informació d'aquests schemas. Indica quin és l'objectiu de cadascun d'ells i posa'n un exemple d'ús.

Podeu extreure metadades sobre objectes d'esquema gestionats per InnoDB mitjançant les taules del sistema INNODB INFORMATION_SCHEMA. Aquesta informació prové de les taules internes del sistema InnoDB (també anomenades diccionari de dades InnoDB), que no es poden consultar directament com les taules InnoDB normals.

````mysql
SHOW TABLES FROM INFORMATION_SCHEMA LIKE 'INNODB_SYS%';
````  
![image](https://user-images.githubusercontent.com/61285257/159727035-9751fa61-38da-44bf-90e4-1fba9dbb49a5.png)

Els noms de les taules són indicatius del tipus de dades proporcionades:  

- **INNODB_SYS_TABLES** proporciona metadades sobre les taules InnoDB, equivalent a la informació de la taula SYS_TABLES del diccionari de dades InnoDB.
- **INNODB_SYS_COLUMNS** proporciona metadades sobre les columnes de la taula InnoDB, equivalent a la informació de la taula SYS_COLUMNS del diccionari de dades InnoDB.
- **INNODB_SYS_INDEXES** proporciona metadades sobre índexs InnoDB, equivalent a la informació de la taula SYS_INDEXES del diccionari de dades InnoDB.
- **INNODB_SYS_FIELDS** proporciona metadades sobre les columnes claus primaries (camps) dels índexs InnoDB, equivalent a la informació de la taula SYS_FIELDS del - diccionari de dades InnoDB.
- **INNODB_SYS_TABLESTATS** proporciona una vista de la informació d'estat de baix nivell sobre les taules InnoDB que es deriva d'estructures de dades a la memòria. No hi ha cap taula interna del sistema InnoDB corresponent.
- **INNODB_SYS_DATAFILES** proporciona informació de la ruta del fitxer de dades per als espais de taula de fitxers per taula d'InnoDB, equivalent a la informació de la taula SYS_DATAFILES del diccionari de dades d'InnoDB.
- **INNODB_SYS_TABLESPACES** proporciona metadades sobre els espais de taula de fitxers per taula d'InnoDB, equivalent a la informació de la taula SYS_TABLESPACES del diccionari de dades d'InnoDB.
- **INNODB_SYS_FOREIGN** proporciona metadades sobre claus foraneas definides a les taules InnoDB, equivalent a la informació de la taula SYS_FOREIGN del diccionari de dades InnoDB.
- **INNODB_SYS_FOREIGN_COLS** proporciona metadades sobre les columnes de claus foraneas que es defineixen a les taules InnoDB, equivalent a la informació de la taula SYS_FOREIGN_COLS del diccionari de dades InnoDB.
- **INNODB_SYS_VIRTUAL** proporciona metadades sobre la columna virtual generada per InnoDB i la columna en la que es basa la columna virtual generada, equivalent a la informació de la taula SYS_VIRTUAL en el diccionari de dades InnoDB.

#### EXEMPLE D'Ús
El que farem ara és, primer, crear una base de daus simple amb una taula i les seves columnes, un index en la primera columna i amb el motor InnoDB.
Després per veure com funcionen aquestes metadades, farem un exemple de cada, cercant cada metadada.

`````mysql
CREATE DATABASE pr1;
USE pr1;
CREATE TABLE t1 (
col1 INT,
col2 CHAR(10),
col3 VARCHAR(10))
ENGINE = InnoDB;
CREATE INDEX index1 ON tabla(columna1);
`````
![image](https://user-images.githubusercontent.com/61285257/159731610-cd7cc2f4-b092-4e6b-8406-8ee31a395f9f.png)

Ara començarem a cercar cada metadada.  
Consultem les metadades de les taules.  
Després de crear la taula t1, consulteu INNODB_SYS_TABLES per localitzar les metadades per a la base de dades pr1/t1:  

`````mysql
SELECT * FROM INFORMATION_SCHEMA.INNODB_SYS_TABLES WHERE NAME='pr1/t1' \G
`````
![image](https://user-images.githubusercontent.com/61285257/159732076-a64a822a-d21d-4ae9-951a-13562b74cbd6.png)


Consultem les metadades de les columnes.  
L'ID de la Taula indicada es la que vam rebre en l'anterior consulta.  

```mysql
SELECT * FROM INFORMATION_SCHEMA.INNODB_SYS_COLUMNS where TABLE_ID = 43 \G
```

![image](https://user-images.githubusercontent.com/61285257/159733076-d73789c8-ac04-4faf-bb33-5d90cad001b5.png)


Consultem les metadades de l'índex.

```mysql
SELECT * FROM INFORMATION_SCHEMA.INNODB_SYS_INDEXES WHERE TABLE_ID = 43 \G
```

![image](https://user-images.githubusercontent.com/61285257/159733367-a1279c8c-0dc3-407e-9a47-60c2adc8d400.png)

Consultem les metadades de les columnes PRIMARY KEY.
INNODB_SYS_FIELDS proporcionarà metadades per a cadascun dels camps indexats.
Consultarem el segon index consultat anteriorment.

```mysql
SELECT * FROM INFORMATION_SCHEMA.INNODB_SYS_FIELDS where INDEX_ID = 46 \G
```

![image](https://user-images.githubusercontent.com/61285257/159734013-56d366f5-0d9f-47a1-8d66-4259644fda65.png)

Consultem les metadades dels espais de taula de fitxers per taula.
En la consulta indicarem l'espai, que el vam obtenir en la consulta d'índex.
```mysql
SELECT * FROM INFORMATION_SCHEMA.INNODB_SYS_TABLESPACES WHERE SPACE = 29 \G
```

![image](https://user-images.githubusercontent.com/61285257/159734779-9beedaf4-cb85-4e02-8862-3b5b4bb34917.png)

Consultem les metadades de la ruta del fitxer de dades per als espais de taula de fitxers.
En la consulta indicarem l'espai, que el vam obtenir en la consulta d'índex o l'anterior.

```mysql
SELECT * FROM INFORMATION_SCHEMA.INNODB_SYS_DATAFILES WHERE SPACE = 29 \G
```

![image](https://user-images.githubusercontent.com/61285257/159735221-2acb9ce6-a22e-4c19-a954-412f54c5c990.png)

Com a pas final, inserim una fila a la taula t1 (TABLE_ID = 43) i visualitzem les dades a la taula INNODB_SYS_TABLESTATS. L'optimitzador de MySQL utilitza les dades d'aquesta taula per calcular quin índex s'ha d'utilitzar quan es consulta una taula InnoDB.

```mysql
INSERT INTO t1 VALUES(5, 'abc', 'def');
SELECT * FROM INFORMATION_SCHEMA.INNODB_SYS_TABLESTATS where TABLE_ID = 43 \G
```

![image](https://user-images.githubusercontent.com/61285257/159736016-b2e7ea5a-804f-4b74-99f5-1f7283135c87.png)

La taula INNODB_SYS_VIRTUAL proporciona metadades sobre les columnes generades virtuals d'InnoDB i les columnes en què es basen les columnes generades virtuals,   equivalent a la informació de la taula SYS_VIRTUAL del diccionari de dades d'InnoDB.  

Apareix una fila a la taula INNODB_SYS_VIRTUAL per a cada columna en què es basa una columna generada virtual.  
Crearem una taula nova per poder crear una tercera columna com a virtual a partir de la 1ra i 2na columna.  
```mysql
CREATE TABLE `t2` (
a int(11) DEFAULT NULL,
b int(11) DEFAULT NULL,
c int(11) GENERATED ALWAYS AS (a+b) VIRTUAL,
h varchar(10) DEFAULT NULL)
ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

SELECT * FROM INFORMATION_SCHEMA.INNODB_SYS_VIRTUAL
WHERE TABLE_ID IN
(SELECT TABLE_ID FROM INFORMATION_SCHEMA.INNODB_SYS_TABLES
WHERE NAME LIKE "pr1/t2");
```
![image](https://user-images.githubusercontent.com/61285257/159738449-ecc12bc5-1bd7-429b-99ee-baf6160dc9e5.png)

![image](https://user-images.githubusercontent.com/61285257/159738502-eda4e22a-8c76-4c80-8c4f-f1dcf10b450d.png)


Les taules INNODB_SYS_FOREIGN i INNODB_SYS_FOREIGN_COLS proporcionen dades sobre les relacions de clau foranea.   
Per quest exemple crearem una taula pare i una taula fill amb una relació de clau foranea per demostrar les dades que es troben a les taules INNODB_SYS_FOREIGN i   INNODB_SYS_FOREIGN_COLS.  
Bàsicament la segona taula tindra una clau foranea indexada sobre la primera.  


```mysql
CREATE TABLE parent (id INT NOT NULL,
PRIMARY KEY (id)) ENGINE=INNODB;

CREATE TABLE child (id INT, parent_id INT,
INDEX par_ind (parent_id),
CONSTRAINT fk1
FOREIGN KEY (parent_id) REFERENCES parent(id)
ON DELETE CASCADE) ENGINE=INNODB;
```

![image](https://user-images.githubusercontent.com/61285257/159739994-395a1915-00a8-4396-bd16-99b841fb36fe.png)

![image](https://user-images.githubusercontent.com/61285257/159740084-350b3ce0-c009-4a7f-ab1a-16874bfda2d3.png)

Consultem les metadades de FOREIGN KEYS.

```mysql
SELECT * FROM INFORMATION_SCHEMA.INNODB_SYS_FOREIGN \G
```

![image](https://user-images.githubusercontent.com/61285257/159740557-4a1cdbe6-a194-4665-9a0f-1c6735b8d315.png)

Consultem les metadades de FOREIGN KEYS(columnes).

```mysql
SELECT * FROM INFORMATION_SCHEMA.INNODB_SYS_FOREIGN_COLS WHERE ID = 'test/fk1' \G
```

![image](https://user-images.githubusercontent.com/61285257/159741079-86810249-8b5b-48d1-a758-2b34c6785c36.png)

### Posa un exemple que produeix un DEADLOCK i mostra-ho al professor.

Si podem utilitzar un cas de la vida real per a descriure-ho, seria així:  


![image](https://user-images.githubusercontent.com/61285257/159742609-f8aa20b2-7754-4824-9728-f0cfbcac4e2c.png)  
Un Deadlock succeeix quan dues o més transaccions intenten fer bloquejos de claus en ordre oposat, per exemple:  
consulta 1: bloquear clau(1), bloquear clau(2);  
consulta 2: bloquear clau(2), bloquear clau(1);  

Si ambas consultes s'executaran al mateix temps, la consulta 1 bloquejarà la clau(1) i la consulta 2 bloquejarà la clau(2) i cada consulta esperarà a que l'altra    alliberarà la clau, provocant el deadlock.  

Aleshores, si canviem les nostres consultes de manera que els bloquejos ocorren en el mateix ordre, és dir:  

consulta 1: bloquear clau(1), bloquear clau(2);  
consulta 2: bloquear clau(1), bloquear clau(2);  
S'evitaria el deadlock.  

Un deadlock, és una situació difícil de reproduir, i es donarà només en algunes circumstàncies molt concretes  

#### EXEMPLE
[Documentació MYSQL InnoDB Deadlock](https://dev.mysql.com/doc/refman/5.7/en/innodb-deadlock-example.html) 
L'exemple següent il·lustra com es pot produir un error quan una sol·licitud de bloqueig provoca un bloqueig. L'exemple inclou dos clients, A i B.  

Primer, el client A crea una taula que conté una fila i després comença una transacció. Dins de la transacció, A obté un bloqueig A a la fila seleccionant-lo en mode compartit:

![image](https://user-images.githubusercontent.com/61285257/159745070-243ad742-6b67-45bd-9927-b5ff0f83b52c.png)

A continuació, el client B(altre sessió) comença una transacció i intenta suprimir la fila de la taula:

![image](https://user-images.githubusercontent.com/61285257/159745176-86e284d5-54da-488f-bb79-501369da5697.png)

L'operació d'eliminació requereix un bloqueig B. 
El bloqueig no es pot atorgar perquè és incompatible amb el bloqueig que té el client A,  
de manera que la sol·licitud passa a la cua de sol·licituds de bloqueig per als blocs de fila i client B.

Finalment, el client A també intenta eliminar la fila de la taula:

![image](https://user-images.githubusercontent.com/61285257/159745313-399235a8-712e-4dfb-8e56-1a5fa6c3d4f4.png)

El bloqueig es produeix aquí perquè el client A necessita un bloqueig B per eliminar la fila. Tanmateix, aquesta sol·licitud de bloqueig no es pot concedir perquè el client B ja té una sol·licitud per a un bloqueig B i està esperant que el client A alliberi el seu bloqueig A. Tampoc es pot actualitzar el bloqueig A que té A a un bloqueig B a causa de la sol·licitud prèvia de B per a un bloqueig B. 



<a name="Ac2"></a>
## Activitat 2. INNODB part I. REALITZA ELS SEGÜENTS APARTATS  
* [Tornar al ÍNDEX](#index)

**Desactiva l’opció que ve per defecte de innodb_file_per_table**
    
En el fitxatge de configuració my.cnf, pondrem el paràmetre innodb_file_per_table en OFF, per desactivar-lo.

````bash
[mysqld]
innodb_file_per_table=OFF
````
![image](https://user-images.githubusercontent.com/61285257/159748607-9e24c1c3-9b8c-4d8b-a305-9ac4d8555305.png)

Reiniciem el servei.
````bash
 systemctl restart mysql
````

![image](https://user-images.githubusercontent.com/61285257/159663980-c73eff63-dab4-49d0-8ea9-e7baee0463b1.png)

Per comprovar que ho hem desactivat, podem revisar la variable amb un SHOW VARIABLES LIKE '%innodb_file_per_table%';

![image](https://user-images.githubusercontent.com/61285257/159749111-2c6b9fea-e8b6-4061-8d5b-696ba88c7b2a.png)

    
**Importa la BD Sakila com a taules InnoDB.**

Sakila amb les taules a InnoDB, és com ve per defecte a l'script proporcionat amb la pràctica,la podem tornar a importar i ja estaria, o també podem canviar els valors de nou amb(dins del directori /var/lib/mysql):
````mysql
sed -i 's/MyISAM/InnoDB/' sakila-schema.sql
````    
Abans d'importar la base de dades nova, estaria bé esborrar l'anterior amb un DROP DATABASE.

````mysql
SOURCE /var/lib/mysql/sakila-schema.sql
````    
![image](https://user-images.githubusercontent.com/61285257/159752732-f7bc030a-24b0-4494-b330-649b66e503ca.png)
![image](https://user-images.githubusercontent.com/61285257/159754941-144a6e1f-35ef-4390-bd4e-1c690a6dfed8.png)

   
**Quin/quins són els fitxers de dades? A on es troben i quin és la seva mida?**
   
Aquests fitxers es troben al directori /var/lib/mysql/sakila.
        
El directori conté els següents fitxers:
*He de comentar que utilitzo la comanda ls -l -S -h, per filtrar,hi han moltes maneres*

![image](https://user-images.githubusercontent.com/61285257/160296254-43f66cba-a5c9-4398-9b05-84fcb977c69e.png)

##### Podem veure els fitxers .frm i alguns TRM, pero tambè haurien d'estar els fitxers .ibd.  
##### Quan innodb_file_per_table=ON està activat, InnoDB utilitza un fitxer d'espai de taula per taula InnoDB. Aquests fitxers d'espai de taula tenen l'extensió .ibd. Quan innodb_file_per_table=OFF està establert, InnoDB emmagatzema totes les taules a l'espai de taula del sistema InnoDB, en aquest cas, en el fitxer .ibdata1, que l'analitzarem al final.

##### En altres paraules, Amb innodb_file_per_table activat, podem emmagatzemar taules InnoDB en un fitxer tbl_name.ibd. A diferència del motor d'emmagatzematge MyISAM, amb els seus fitxers separats tbl_name.MYD i tbl_name.MYI per a índexs i dades, InnoDB emmagatzema les dades i els índexs junts en un únic fitxer .ibd. 
##### El fitxer tbl_name.frm encara es crea com de costum.

##### Si desactivem innodb_file_per_table  i reiniciem el servidor, o el desactivem amb l'ordre SET GLOBAL, InnoDB crea taules noves dins de l'espai de taula del sistema tret que hàgim col·locat explícitament la taula a l'espai de taula fitxer per taula o a l'espai de taula general mitjançant l'opció CREATE. TAULA ... opció TABLESPACE.

[Font Oficial](https://mariadb.com/kb/en/innodb-file-per-table-tablespaces/#:~:text=When%20innodb_file_per_table%3DOFF%20is%20set,are%20created%20with%20CREATE%20TABLESPACE%20)
[Font Oficial 2](http://mysql.babo.ist/#/en/tablespace-enabling.html)
**Puc demostrar-ho si importo la base de dades amb innodb_file_per_table=ON(per defecte), podem tornar a veure els fitxers de dades**  

![image](https://user-images.githubusercontent.com/61285257/160296904-11f17083-3a3b-473c-b313-0ed48bdd1b0d.png)  
*Captura realitzada a partir d'importar la base de dades sense desactivar innodb_file_per_table*

##### Independentment de tot això, explicaré igualment tots els fitxers.  

**Informació dels fitxers**  
- Format file (.frm): Conté la definició de l'estructura de la taula. (Definir camps)
- Integrated Backup (.ibd): IBD és un fitxer d'espai de taula de base de dades MySQL. L'extensió IBD està associada amb el motor d'emmagatzematge de base de dades InnoDB, disponible sota llicència de codi obert GNU i utilitzat per defecte per MySQL versió 5.5 i posterior.
   Aquests fitxers emmagatzemen taules que constitueixen una base de dades.
- Backup file (.TRN): Els fitxers TRN s'utilitzen per restaurar la base de dades i es poden utilitzar amb altres fitxers TRN per retrocedir en cadena a qualsevol estat anterior de la base de dades.

**Mida dels fitxers**  
Com podem veure, tot el directori té una mida de 304K.  
Els fitxers .frm tenen una mida mínima de 1,2K fins al màxim que és 9,0K.  
Els fitxers .ibd tenen una mida mínima de 96K fins al màxim que és 160K. 
Els fitxers .TRN tenen una mida de mitjana d'uns 50 bytes aproximadament.  

### Pràcticament no es pot veure el contingut de cap fitxer perquè estan encryptats.

**InnoDB TableSpace**
Després tenim el .ibdata1, que està al directori per defecte on es guarden les dades MySQL, a /var/lib/mysql.  
Ho volia separar dels altres perquè al final, si utilitzem innodb com a motor de mysql per defecte emmagatzemarà totes les seves bases de dades a ibdata1.  

A més, conté el diccionari de les dades i l'historial de les transaccions de totes les taules.  

Té una mida de 76M.  

![image](https://user-images.githubusercontent.com/61285257/160297561-f133128b-7d47-4f96-ad26-9c67837a5634.png)

   
### Canvia la configuració del MySQL per:

- **Canviar la localització dels fitxers del tablespace de sistema per defecte a /discs-mysql/**
    
![image](https://user-images.githubusercontent.com/61285257/160298760-b398828b-6378-4808-8b73-981fefc92aaf.png)  
Primer de tot, amb una consulta al @@datadir, si no sabem el directori per defecte de dades de mysql, ho podrem saber.  
    
Suposarem que el nostre nou directori de dades és /discs-mysql, que en aquest cas ho es.
Després de crear el directori amb un simple mkdir,és important tenir en compte que aquest directori hauria de ser propietat de mysql:mysql i ens assegurarem que tinguin els permisos corresponents al d'un owner(propietari).  

````bash
 sudo mkdir /discs-mysql
 sudo chown -R mysql:mysql /discs-mysql
 sudo chmod 750 /discs-mysql/
````
![image](https://user-images.githubusercontent.com/61285257/160303857-a5e7a16d-1bf4-4b75-a40c-eff09d2bdc27.png)

Per garantir la integritat de les dades e la corrupció de la mateixa, aturarem el servei MySQL amb un stop(systemctl) abans de fer canvis al directori de dades.  
Si volem estar segur que ho hem fet, amb un status(systemctl), ho podrem comprovar.  

````bash
 sudo systemctl stop mysql
 sudo systemctl status mysql
````
![image](https://user-images.githubusercontent.com/61285257/160303894-d9fb0475-0226-4e1a-9114-9153e9aa6970.png)


Lògicament,ara que el servei mysql està aturat, copiarem recursivament el directori de la base de dades existent a la nova ubicació, és a dir,copiarem el contingut de /var/lib/mysql a /discs-mysql.  
     
````bash
 cp -R -p /var/lib/mysql/* /discs-mysql
````
![image](https://user-images.githubusercontent.com/61285257/160304177-a928cfc3-9459-49ab-90c5-b1c703a7d026.png)

*Establirem la seguretat de SELinux sobre aquest directori.*  
SELinux és un sistema administrat a través de certes regles anomenades «polítiques» que restringeixen o permeten l'ús de certes aplicacions per a parts essencials del sistema.

Instal·lem el paquet policycoreutils-python(eines de gestió que s'utilitzen per gestionar SELinux) que conté l'ordre semanage.

````bash
  yum -y install policycoreutils-python
````
![image](https://user-images.githubusercontent.com/61285257/160304216-036673b7-c66b-4016-aafb-1f7dc34ccffc.png)
![image](https://user-images.githubusercontent.com/61285257/160304227-cac945d1-5e60-46f6-ad59-1715673a4096.png)

Per afegir el tipus SELinux al mapa de context, és a dir, per establir el mapa de context SELinux al directori nou, utilitzem l'ordre semanage. Bàsicament amb aixó 
estem informant a SELinux quin és el context correcte afegint el tipus mysqld_db_t.

````bash
 semanage fcontext -a -t mysqld_db_t "/discs-mysql(/.*)?"      
````
![image](https://user-images.githubusercontent.com/61285257/160304278-bc008ad2-aad7-4b62-aeba-c2a0894b8e93.png)

Amb l'ordre restorecon, podem restaurar el context SELinux adequat al nou directori. 

````bash
 restorecon -R /discs-mysql      
````
![image](https://user-images.githubusercontent.com/61285257/160304308-12c241bd-5155-40d9-899a-3bc7eee5d5cf.png)


Editem el fitxer de configuració (my.cnf) per indicar el directori de dades nou (/discs-mysql en aquest cas) i el socket(Gestió les connexions al servidor,el fitxer socket s'anomena mysqld.sock i facilita les comunicacions entre diferent porcessos) tambè el redirigim al nou directori.
Localitzem les seccions [mysqld] i [client] i fem els canvis següents:

 ![image](https://user-images.githubusercontent.com/61285257/160304448-8728ad82-c9f9-4465-aa6c-bc51a9548f5b.png)

    
Iniciem el servei de mysql,també podem mirar l'estat d'aquest i comprovem si hem aplicat els canvis realitzats consultant @@datadir dins de mysql.

````bash
 systemctl start mysql
````
![image](https://user-images.githubusercontent.com/61285257/160304491-007027e7-4cdc-4650-b699-5202375262d5.png)


````mysql
SELECT @@datadir;
````
![image](https://user-images.githubusercontent.com/61285257/160304549-c2256f32-1887-4b28-a9fc-ebdd7572b7a0.png)

- **Tinguem dos fitxers corresponents al tablespace de sistema.**
- **Tots dos han de tenir la mateixa mida inicial (10MB)**
- **El tablespace ha de créixer de 5MB en 5MB.**
- **Situa aquests fitxers (de manera relativa a la localització per defecte) en una nova localització simulant el següent:**
- **/discs-mysql/disk1/primer fitxer de dades → simularà un disc dur**
- **/discs-mysql/disk2/segon fitxer de dades → simularà un segon disc dur.**
- **(0,5 punts) si les dues rutes anteriors són dos discs físics diferents.**


Aquesta configuració podem establir-la al fitxer de configuració /etc/my.cnf.  
Primer crearem els directoris corresponents amb els seus respectius permisos i owners.  
    
````bash
 sudo mkdir /discs-mysql/disk1
 sudo mkdir /discs-mysql/disk2
 sudo chown -R mysql:mysql /discs-mysql/disk1
 sudo chown -R mysql:mysql /discs-mysql/disk2
 sudo chmod 751 /discs-mysql/disk1
 sudo chmod 751 /discs-mysql/disk2
````
![image](https://user-images.githubusercontent.com/61285257/160306649-d0eb1fd4-b177-4c78-bed7-25ae02a11850.png)

Podem veure que aquests arxius (Aquests són els arxius on s'emmagatzemen les dades reals, els índexs i el registre d'innodb per a totes les bases de dades d'innodb),  tenen mides per defecte força superiors als quals volem definir, per tant si no els esborrem no podem aplicar després de la nostra configuració.
Per tant, hem de fer si o si, esborrar l'ibdata i els ib_logfiles,aturar el servei mysql i reiniciar el servei i carregar el dump amb la nostra configuració, ho veurem mes endevant..

````bash
 sudo rm ibdata1
 sudo rm ib_logfile0
 sudo rm ib_logfile1
````
![image](https://user-images.githubusercontent.com/61285257/160306912-4f7bd19c-0cd6-4657-9412-0b3c88e68035.png)  
Podem veure les mides d'aquests fitxers, que es van engrandint a mesura que el contingut de la base de dades creix.  

![image](https://user-images.githubusercontent.com/61285257/160306968-c76cc800-d818-417c-b490-bfb829b2e023.png)

Aturarem el servei mysql abans de configurar i aplicar aquesta configuració.

````bash
 sudo systemctl stop mysql
````
![image](https://user-images.githubusercontent.com/61285257/160307018-3d3636cc-9709-46dd-891f-2cc560b0fc53.png)

Modificarem el fitxer /etc/my.cnf de la següent manera.

Podem definir el nom de fitxer o els noms de fitxer de l'espai de taula del sistema, la mida i altres opcions configurant la variable del sistema **innodb_data_file_path**.
Quan configurem la variable del sistema **innodb_data_file_path**, podem definir una mida per a cada fitxer donat. 
Amb **autoextend**, quan l'últim espai de taula del sistema creix més enllà de l'assignació de mida, InnoDB augmenta la mida del fitxer per increments
Vull dir que quan s'especifica l'atribut **autoextend**, el fitxer de dades augmenta automàticament a mesura que es necessita espai. La variable **innodb_autoextend_increment** controla la mida de l'increment.
Els fitxers d'espai de taula del sistema es creen al directori de dades de manera predeterminada (datadir), que en aquest cas es **/discs-mysql**. 

![image](https://user-images.githubusercontent.com/61285257/160307596-6d8ef8d5-b7aa-46fd-bcc0-23c1e8cda8b6.png)  

Per especificar una ubicació alternativa, utilitzem l'opció **innodb_data_home_dir**.
També podem especificar **innodb_data_home_di** com a cadena buida, amb la variable buida es podria definir una ruta absoluta del fitxer de dades, en aquest cas, per exemple, **../disk1**. 


````bash
[mysqld]
innodb_data_home_dir=
innodb_data_file_path=disk1/ibdata1:10M;disk2/ibdata2:10M:autoextend
innodb_autoextend_increment=5M
````
![image](https://user-images.githubusercontent.com/61285257/160307074-8ff36db8-28a4-4fd4-a92c-a8daea1e1a8f.png)


Un cop configurat, finalment iniciarem el servei.

````bash
 sudo systemctl start mysql
````
Dins de mysql, comprovarem aquesta variable, que és la que vam definir anteriorment i si tot és correcte
ens hauria de sortir el que vam configurar.

![image](https://user-images.githubusercontent.com/61285257/160307332-8f6538d1-fc52-4828-bd4b-84311e91f7db.png)

Per comprovar que els dos tablespace s'estan utilitzant segons la mida que la configurem, ho podem fer
a partir d´aquesta comanda.  

![image](https://user-images.githubusercontent.com/61285257/160307801-daba120a-5ea3-4af2-90a5-b861059a997a.png)  

També podem comprovar el directori dels discos, com tenen i com estan creades els tablespaces corresponents.

![image](https://user-images.githubusercontent.com/61285257/160308357-d5ab3c00-ad1b-4d63-b631-101a4f5119aa.png)

**Checkpoint: Mostra al professor els canvis realitzats i que la BD continua funcionant.**

<a name="Ac3"></a>
## Activitat 3. INNODB part II. REALITZA ELS SEGÜENTS APARTATS 
* [Tornar al ÍNDEX](#index)


**Partint de l'esquema anterior configura el Percona Server perquè cada taula generi el seu propi tablespace en una carpeta anomenada tspaces (aquesta pot estar situada a on vulgueu).**
**Indica quins són els canvis de configuració que has realitzat.**

Els passos en aquesta activitat, pràcticament seran identics a l'activitat anterior, per començar primer de tot habilitarem el paràmetre innodb_file_per_table al /etc/my.cnf per poder generar el tablespace per cada taula. Perquè quan innodb_file_per_table=ON està activat, InnoDB utilitza un fitxer d'espai de taula per taula InnoDB.
Per defecte ja ve activat per tant només hem de comentar la desactivació,guardarem el fitxer i reiniciarem el servei.

![image](https://user-images.githubusercontent.com/61285257/160423072-6f977ec1-6472-4985-91c1-1888a7767893.png)

````bash
systemctl restart mysql
````

Per comprovar els canvis, consultarem la variable per shell:  

````mysql
SHOW VARIABLES LIKE '%innodb_file_per_table%';
````

![image](https://user-images.githubusercontent.com/61285257/160424833-a0d3d327-3f6c-41f0-a2a3-0c0a01ec71c5.png)  


En aquest cas el directori de dades que generara tablespace és /tspaces.
Després de crear el directori amb un simple mkdir,és important tenir en compte que aquest directori hauria de ser propietat de mysql:mysql i ens assegurarem que tinguin els permisos corresponents al d'un owner(propietari).  

````bash
 sudo mkdir /tspaces
 sudo chown -R mysql:mysql /tspaces
 sudo chmod 751 /tspaces
````
![image](https://user-images.githubusercontent.com/61285257/160425336-ede2250b-42e2-493f-9d9d-74008028e38f.png)


Per garantir la integritat de les dades e la corrupció de la mateixa, aturarem el servei MySQL amb un stop(systemctl) abans de fer canvis al directori de dades.  
Si volem estar segur que ho hem fet, amb un status(systemctl), ho podrem comprovar.  

````bash
sudo systemctl stop mysql
sudo systemctl status mysql
````

![image](https://user-images.githubusercontent.com/61285257/160427771-5d432522-d856-48d4-9275-2369f4201241.png)  
    

Lògicament,ara que el servei mysql està aturat, copiarem recursivament el directori de la base de dades existent a la nova ubicació, és a dir,copiarem el contingut de /var/lib/mysql a /discs-mysql.

*Podríem copiar també des de la /discs-mysql, ja que és el nostre actual datadir, però podríem tenir problemes amb els espais de taules que creem per a discos.*
   
````bash
  cp -R -p /var/lib/mysql/* /tspaces
 ````
 
 ![image](https://user-images.githubusercontent.com/61285257/160428199-0019f77e-97db-4f66-9490-79956095f10a.png)

 
Establirem la seguretat de SELinux sobre aquest directori.*  
SELinux és un sistema administrat a través de certes regles anomenades «polítiques» que restringeixen o permeten l'ús de certes aplicacions per a parts essencials del sistema.

Com que ja tenim instal·lat el paquet policycoreutils-python de l'anterior exercici, directament passarem amb el semanage i restorecon

Per afegir el tipus SELinux al mapa de context, és a dir, per establir el mapa de context SELinux al directori nou, utilitzem l'ordre semanage. Bàsicament amb aixó 
estem informant a SELinux quin és el context correcte afegint el tipus mysqld_db_t.

Amb l'ordre restorecon, podem restaurar el context SELinux adequat al nou directori. 

````bash
  semanage fcontext -a -t mysqld_db_t "/tspaces(/.*)?"
  restorecon -R /tspaces    
 ````
 ![image](https://user-images.githubusercontent.com/61285257/160428416-bf6afb67-6509-4e3e-b36e-561f9e80ad7a.png)


Repetirem el pas 4,punt 1, de l'activitat 2 i editem el fitxer de configuració (my.cnf) per indicar el directori de dades nou (/tspaces en aquest cas).

En aquest cas l'exercici no ens demana i tampoc no cal que el socket s'apliqui des del directori corresponent /tspaces, per tant ho deixem per defecte com a socket=/var/lib/mysql/mysql.sock

![image](https://user-images.githubusercontent.com/61285257/160437324-1a79cd7a-69da-41e7-9026-131bf94aeb46.png)


Iniciem el servei de mysql,també podem mirar l'estat d'aquest i comprovem si hem aplicat els canvis realitzats consultant @@datadir dins de mysql.

````bash
sudo systemctl start mysql
SELECT @@DATADIR;
````
![image](https://user-images.githubusercontent.com/61285257/160439399-4ae688af-6dd5-4407-ad29-48c58be4e58e.png)

![image](https://user-images.githubusercontent.com/61285257/160439612-83819421-2778-4fb6-9a7e-18e01416cf53.png)

<a name="Ac4"></a>
## Activitat 4. INNODB part III. REALITZA ELS SEGÜENTS APARTATS  
* [Tornar al ÍNDEX](#index)



**Crea un tablespace anomenat 'ts1' situat a `/discs-mysql/disk1/` i col·loca les taules actor, address i category de la BD Sakila.**

Primer de tot, abans de començar ens tornarem a assegurar que innodb_file_per_table estigui activat, perquè així, podem emmagatzemar els espais de taules InnoDB en un fitxer .ibd

Ho farem a partir de querys a mysql, crearem el tablespace i posteriorment d'utilitzar la base de dades sakila, passarem les taules corresponents al tablespace amb consultes DDL.  

````mysql
CREATE TABLESPACE ts1 ADD DATAFILE '/discs-mysql/disk1/ts1.ibd' ENGINE=InnoDB;
USE sakila;
ALTER TABLE actor TABLESPACE ts1;
ALTER TABLE address TABLESPACE ts1;
ALTER TABLE category TABLESPACE ts1;
````
![image](https://user-images.githubusercontent.com/61285257/160504456-c84b7656-fff1-4ea2-9d73-00334756d724.png)

Podem verificar la creació de l'espai de taula anterior mitjançant la consulta següent:

````mysql
SELECT FILE_ID,FILE_NAME,FILE_TYPE,TABLESPACE_NAME,ENGINE,STATUS FROM INFORMATION_SCHEMA.FILES WHERE TABLESPACE_NAME ='ts1'\G
````
![image](https://user-images.githubusercontent.com/61285257/160507899-c489d8a0-b0d6-44d4-89b1-36d8020a4755.png)

**Crea un altre tablespace anomenat 'ts2' situat a `/discs-mysql/disk2/` i col·loca-hi la resta de taules.**

Bàsicament, el procés és el mateix que l'anterior.
Primer ens informarem de totes les taules de la base de dades sakila amb un show tables, per després col·locar les que falten comparat amb l'exercici anterior.

````mysql
SHOW FULL TABLES FROM sakila WHERE table_type = 'BASE TABLE';
````
Els filtre per tipus de taula base perquè si no em sortirien totes les taules, fins a les virtuals(ocultes)  

![image](https://user-images.githubusercontent.com/61285257/160509216-77b540d6-f0f1-4330-b6a9-e55b0b70c334.png)

Com que són moltes consultes, se'm feia més amè executar aquestes consultes important-les des d'un fitxer i això he fet.  
He desat totes aquestes consultes a /var/lib/mysql/ts2.sql i les he importat amb un SOURCE.  


````sql
CREATE TABLESPACE ts2 ADD DATAFILE '/discs-mysql/disk2/ts2.ibd' ENGINE=InnoDB;
USE sakila;
ALTER TABLE city TABLESPACE ts2;
ALTER TABLE country TABLESPACE ts2;
ALTER TABLE customer TABLESPACE ts2;
ALTER TABLE film TABLESPACE ts2;
ALTER TABLE film_actor TABLESPACE ts2;
ALTER TABLE film_category TABLESPACE ts2;
ALTER TABLE film_text TABLESPACE ts2;
ALTER TABLE inventory TABLESPACE ts2;
ALTER TABLE language TABLESPACE ts2;
ALTER TABLE payment TABLESPACE ts2;
ALTER TABLE rental TABLESPACE ts2;
ALTER TABLE staff TABLESPACE ts2;
ALTER TABLE store TABLESPACE ts2;
````
![image](https://user-images.githubusercontent.com/61285257/160509676-8e9f9840-a62e-43ae-bc14-dcf1f0d14e4e.png)

![image](https://user-images.githubusercontent.com/61285257/160509799-d8623a6f-84eb-43e8-8cfe-039a75ff22f2.png)

Finalment podem verificar la creació de l'espai de taula anterior mitjançant la consulta següent:

````mysql
SELECT FILE_ID,FILE_NAME,FILE_TYPE,TABLESPACE_NAME,ENGINE,STATUS FROM INFORMATION_SCHEMA.FILES WHERE TABLESPACE_NAME ='ts2'\G
````

![image](https://user-images.githubusercontent.com/61285257/160509976-a1012d60-49b3-44d6-b65d-d8eeaf114a89.png)

Vull demostrar que s'han creat els datafiles .ibd, encara que amb la consulta anterior ja ho estic demostrant.

![image](https://user-images.githubusercontent.com/61285257/160511522-75114804-446c-4f6d-b7d4-ae5ea26f1011.png)

També podem consultar els tablespaces a les metadades.

![image](https://user-images.githubusercontent.com/61285257/160512212-f291bc5b-47de-48ad-916b-7ce66ea73c35.png)


**Comprova que pots realitzar operacions DML a les taules dels dos tablespaces.**

Per comprovar la realització de les operacions en aquests tablespaces, el que farem és inserir registres a aquestes taules.
Exactament afegirem registres a dues taules, bàsicament una taula corresponent a cada tablespace, en aquest cas, per exemple, les taules actor i language.

Els registres s'agreguen en columnes i per obtenir informacio primer d'aquestes columnes farem un SHOW COLUMNS;

````mysql
SHOW COLUMNS FROM actor;
SHOW COLUMNS FROM language;

````
![image](https://user-images.githubusercontent.com/61285257/160510733-89d8c00a-e132-4c74-ba53-91eaa7483a50.png)  

Inserim registres a les taules dels tablespaces ts1 i ts2 respectivament.

````mysql
INSERT INTO actor(actor_id,first_name,last_name) VALUE(1,'Sylvester','Stallone');
````

````mysql
INSERT INTO language(language_id,name) VALUES(6,'Spanish');
````

![image](https://user-images.githubusercontent.com/61285257/160511234-d45c3856-8870-48ed-9352-64a9070f8a80.png)

Podem consultar els registres de les taules introduïdes anteriorment amb un simple SELECT de tots els registres de la taula indicada (DML).

![image](https://user-images.githubusercontent.com/61285257/160512457-080ea294-aafc-4edf-bac3-174251632114.png)


**Quines comandes i configuracions has realitzat per fer els dos apartats anteriors?**

Les configuracions s'han mostrat anteriorment a aquesta pregunta, vull dir, que s'han mostrat mentre s'ha fet aquest apartat.

L'arxiu de configuració my.cnf l'he deixat com a l'exercici 2, amb el directori de dades a /discs-mysql

Però per comentar per sobre, en l'aspecte de les querys MySQL, per explicar una mica he fet servir:
 
- **CREATE TABLESPACE**-*DDL*: Per crear espais de taules, amb un ADD DATAFILE per afegir i crear un directori de dades com ts1.ibd dins de /discs-mysql/disk1,per exemple.  
- **ALTER TABLE**-*DDL*: Per afegir les taules de la base de dades sakila als tablespaces..*
- **INSERT INTO**-*DML*: Per afegir registres a les taules.

**Checkpoint: Mostra al professor els canvis realitzats i que la BD continua funcionant**

<a name="Ac5"></a>
## Activitat 5. REDOLOG. REALITZA ELS SEGÜENTS APARTATS. (2 punt)  
* [Tornar al ÍNDEX](#index)


**Com podem comprovar (Innodb Log Checkpointing)**
**LSN (Log Sequence Number)**
**L'últim LSN actualitzat a disc**
**Quin és l'últim LSN que se li ha fet Checkpoint**

Per comprovar el Log Checkpointing d'InnoDB podem fer-ho a través d'aquesta comanda, que és força llarg el resultat però només mostrarà els logs que demana l'activitat, subratllat en groc.  

````mysql
SHOW ENGINE INNODB STATUS\G
````
![image](https://user-images.githubusercontent.com/61285257/160849840-fe2433bd-9131-4d2e-b9dd-d451300a083e.png)  
![image](https://user-images.githubusercontent.com/61285257/160849610-a67af6cf-eee8-4345-be18-7aa0277eca6e.png)

**Proposa un exemple a on es vegi l'ús del redolog**

Per entendre una mica lús dels redologs, deixo aquesta miniteoria.

*Què fan els redo log files?*

Els fitxers de xarxa log registren canvis a la base de dades com a resultat de transaccions o accions internes del servidor Oracle.

*Perquè serveixen els redo log file?*

Protegeixen la base de dades de la pèrdua d'integritat en casos de fallades causades per subministrament elèctric, errors a discos durs.

*Com funcionen els red log files?*

Treballen de manera cíclica. Si un fitxer red log online s'omple  passarà al següent grup de log en el qual es produeix una operació de punt de control (check point).

L'exemple que utilitzare serà ampliar la mida dels red logs d'InnoDB, per així millorar rendiment de la nostra Base de Dades.

Per dur a terme aquesta proposta, farem el següent:

El número o la mida dels fitxers dels redologs es poden canviar amb el procés següent:
- Aturarem el servei MySQL.
- Esborrarem els red logs actuals(ib_logfile) que es troben al directori de dades /var/lib/mysql. Perquè si ho sobreescrivim el més segur és que es pugui arribar a     corrompre el mysql.  
![image](https://user-images.githubusercontent.com/61285257/160861709-bcf942af-1304-4cf2-86a4-1a322428e559.png)

- Per canviar la mida del fitxer de registre, configurem innodb_log_file_size. Per augmentar el nombre de fitxers de registre, configurem innodb_log_files_in_group.
  Tot aixó en la configuració /etc/my.cnf.
  ![image](https://user-images.githubusercontent.com/61285257/160862339-9c5f56fe-6ba3-406e-a372-175af8ef909a.png)
  Abans com podem comprovar aquí, tenim dos logs amb 48M de mida cadascun, ara haurem de millorar-ho, per exemple 3 fitxers de 64M cadascun. Com podem veure a la       configuració.
  
  ![image](https://user-images.githubusercontent.com/61285257/160862662-98086912-2e03-4778-b361-eb9b727253eb.png)

- Arranquem de nou el servei MySQL.

  ![image](https://user-images.githubusercontent.com/61285257/160862984-bc47eac8-bb1a-4abc-81d7-6167120966ef.png)  
 
- Finalment, comprovarem de nou la ruta /var/lib/mysql i veurem els 3 fitxers de 64M cadascun.

  ![image](https://user-images.githubusercontent.com/61285257/160863458-49a6dc04-d448-44b6-86e6-a175f4d6d04e.png)


**Com podem mirar el número de pàgines modificades (dirty pages)? I el número total de pàgines?**

Ho podrem comprovar amb la mateixa comanda que el primer punt.

![image](https://user-images.githubusercontent.com/61285257/160865357-e204a0e9-72b1-488f-8e21-3df2225d34d5.png)

Podem veure que no hi ha cap modificació, perquè no hem modificat cap pàgina (Una pàgina a Innodb conté files, índexs, etc.)

**Checkpoint: Mostra al professor els canvis realitzats i que la BD continua funcionant.**  

<a name="Ac6"></a>
## Activitat 6. Implentar BD Distribuïdes. (1,5 punts)  
* [Tornar al ÍNDEX](#index)

**Com s'ha vist a classe MySQL proporciona el motor d'emmagatzemament FEDERATED que té com a funció permetre l'accés remot a bases de dades MySQL en un servidor local sense utilitzar tècniques de replicació ni clustering.**  


**Prepara un Servidor Percona Server amb la BD de Sakila.**  
**Prepara un segon servidor Percona Server a on hi hauran un conjunt de taules FEDERADES al primer servdor.**  
El motor "FEDERAT= Federated Storage Engine " permet a un usuari de base de dades crear una rèplica local d'una taula present en una base de dades diferent.  

En altres paraules, podrem accedir a dades de taules de bases de dades remotes en lloc de bases de dades locals.  

La meva proposta i objectiu en aquesta activitat és des del servidor local(slave), accedir a les taules de la base de dades sakila, que s'ubiquen al servidor remot(master).  

Accedirem a aquestes dades a base de crear taules amb el motor corresponent, en aquest cas FEDERATED.  

![image](https://user-images.githubusercontent.com/61285257/161570772-b80c995b-2f01-4881-b411-16926fdcd09d.png)  

Tindrem dues maquines un que actuarà com Master i l’altre com Slave.  

*Mestre-esclau (Master-slave en anglès) és un model de protocol de comunicació en què un dispositiu o procés (conegut com a mestre o màster) controla un o més d'altres dispositius o processos (coneguts com a esclaus o slaves).*

Aquestes serien les dades que necessitem saber, per ubicar-nos, tant com de la maquina com del sistema de gestió:  

![image](https://user-images.githubusercontent.com/61285257/161571071-a5afa930-1b56-4a66-b96f-81d7c5ffd217.png)

Creació d’usuaris:

![image](https://user-images.githubusercontent.com/61285257/161571150-171fdfd9-ec1d-4367-8e7a-02447eae5014.png)

![image](https://user-images.githubusercontent.com/61285257/161571186-146e1ae4-f800-407a-89ea-f592f462e81d.png)

![image](https://user-images.githubusercontent.com/61285257/161571228-202bbc67-6e93-4b46-9fa2-5808668bf85d.png)

![image](https://user-images.githubusercontent.com/61285257/161571256-46974a6b-bae1-4315-8c05-e162e3423b74.png)

![image](https://user-images.githubusercontent.com/61285257/161571301-290f2c18-f665-422c-8e28-a0a4450b99d6.png)

![image](https://user-images.githubusercontent.com/61285257/161571339-5bc9456d-3389-4319-878a-1c96d9b9e3e5.png)![image](https://user-images.githubusercontent.com/61285257/161571369-2e6753b4-38cc-44fc-a735-f4c8af40cc34.png)

![image](https://user-images.githubusercontent.com/61285257/161571599-b09b06ab-6e74-424a-b916-a3229f8975a8.png)

Farem les següents configuracions en el Slave.  
L'opció del motor FEDERATED no cal que estigui habilitada als dos servidors, en aquest cas, només cal que estigui habilitada el servidor local(slave) on s'emmagatzema/crearem la taula federada.  

Deshabilitarem la seguretat SELinux per si ens arriba a donar problemes a l'hora d'accedir a la base de dades del servidor master.  

![image](https://user-images.githubusercontent.com/61285257/161571716-8b7393bf-c9ab-4a36-acfd-e4623c721405.png)

![image](https://user-images.githubusercontent.com/61285257/161571727-d430048a-677f-456f-8d59-6eb60aa049f2.png)

Després d’habilitar el motor en la configuració, reiniciarem el servei mysql.  

![image](https://user-images.githubusercontent.com/61285257/161571760-913a82ed-333c-4d1b-9013-32064e0d728d.png)

Comprovarem els canvis aplicats, podem veure que tenim el motor FEDERATED habilitat.  

![image](https://user-images.githubusercontent.com/61285257/161571810-0106e1ab-230e-46ff-9e73-0cb170accc8c.png)

**Per realitzar aquest link entre les dues BD podem fer-ho de dues maneres:**  

**Opció1: especificar TOTA la cadena de connexió a CONNECTION.**

**Opció2: especifficar una connexió a un server a CONNECTION que prèviament s'ha creat mitjançant CREATE SERVER.**  

**Posa un exemple de 2 taules de cada opció.**  


Comencem ara amb el que seria la practica, amb les connexions crearem una taula federated al slave.

![image](https://user-images.githubusercontent.com/61285257/161572081-cad1f158-fb28-4041-89c9-0c694da6749f.png)

Aquestes connexions es poden fer de dues maneres i com que la pràctica ens demana dos exemples de cada opció, accedirem a dues taules de cada opció.  

![image](https://user-images.githubusercontent.com/61285257/161572215-e6c80418-b73b-4c45-9154-cf431a3be6f7.png)

Ens connectarem des del slave a la taula actor(sakila) del master.  

Per tant per simplificar el procés, agafarem la consulta DDL ubicada al sakila-schema.sql de la creació de la taula actor i farem les modificacions necessàries(engine=federated i connexions), per desprès executar-ho en el slave.
Òbviament també canviarem el nom de la taula per diferenciar-ho davant la taula del master.  

![image](https://user-images.githubusercontent.com/61285257/161572296-73ec5a3f-0e60-4320-bc89-c54fa382c000.png)

Aquesta es la plantilla que agafarem i el convertirem tal com així:  

![image](https://user-images.githubusercontent.com/61285257/161572359-541c2b88-7268-4f72-80c0-33f28ec9bf6f.png)

````mysql
CREATE TABLE actor_federated (
  actor_id SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
  first_name VARCHAR(45) NOT NULL,
  last_name VARCHAR(45) NOT NULL,
  last_update TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY  (actor_id),
  KEY idx_actor_last_name (last_name)
)ENGINE=FEDERATED CONNECTION='mysql://aryan:aryan@192.168.1.114:3306/sakila/actor';
````

Aquest procés el replicarem fins al que queda de pràctica.

![image](https://user-images.githubusercontent.com/61285257/161572782-86a6fb1f-50f9-44df-8802-8929140577b9.png)


![image](https://user-images.githubusercontent.com/61285257/161572734-1f2747d7-b8d7-426d-abf9-d7b9e2068626.png)

Afegirem un registre a aquesta taula federada i ho comprovarem.

Crearem la taula federada i podem veure que tenim un registre, que exactament el vam crear en l’activitat 4 això, ens diu que la connexió FEDERATED es successiva.

![image](https://user-images.githubusercontent.com/61285257/161572881-6941915c-27f9-4245-bce0-35f7878d5096.png)

![image](https://user-images.githubusercontent.com/61285257/161572920-00780f46-33ad-4bb2-933c-72b32372ace8.png)

Si ho comprovem al servidor remot, podrem veure com ha replicat el registre.

![image](https://user-images.githubusercontent.com/61285257/161572950-5a5f8d85-ca75-4078-ad98-495f121b566f.png)

Farem el mateix procés amb un altre taula.

![image](https://user-images.githubusercontent.com/61285257/161573025-3e5743b0-b620-4562-b416-29a2260f21dd.png)

````mysql
CREATE TABLE language_federated (
  language_id TINYINT UNSIGNED NOT NULL AUTO_INCREMENT,
  name CHAR(20) NOT NULL,
  last_update TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (language_id)
)ENGINE=FEDERATED CONNECTION='mysql://aryan:aryan@192.168.1.114:3306/sakila/language';
````

![image](https://user-images.githubusercontent.com/61285257/161573171-59fea5cf-e248-4120-a32b-34151c7b7cd0.png)

Crearem la taula federada, ens agafa tambè el registre afegit en l'activitat 4, pero igualment tornarem a fer les comprovacions.

![image](https://user-images.githubusercontent.com/61285257/161573205-36329466-68bb-40d1-a722-2ba0224fe86a.png)

Afegirem un registre a aquesta taula federada i ho comprovarem.

![image](https://user-images.githubusercontent.com/61285257/161573245-d0f11d37-8ade-473a-ac6d-93a296b18fa4.png)  

![image](https://user-images.githubusercontent.com/61285257/161573282-d4200f77-6bf0-4b7d-b12e-02dd6c1a00e7.png)

Si ho comprovem al servidor remot, podrem veure com ha replicat el registre.

![image](https://user-images.githubusercontent.com/61285257/161573332-8e58cb35-5b3a-4d64-a6a4-87bf238e7d5e.png)  

![image](https://user-images.githubusercontent.com/61285257/161573381-d9d1747f-acfd-47c4-b243-fa9aea16e664.png)

Si estem creant diverses taules FEDERATED al mateix servidor, o si volem simplificar el procés de creació de taules FEDERATED, podem utilitzar la instrucció CREATE SERVER per definir els paràmetres de connexió del servidor, tal com ho faríem amb la cadena CONNECTION.

![image](https://user-images.githubusercontent.com/61285257/161573459-55896e63-ec02-4b59-98b8-ce31e3ee63ee.png)

El format de la instrucció CREATE SERVER és:

````mysql
CREATE SERVER master
FOREIGN DATA WRAPPER mysql
OPTIONS (USER 'aryan', PASSWORD 'aryan', HOST '192.168.1.114', PORT 3306, DATABASE 'sakila');
````

![image](https://user-images.githubusercontent.com/61285257/161573574-1a8dad5b-0a11-48d3-a292-743eaefe6265.png)

Ara tindríem la connexió feta, a l’hora de crear les taules federades nomes hauríem de especificar el servidor i la taula.  

````mysql
CREATE TABLE country_federated (
  country_id SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
  country VARCHAR(50) NOT NULL,
  last_update TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY  (country_id)
)ENGINE=FEDERATED CONNECTION='master/country';
````

Crearem la taula federada i afegirem un registre.
![image](https://user-images.githubusercontent.com/61285257/161573738-3bb6e8e0-02d9-422d-b252-3fee8b3459a4.png)

![image](https://user-images.githubusercontent.com/61285257/161573771-6cca77ce-cfd6-4460-875f-a2fbf3650b5b.png)

Si ho comprovem al servidor remot, podrem veure com ha replicat el registre.

![image](https://user-images.githubusercontent.com/61285257/161573788-c659745a-ecfa-4f60-aeec-1547030c08a7.png)

![image](https://user-images.githubusercontent.com/61285257/161573838-b30f172c-53d7-4899-8cb0-459c4de137e2.png)

Exactament el mateix procés.

````mysql
CREATE TABLE category_federated (
  category_id TINYINT UNSIGNED NOT NULL AUTO_INCREMENT,
  name VARCHAR(25) NOT NULL,
  last_update TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY  (category_id)
)ENGINE=FEDERATED CONNECTION='master/category';
````

Crearem la taula federada,fegirem un registre a aquesta taula federada i ho comprovarem.  

![image](https://user-images.githubusercontent.com/61285257/161573984-f96b0d9c-76b6-42df-872c-884e45ffde8f.png)

![image](https://user-images.githubusercontent.com/61285257/161574040-4f119efa-64e7-4ac1-924e-412fa99606b7.png)


Si ho comprovem al servidor remot, podrem veure com ha replicat el registre.

![image](https://user-images.githubusercontent.com/61285257/161574055-e4a1fc71-b10f-4fc1-bb99-997e52d50db6.png)




**Tingues en compte els permisos a nivell de BD i de SO així com temes de seguretat com firewalls, etc...**  

Els usuaris tenien privilegis de root en tot moment, el tallafocs en aquesta pràctica estava deshabilitat i també hem deshabilitat les seguretat selinux que de vegades pot provocar interferències entre els dos servidors.

**Detalla quines són els passos i comandes que has hagut de realitzar en cada màquina.**

Aquest exercici s'ha documentat mentre es feia, als apartats corresponents.

<a name="Ac7"></a>   
## Activitat 7. Storage Engine CSV (0,5 punts)  
* [Tornar al ÍNDEX](#index)


**Documenta i posa exemple de com utilitzar ENGINE CSV.**

Les taules CSV es poden utilitzar per importar dades del programa de full de càlcul preferit( per exemple,excel), intercanviar dades des d'altres fonts de dades o amb altres o simplement migrar dades d'un altre sistema de bases de dades a MySQL.

Per habilitar les taules CSV, que es un motor d'emmagatzematge, podem utilitzar l'opció --with-csv-storage-engine en el /etc/my.cnf.

Pero des de la versio de MySQL 5.0 ja ve activat, per defecte.

Per comprovar que esta disponible podem executar la seguent consulta:


```mysql
SHOW ENGINES;
``` 
![image](https://user-images.githubusercontent.com/61285257/160831446-53223fdd-8a0a-434a-8e8a-8963370aefae.png)


**Cal documentar els passos que has hagut de realitzar per preparar l'exemple: configuracions, instruccions DML, DDL, etc....**

Per demostrar com funciona farem consultes DDL i DML bàsiques.

Primer crearem una base de dates i comprovarem l'existencia d'aquesta.

```mysql
CREATE DATABASE CSV;
SHOW DATABASES;
``` 

![image](https://user-images.githubusercontent.com/61285257/160832286-a4457c34-3bb1-45af-a646-2b7ee4021b41.png)

Ara ens toca crear la taula CSV.
Quan creem una taula CSV, el servidor crea un fitxer de definició de taula al directori de la base de dades. 
El fitxer comença amb el nom de la taula i té una extensió .frm.  

El motor d'emmagatzematge també crea un fitxer de dades. El seu nom comença amb el nom de la taula i té una extensió .CSV.

Al final de tot, comprovarem l'existencia dels fitxers de dades.

Com sempre, quan volem crear taules cobren un motor tenim que especificar amb la consulta, en aquest cas ENGINE = csv;
	
```mysql
USE CSV;
CREATE TABLE test (i INT NOT NULL, c CHAR(10) NOT NULL) ENGINE = CSV;
``` 

![image](https://user-images.githubusercontent.com/61285257/160833251-71c0355c-cedd-486d-904f-7c2d0117c1b7.png)

Afegirem registres a aquesta taula i desprès ho comprovem:

```mysql
INSERT INTO test VALUES(1,'record one'),(2,'record two');
SELECT * FROM test;
``` 

![image](https://user-images.githubusercontent.com/61285257/160833736-ec38009d-839b-474c-a40f-3b1d47ab9bfe.png)

Podem comprovar que tota la creació es correcte.

Ara comprovarem els fitxters de dades que es genereren,que com vam comentar el principi.
Mirarem nomes el contingut del .CSV ja que .frm com sabem sempre estara encryptat.

![image](https://user-images.githubusercontent.com/61285257/160834703-c0bcdf43-3848-4f53-8d33-0b9b72c52214.png)

![image](https://user-images.githubusercontent.com/61285257/160840280-9cbd12f7-bac6-4e12-b4ef-59098e134b73.png)

Podem veure fins a la mida daquests arxius, insignificants.  

Ara amb el fitxer .csv podem llegir, i fins i tot escriure, mitjançant aplicacions de full de càlcul com Microsoft Excel.  

**Checkpoint: Mostra al professor la configuració que has hagut de realitzar i el seu funcionament.**  

<a name="Ac8"></a>
## Activitat 8. Storage Engine MyRocks (1 punt)  
* [Tornar al ÍNDEX](#index)


MyRocks és un motor d'emmagatzematge per MySQL basat en RocksDB (SGBD incrustat de tipus clau-valor). 
Aquest tipus d’emmagatzematge està optimitzat per ser molt eficient en les escriptures amb lectures acceptables.  

**Documenta i posa exemple de com utilitzar ENGINE MyRocks. Crea una Base de dades amb 2 o 3 taules i inserta-hi contingut.**

Si hem fet l'activitat un, hauríem de tenir el myrocks instal·lat, l'instal·lem amb suo "yum install Percona-Server-rocksdb-57" i ho habilitem amb "ps-admin --enable-rocksdb -u <mysql_admin_user> -p[mysql_admin_pass ]".

Per comprovar que tenim aquest enginy habilitat ho farem amb:

```mysql
SHOW STORAGE ENGINES;
``` 

![image](https://user-images.githubusercontent.com/61285257/160836913-ad3fd5ed-7282-47bf-90d3-34c48f30a0c8.png)

        
**Cal documentar els passos que has hagut de realitzar per preparar l'exemple: configuracions, instruccions DML, DDL, etc....**

Bàsicament, en el tema DML i DDL, vull dir, les creacions de la basedades,taules, registres ho farem igual que lexercici anterior, òbviament amb la diferència de crear les taules amb el motor ROCKSDB.

Després al final mostrare, el que seria el tema de la compressió.

```mysql
CREATE DATABASE ROCKS;
SHOW DATABASES;
``` 
![image](https://user-images.githubusercontent.com/61285257/160837655-61a85901-3da1-4fea-9fe4-7670c6710929.png)


Un cop creada la base de dades de dades per realitzar la prova creem un parell de taules on especifiquem que el Storage Engine és el RocksDB i afegim registres per fer la prova i desprès comprovem aquests registres.

```mysql
USE ROCKS;
CREATE TABLE test (i INT NOT NULL, c CHAR(10) NOT NULL) ENGINE = RocksDB;
CREATE TABLE testROCKS (i INT NOT NULL, c CHAR(10) NOT NULL) ENGINE = RocksDB;
``` 
![image](https://user-images.githubusercontent.com/61285257/160838932-8abe8c40-e226-499d-946a-7d7943482b7e.png)


```mysql
INSERT INTO test VALUES(1,'record one'),(2,'record two');
INSERT INTO testROCKS VALUES(1,'RocksDB'),(2,'RocksDB');
SELECT * FROM test;
SELECT * FROM testROCKS;
``` 
![image](https://user-images.githubusercontent.com/61285257/160839036-37417555-9e06-485d-ab8f-f6032c4549f5.png)

Base de dades creada i modificada correctament.
    
**A quin directori es guarden els fitxers de dades? Fes un llistat de a on són els fitxers i què ocupen.**

Com sempre, es crearà una carpeta corresponent al nom de la base de dades al directori de dades, i aquesta contè els fitxers de dades.

![image](https://user-images.githubusercontent.com/61285257/160839951-05fc3a83-eacb-4886-8cee-5cc515a2fe4e.png)

Podem observar la mida dels fitxers .frm (contenen informació relacionada amb el format i l'estructura de bases de dades MySQL).
8,4K cadascun.

**Quina és la compressió per defecte que utilitza per les taules? Com ho faries per canviar-lo. Per exemple utilitza Zlib o ZSTD o sense compressió.**

Per defecte els arxius on estan els taules i les dades estan guardades e encryptades(compressió) en KLZ4, respaldades enun fitxer .frm, com ho hem vist en l'anterior captura.

Per saber-ho, utilitzarem aquesta consulta DML:

````mysql
SELECT * FROM INFORMATION_SCHEMA.ROCKSDB_CF_OPTIONS 
WHERE OPTION_TYPE LIKE '%ompression%' and cf_name='default';
````

![image](https://user-images.githubusercontent.com/61285257/161578544-c598a355-5000-4503-8bec-371e0bd39893.png)

Com podem veure i com he comentat abans, la compressió la tenim a ZLZ4. En el meu cas la canviaré a ZSTD. 
Ho farem afegint aquesta línia al fitxer de configuració /etc/my.cnf

````bash
[mysqld]
rocksdb_default_cf_options=compression=kZSTD;bottommost_compression=kZSTD
````


![image](https://user-images.githubusercontent.com/61285257/161592943-46aebb12-6160-4822-9492-426d2842cddf.png)

Desem els canvis i reiniciem el servei mysql.

![image](https://user-images.githubusercontent.com/61285257/161593047-139c0c0f-b5ad-4858-b013-7f8317c3dc9b.png)


Per comprovar que hem canviat el tipus de compressió, farem servir la mateixa consulta DML esmentada al principi.


![image](https://user-images.githubusercontent.com/61285257/161593111-61e50ccd-049a-4ad3-a372-b0218ebbc83d.png)

Compressió actualitzat a ZSTD.



#### També podem fer un without-compress, com a alternativa(opcional).

Per canviar aixó utilitzarem una eina de linux anomenada Hexdump.

Hexdump és una utilitat que mostra el contingut dels fitxers binaris en hexadecimal, decimal, octal o ASCII.

Farem la prova només amb una taula, per veure com funciona.

Indiquem juntament amb l'eina, els paràmetres -v(fa que hexdump mostri totes les dades dentrada. Aquesta opció és útil quan es realitza lanàlisi de la sortida completa de qualsevol cadena o text.)  
També el parametre -C(mostra el desplaçament d'entrada en hexadecimal. La cadena de sortida serà seguida per setze caràcters de dades d'entrada separats per espais, tres columnes, plens d'espais, per línia.)  

```bash
hexdump -v -C testROCKS.frm
``` 
*La comanda es molt llarga, lògicament. Per tant mostrare els resultats del principi,meitat i final,*

![image](https://user-images.githubusercontent.com/61285257/160844702-5f0f2e76-8455-4c5a-b103-6092120911c4.png)
![image](https://user-images.githubusercontent.com/61285257/160844319-5731b532-8837-4168-b0fe-3d5ae95df6fb.png)
![image](https://user-images.githubusercontent.com/61285257/160844376-90dae3f3-e691-46ca-ac2e-65faaa7a6aad.png)

**Checkpoint: Mostra al professor la configuració que has hagut de realitzar i el seu funcionament.**
