# Documentación de PostgreSQL 16.4

El Grupo de Desarrollo Global de PostgreSQL

Copyright © 1996-2024 Grupo de Desarrollo Global de PostgreSQL

Aviso legal

Tabla de contenido

Prefacio

* 1.¿Qué es PostgreSQL?
* 2.Breve historia de PostgreSQL
* 3.Convenciones Convenciones
* 4.Información adicional
* 5.Directrices para la notificación de errores

I. Tutorial 

* 1.Introducción
* 2.El lenguaje SQL
* 3.Características avanzadas

II. El lenguaje SQL

* 4.Sintaxis SQL
* 5.Definición de datos 
* 6.Manipulación de datos
* 7.Consultas 
* 8.Tipos de datos 
* 9.Funciones y operadores 
* 10.Conversión de tipos
* 11.Índices 
* 12.Búsqueda de texto completo
* 13.Control de concurrencia
* 14.Consejos de rendimiento
* 15.Consulta paralela

III. Administración del servidor 

* 16.Instalación desde binarios
* 17.Instalación desde código fuente 
* 18.Instalación desde código fuente en Windows 
* 19.Instalación y funcionamiento del servidor
* [20.Configuración del servidor](III.Administracion_de_servidores/Capitulo_20.Configuracion_del_servidor/20.00.Configuracion_del_servidor.md)  
* 21.Autenticación de clientes
* 22.Roles de Base de Datos 
* 23.Gestión de bases de datos 
* 24.Localización 
* 25.Tareas rutinarias de mantenimiento de bases de datos 
* 26.Copia de Seguridad y Restauración 
* 27.Alta Disponibilidad, Equilibrio de Carga y Replicación 
* 28.Monitorización de la Actividad de la Base de Datos 
* 29.Monitorización del Uso del Disco 
* 30.Fiabilidad y el registro de escritura anticipada 
* 31.Replicación lógica Replicación lógica 
* 32.Compilación Just-in-Time (JIT) 
* 33.Pruebas de regresión Pruebas de regresión

IV. Interfaces de cliente 

* 34.libpq - Biblioteca C 
* 35.Objetos grandes Objetos grandes 
* 36.ECPG - SQL incrustado en C 
* 37.Esquema de información El esquema de información

V. Programación de servidores 

* 38.Extensión de SQL 
* 39.Disparadores 
* 40.Activadores de eventos 
* 41.El sistema de reglas 
* 42.Lenguajes procedimentales 
* 43.PL/pgSQL - Lenguaje de procedimiento SQL 
* 44.PL/Tcl - Lenguaje de procedimiento Tcl 
* 45.PL/Perl - Lenguaje de Procedimientos Perl PL/Perl
* 46.PL/Python - Lenguaje de procedimiento Python 
* 47.Interfaz de programación de servidor 
* 48.Procesos de trabajo en segundo plano 
* 49.Decodificación Lógica 
* 50.Seguimiento del progreso de la replicación 
* 51.Módulos de archivo

VI. Referencia 

* I.Comandos SQL 
* II.Aplicaciones Cliente PostgreSQL 
* III. Aplicaciones de Servidor PostgreSQL

VII. Internos 

* 52.Visión General de los Sistemas Internos de PostgreSQL 
* 53.Catálogos de Sistema 
* 54.Vistas del sistema 
* 55.Protocolo Frontend/Backend 
* 56.Convenciones de Codificación PostgreSQL 
* 57.Soporte de Lenguaje Nativo 
* 58.Escribir un Manejador de Lenguaje Procedimental 
* 59.Escritura de una Envoltura de Datos Externos 
* 60.Escritura de un Método de Muestreo de Tabla 
* 61.Escritura de un Proveedor de Escaneo Personalizado 
* 62.Optimizador Genético de Consultas 
* 63.Definición de la Interfaz del Método de Acceso a Tablas 
* 64.Definición de la interfaz del método de acceso a índices 
* 65.Registros WAL genéricos 
* 66.Gestores de Recursos WAL Personalizados 
* 67.Índices B-Tree 
* 68.Índices GiST 
* 69.Índices SP-GiST 
* 70.Índices GIN 
* 71.Índices BRIN 
* 72.Índices Hash 
* 73.Almacenamiento físico de bases de datos 
* 74.Procesamiento de Transacciones 
* 75.Declaraciones del Catálogo de Sistema y Contenido Inicial 
* 76.Cómo utiliza las estadísticas el planificador 
* 77. Formato del Manifiesto de Copia de Seguridad

VIII. Apéndices 

* A.Códigos de Error PostgreSQL 
* B.Soporte de Fecha/Hora 
* C.Palabras Clave SQL 
* D.Conformidad SQL 
* E.Notas de la Versión 
* F.Módulos y Extensiones Adicionales Suministrados 
* G.Programas Adicionales Suministrados 
* H.Proyectos Externos 
* I.El Repositorio de Código Fuente 
* J.Documentación 
* K.Límites de PostgreSQL 
* L.Acrónimos 
* M.Glosario 
* N.Soporte de Colores 
* O.Características Obsoletas o Renombradas

Bibliografía

Índice
