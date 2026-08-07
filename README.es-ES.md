

# Notas de Artículos

[![github pages](https://github.com/zhjwpku/paper-notes/actions/workflows/gh-pages.yml/badge.svg)](https://github.com/zhjwpku/paper-notes/actions/workflows/gh-pages.yml)

He  modificado este repositorio a mdBook. Ejecuta `mdbook serve` para leer las notas localmente o simplemente visita [paper-notes.zhjwpku.com](https://paper-notes.zhjwpku.com).

## Tabla de Contenidos (TOC)

- [Notas de Artículos](#论文笔记)
  - [Tabla de Contenidos (TOC)](#目录toc)
  - [Estructuras de Datos (Data Structure)](#数据结构data-structure)
    - [BTree](#btree)
      - [**The Ubiquitous B-Tree**](#the-ubiquitous-b-tree)
    - [LSMTree](#lsmtree)
      - [**The Log-Structured Merge-Tree**](#the-log-structured-merge-tree)
  - [Sistemas Distribuidos (Distributed Systems)](#分布式distributed-systems)
    - [Almacenamiento (Storage)](#存储storage)
      - [**Bigtable: A Distributed Storage System for Structured Data**](#bigtable-a-distributed-storage-system-for-structured-data)
  - [Memoria Persistente (Persistent Memory)](#persistent-memory)
      - [**System Evaluation of the Intel Optane Byte-addressable NVM**](#system-evaluation-of-the-intel-optane-byte-addressable-nvm)
      - [**An Empirical Guide to the Behavior and Use of Scalable Persistent Memory**](#an-empirical-guide-to-the-behavior-and-use-of-scalable-persistent-memory)

## Estructuras de Datos (Data Structure)

### BTree

#### **[The Ubiquitous B-Tree](http://carlosproal.com/ir/papers/p121-comer.pdf)**

> Desafortunadamente, un árbol B puede no funcionar bien en un entorno de procesamiento secuencial. Mientras que un recorrido en preorder simple de un árbol [KNUT68] extrae todas las claves en orden, requiere espacio para al menos h = logd(n + 1) nodos en la memoria principal ya que apila los nodos a lo largo de un camino desde la raíz para evitar leerlos dos veces.

> Además, procesar una operación siguiente puede requerir trazar un camino a través de varios nodos antes de alcanzar la clave deseada.

**B* Tree**

Al insertar en un  B se producen desbordamientos (overflow). Se puede reducir la frecuencia de divisiones mediante redistribución, pero al realizarse la división, se hace en dos mitades, lo que garantiza solo una utilización del espacio del 50%. El B* Tree también utiliza redistribución para reducir las divisiones. Además, solo divide dos nodos en tres cuando ambos nodos vecinos están llenos, logrando que cada nodo alcance un 2/3 de ocupación, lo que reduce la altura total del árbol y optimiza la velocidad de búsqueda. No obstante, su aplicación práctica es limitada.

**B+ Tree**

El B+ Tree mantiene el equilibrio del árbol y conecta todos sus nodos hoja mediante punteros (de izquierda a derecha). Por lo tanto, al iterar secuencialmente a la siguiente clave, solo se requiere una lectura en disco como máximo, y solo un nodo debe residenciar en la memoria. El conjunto de todos los nodos hoja enlazados se denomina `sequence set`. Los nodos índice y los nodos hoja del B+ Tree suelen utilizar formatos de disposición diferentes, ya que los nodos índice no requieren información de valor. Las claves en los nodos no hoja actúan como `índice` y contienen un subconjunto de claves. Las operaciones de inserción y eliminación del B+ Tree se pueden probar en [B+ Tree Visualization](https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html).

> Además, cabe destacar que al eliminar datos, se pueden conservar las claves eliminadas en el índice para simplificar la operación.

> Cuando se utiliza un B+ Tree en aplicaciones con alta concurrencia, como las bases de datos, se deben considerar los aspectos relacionados con el `bloqueo`. Dado que las inserciones y eliminaciones pueden modificar los nodos superiores, la concurrencia puede provocar condiciones de carrera.

**Prefix B+ Tree**

*Nota: [BAYE77][baye77] denomina B\* Tree al B+ Tree.*

Dado que el B+ Tree no necesita utilizar la clave completa para construir el `índice`, se propuso el Prefix B+ Tree, que utiliza cadenas más cortas para construir el índice del árbol B+ Tree, comprimiendo así la altura del árbol y mejorando la eficiencia de la búsqueda.

### LSMTree

#### **[The Log-Structured Merge-Tree](https://www.cs.umb.edu/~poneil/lsmtree.pdf)**

El LSM-tree ha captado gran atención en el campo del almacenamiento. A diferencia del costo de rendimiento asociado a las escrituras en tiempo real del B-Tree, el LSM-tree devuelve el resultado inmediatamente después de escribir los datos en el Log (archivo de registro secuencial) y en la memoria, posponiendo y agrupando las operaciones en disco. Esto promedia la latencia de búsqueda y rotación del disco en cada operación de escritura, mejorando así el rendimiento de escritura.

**Puntos Clave del Artículo**

- LSM-Tree es adecuado para escenarios donde la escritura es mucho mayor que la lectura
- LSM-Tree consiste en un árbol C0 residente en memoria y varios árboles Ck residentes en disco
- El árbol C0 puede implementarse como un árbol (2-3) o AVL, mientras que el árbol Ck se implementa como un B-tree
- Cuando el árbol C0 alcanza cierto umbral, migra una parte de los datos al árbol C1. De manera similar, el árbol Ck-1 migra al árbol Ck al alcanzar su umbral. Este proceso se conoce como `rolling merge` continuo
- Durante la fusión, se utiliza E/S de bloques multipágina (256K bytes == 64 páginas), lo que aumenta el ancho de banda
- Los nuevos datos generados durante la fusión no sobrescriben los datos antiguos
- Para permitir una recuperación rápida, la implementación realiza checkpoints del árbol C0 interrumbidamente

El artículo analiza la comparación de costos entre B-Tree y LSM-Tree para alcanzar cierto nivel de E/S, así como el proceso de optimización de las proporciones de tamaño de los diferentes componentes. Para más detalles, consulta el artículo original.

## Sistemas Distribuidos (Distributed Systems)

### Almacenamiento (Storage)

#### **[Bigtable: A Distributed Storage System for Structured Data](https://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf)**

Bigtable es el sistema de almacenamiento distribuido de Google para almacenar datos estructurados. Utiliza GFS en la capa inferior para almacenar datos y archivos de registro. Cuenta con características como escalabilidad, alto rendimiento y alta disponibilidad. Apache HBase es una implementación de código abierto basada en Bigtable.

**Puntos Clave del Diseño**

- Bigtable no tiene un esquema fijo; los datos almacenados suelen ser dispersos y constituyen un mapa ordenado multidimensional
- (row:string, column:string, time:int64) --> string
- En el nivel de vista lógica, incluye conceptos como tabla, fila, columna, versión y celda, similar a un SGBDR
- En el nivel de vista física, debido al concepto de Column Family, los datos se almacenan como K/V, y los datos de la misma CF se ordenan y agrupan juntos
- Dado que una tabla puede ser muy grande y los datos se almacenan ordenados por clave de fila, la tabla se divide en múltiples tabletas almacenadas en diferentes tablet servers
- La Column Family actúa como la unidad básica de control de acceso; debe estar creada antes de almacenar datos en ella
- Una tabla no debe tener demasiadas CFs (máximo unas pocas centenas), pero el número de columnas en una tabla puede ser ilimitado
- Cada celda puede contener múltiples versiones de datos, utilizando un timestamp como índice
- Bigtable solo soporta transacciones a nivel de fila única

**SSTable, MemTable, CommitLog**

Bigtable utiliza SSTable (Sorted Strings Table) para guardar los datos persistentes, incluyendo archivos de datos y archivos de registro.

> Un SSTable proporciona un mapa persistente, ordenado e inmutable de clave a valores, donde tanto las claves como los valores son cadenas de bytes arbitrarias.

Los archivos SSTable guardan una serie de bloques contiguos, típicamente de 64 KB. Al final del archivo se conserva un índice para localizar los bloques (block index), el cual suele cargarse en memoria al abrir el SSTable. De este modo, al buscar datos, primero se realiza una búsqueda binaria del block index en memoria y luego una única lectura en disco para hallar los datos correspondientes a la clave.

El MemTable es un búfer ordenado almacenado en la memoria. Las actualizaciones se registran primero como redo log en el commit log y luego se insertan en el MemTable.

El commit log se utiliza para recuperación tras fallos. Cuando el MemTable alcanza cierto umbral, se convierte en SSTable y finalmente se fusiona con otros SSTables en un único SSTable, proceso conocido como Compaction. Para más detalles, véase la sección 5.4.

Estas tres estructuras garantizan una persistencia de datos efectiva y sin pérdida.

**Arquitectura**

- Bigtable sigue una arquitectura maestro-esclavo típica, dependiendo del servicio de bloqueo distribuido proporcionado por Chubby para garantizar que solo un maestro esté activo al mismo tiempo
- Chubby además de proporcionar el servicio de bloqueo distribuido anterior:
    - Almacena la información del esquema de Bigtable (información de las column families de cada tabla)
    - Se utiliza para el descubrimiento de tablet servers y para detectar cuando dejan de funcionar
- Un maestro, múltiples esclavos:
    - El master se encarga de asignar tabletas a los tablet servers, detectar la unión o salida de tablet servers, equilibrar la carga de los tablet servers, gestionar la recolección de basura (GC) y modificar la información de las tablas
    - Cada tablet server administra una serie de tabletas, y gestiona las lecturas y escrituras correspondientes. Además, los tablet servers son responsables de la división de las tabletas
    - Cada tabla almacena todos los datos dentro de un rango de claves; cada tabla suele tener un tamaño de 100-200 MB
- Los clientes realizan solicitudes de lectura y escritura directamente con los tablet servers
- Para reducir las lecturas en disco, se asignan filtros de Bloom a los SSTables para mejorar la eficiencia en la lectura de datos

La Column Family en Bigtable es un concepto que considero relativamente difícil de comprender. Los lectores pueden acelerar su comprensión combinándola con la documentación y operaciones de HBase. A continuación, se muestran algunos videos relacionados con HBase:

1. [Apache HBase - Just the Basics](https://www.youtube.com/watch?v=2Ci_QxJ1kiE)
2. [HBase Tutorial For Beginners](https://www.youtube.com/watch?v=V1fXSCASVDc)
3. [Introductio to HBase Command Line](https://www.youtube.com/watch?v=_T9-Hmp1mEY)

## Memoria Persistente

La memoria persistente (PMem) actúa como un nuevo medio de almacenamiento que cierra la brecha de rendimiento entre la RAM y las unidades SSD. Sus características incluyen ser no volátil (en comparación con DRAM), tener una mayor capacidad (en comparación con DRAM) y una menor latencia (en comparación con SSD).

#### **[System Evaluation of the Intel Optane Byte-addressable NVM](https://arxiv.org/pdf/1908.06503.pdf)**

La memoria persistente presenta características de alta densidad, bajo consumo de energía y menor costo por bit. Sin embargo, también tiene algunas desventajas: latencia de acceso de 3 a 20 veces superior a la de DRAM, menor ancho de banda y asimetría en el rendimiento de lectura y escritura. Los autores realizaron pruebas y análisis de las características de latencia, ancho de banda y consumo de energía de DRAM y PMem en diferentes configuraciones, y sus conclusiones ofrecen orientaciones útiles para el análisis de rendimiento y la optimización en la programación de memoria persistente.

- Usar DRAM como caché para NVM puede cerrar efectivamente la brecha de rendimiento y acercarlo al de DRAM
- Coordinar accesos de 256B a PMem para explotar la localidad puede reducir la latencia y la amplificación de escritura
- La colocación explícita de datos que utiliza PMem local podría mitigar el alto costo de acceder a DRAM en el socket remoto

#### **[An Empirical Guide to the Behavior and Use of Scalable Persistent Memory](https://www.usenix.org/system/files/fast20-yang.pdf)**

Antes de la aparición de Optane DC Persistent Memory, muchos investigadores estudiaban la NVM mediante simulaciones. Este artículo analiza y corrige las experiencias previas mediante experimentos con DIMMs Optane, y establece cuatro principios para su uso:

1. Evitar lecturas y escrituras aleatorias inferiores a 256 B
2. Utilizar ntstore (non-temporal stores) siempre que sea posible para escrituras de grandes volúmenes de datos, y controlar la evicción del caché
3. Limitar el número de hilos concurrentes que acceden a los DIMMs Optane
4. Evitar accesos NUMA (especialmente en secuencias de operaciones read-modify-write)

[baye77]: https://dl.acm.org/doi/pdf/10.1145/320521.320530
