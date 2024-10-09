# Documentación de PostgreSQL 14.10
El Grupo de Desarrollo Global de PostgreSQL

Copyright © 1996-2023 Grupo de Desarrollo Global de PostgreSQL

Aviso legal

Tabla de contenido

Prefacio
1. ¿Qué es PostgreSQL?
2. Breve historia de PostgreSQL
3. Convenciones
4. Información adicional
5. Directrices para la notificación de errores

I. Tutorial
1. Primeros pasos
2. El lenguaje SQL
3. Funciones avanzadas

II. El lenguaje SQL

4. Sintaxis SQL
5. Definición de datos
6. Manipulación de datos
7. Consultas
8. Tipos de datos
9. Funciones y operadores
10. Conversión de tipos
11. Índices
12. Búsqueda de texto completo
13. Control de concurrencia
14. Consejos de rendimiento
15. Consulta paralela

III. Administración de servidores

16. Instalación a partir de binarios
17. Instalación a partir del código fuente
18. Instalación desde código fuente en Windows
19. Configuración y funcionamiento del servidor
20. Configuración del servidor
21. Autenticación de clientes
22. Funciones de la base de datos
23. Gestión de bases de datos
24. Localización
25. Tareas rutinarias de mantenimiento de la base de datos
26. Copia de seguridad y restauración
27. Alta disponibilidad, equilibrio de carga y replicación
28. Seguimiento de la actividad de la base de datos
29. Supervisión del uso del disco
30. Fiabilidad y registro de anotaciones
31. Replicación lógica
32. Compilación justo a tiempo (JIT)
33. Pruebas de regresión

IV. Interfaces de cliente

34. libpq - Biblioteca C
35. Objetos grandes
36. ECPG - SQL incrustado en C
37. El esquema de información

V. Programación de servidores

38. Ampliación de SQL
39. Triggers
40. Triggers de eventos
41. El sistema de reglas
42. Lenguajes de procedimiento
43. PL/pgSQL - Lenguaje de procedimiento SQL
44. PL/Tcl - Lenguaje procedimental Tcl
45. PL/Perl - Lenguaje procedimental Perl
46. PL/Python - Lenguaje procedimental Python
47. Interfaz de programación del servidor
48. Procesos de los trabajadores en segundo plano
49. Decodificación lógica
50. Seguimiento del progreso de la replicación

VI. Referencia

    I. Comandos SQL
    II. Aplicaciones cliente PostgreSQL
    III. Aplicaciones del servidor PostgreSQL

VII. Internos

51. Visión general de los componentes internos de PostgreSQL
52. Catálogos de sistemas
53. Protocolo Frontend/Backend
54. Convenciones de codificación PostgreSQL
55. Apoyo a la lengua materna
56. Escribir un manejador de lenguaje procedimental
57. Escribir una envoltura de datos ajena
58. Escribiendo una tabla Método de muestreo
59. Escribir un proveedor de escaneo personalizado
60. Optimizador genético de consultas
61. Tabla Método de acceso Definición de interfaz
62. Definición de la interfaz del método de acceso al índice
63. Registros WAL genéricos
64. Índices B-Tree
65. Índices GiST
66. Índices SP-GiST
67. Índices GIN
68. Índices BRIN
69. Índices Hash
70. Almacenamiento físico de bases de datos
71. Declaraciones del catálogo de sistemas y contenido inicial
72. Cómo utiliza las estadísticas el planificador
73. Formato del manifiesto de copia de seguridad

VIII. Apéndices

    A. Códigos de error PostgreSQL
    B. Soporte fecha/hora
    C. Palabras clave SQL
    D. Conformidad SQL
    E. Notas de publicación
    F. Módulos adicionales suministrados
    G. Programas adicionales suministrados
    H. Proyectos externos
    I. Depósito de código fuente
    J. Documentación
    K. Límites de PostgreSQL
    L. Acrónimos
    M. Glosario
    N. Soporte de color
    O. Características obsoletas o renombradas

Bibliografía

Índice




