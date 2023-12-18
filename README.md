# Actividad 3 - Conceptos y Comandos básicos del particionamiento  en bases de datos NoSQL
## Requerimientos No Funcionales
1. Escalamiento horizontal de datos.
2. Fragmentación de bases de datos.
3. La escritura se debe realizar desde un solo servidor.
4. El grupo de partición debe estar compuesto por servidores de configuración, enrutadores de consultas y shards.
5. El sistema debe emplear particionamiento basado en hash. 

## Comandos
|Comando|Función|
|--|--|
|md|Crea directorio en ruta especifica|
|--port|Indica el puerto al que se hace el llamado|
|--dbpath|Indica el directorio que contiene los datos|
|--configsvr|Especifica el servidor de configuración|
|--configdb|Asigna los enrutadores de consulta|
|--shardsvr|Crea un shard en una ruta especifica|

## Scripts
Creación de directorios:
<pre><code>
md C:\data\repl\n1 
md C:\data\repl\n2 
md C:\data\repl\n3 
md C:\data\repl\n4 
md C:\data\repl\n5 
md C:\data\repl\n6 

md C:\data\config\n1 
md C:\data\config\n2 
md C:\data\config\n3 
</code></pre>

Creación de nodos de conjunto de replicas: 
<pre><code>
mongod --port 27017 --dbpath \data\repl\n1 --replSet rs0
mongod --port 27018 --dbpath \data\repl\n2 --replSet rs0
mongod --port 27019 --dbpath \data\repl\n3 --replSet rs0
</code></pre>

Adición de nodos secundarios desde el puerto 27017: 
<pre><code>
rs.add("192.168.0.18:27018")
rs.add("192.168.0.18:27019")
</code></pre>

Llamado a base de datos:
<pre><code>
use myfootballnine 
</code></pre>

Activación de servidores:
<pre><code>
mongod --port 27022 --dbpath \data\config\n1 --configsvr --bind_ip 192.168.0.18
mongod --port 27023 --dbpath \data\config\n2 --configsvr --bind_ip 192.168.0.18
mongod --port 27024 --dbpath \data\config\n3 --configsvr --bind_ip 192.168.0.18
</code></pre>

Activación de enrutador:
<pre><code>
mongos --configdb 192.168.0.18:27022,192.168.0.18:27023,192.168.0.18:27024 --port  27021 --bind_ip 192.168.0.18
</code></pre>

Asignación de réplica inicial como shard:
<pre><code>
mongo --port 27021 --host 192.168.0.18
</code></pre>

Creación de segundo shard:
<pre><code>
mongod --port 27013 --dbpath \data\repl\n4 --replSet rs1
mongo --port 27013
rs.initiate()
mongod --port 27014 --dbpath \data\repl\n5 --replSet rs1
mongod --port 27015 --dbpath \data\repl\n6 --replSet rs1
</code></pre>

Adición de nodos secundarios desde el puerto 27013:
<pre><code>
rs.add("192.168.0.18:27014")
rs.add("192.168.0.18:27015")
</code></pre>

Adición de réplica como shard:
<pre><code>
sh.addShard("rs1\192.168.0.18:27013,192.168.0.18:27014,192.168.0.18:27015")
sh.status()
</code></pre>

Distribución de colección desde enrutador:
<pre><code>
use myfootballnine
sh.enambleSharding("myfootballnine")
db.ensayo.createIndex({"_id":1})
sh.shardCollection("myfootballnine.ensayo",{"_id:1"}){ "collectionsharded" : "myfootballnine.ensayo", "ok" : 1 }
</code></pre>
