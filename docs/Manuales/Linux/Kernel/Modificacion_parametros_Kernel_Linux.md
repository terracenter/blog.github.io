---
title: Modificación de los parámetros del kernel
---
Fuente(s):

* https://www.ibm.com/docs/es/db2/11.1?topic=unix-modifying-kernel-parameters-linux


`ipcs`
----

Muestra información sobre las facilidades de comunicación entre procesos para las cuales el proceso llamante tiene acceso de lectura. De forma predeterminada, muestra información sobre los tres recursos:

* Segmentos de memoria compartida

  La memoria compartida en Linux es un mecanismo que permite a dos o más procesos acceder y modificar la misma zona de memoria. Es una forma rápida y eficiente de comunicación entre procesos, ya que evita la necesidad de copiar datos entre ellos.
* Colas de mensajes

  Las colas de mensajes en Linux son un mecanismo de comunicación entre procesos (IPC) que permite a los procesos enviar y recibir mensajes de forma ordenada y eficiente.
* Arrays de semáforos.




| Parámetro del kernel de IPC       | Valor mínimo impuesto                                                           |
| --------------------------------- | ------------------------------------------------------------------------------- |
| **kernel.shmmni** (**SHMMNI**)    | 256 *<size of RAM in GB>                                                      |
| **kernel.shmmax** (**SHMMAX**)    | <size of RAM in bytes>^1^                                                     |
| **kernel.shmall** (**SHMALL**)    | 2 * < tamaño de RAM en el tamaño de página predeterminado del sistema >^2^ |
| **kernel.sem** (**SEMMNI**)       | 256 *<size of RAM in GB>                                                      |
| **kernel.sem** (**SEMMSL**)       | 250                                                                           |
| **kernel.sem** (**SEMMNS**)       | 256 000                                                                       |
| **kernel.sem** (**SEMOPM**)       | 32                                                                            |
| **kernel.msgmni** (**MSGMNI**)    | 1 024 *<size of RAM in GB>                                                    |
| **kernel.msgmax** (**MSGMAX**)    | 65.536                                                                        |
| **kernel.msgmnb** (**MSGMNB**)    | 65 536 ^3^                                                                    |

* SHMALL limita la cantidad total de memoria compartida virtual que puede asignarse en un sistema. 

* La gestión de memoria dinámica es el proceso de aumentar y disminuir el uso de memoria real en áreas separadas de memoria compartida virtual. Para dar soporte eficazmente a la preasignación de memoria y a la gestión de memoria dinámica, los servidores de datos (Base de Datos) deben asignar, con frecuencia, más memoria compartida virtual en un sistema que la cantidad de RAM física. El kernel requiere este valor como un número de páginas

* El rendimiento de la carga podría beneficiarse de un límite de tamaño de cola de mensajes mayor, especificado en bytes mediante **MSGMNB**. Puede ver el uso de la cola de mensajes ejecutando el mandato `ipcs -q`. Si las colas de mensajes están a su máxima capacidad, o a punto de alcanzarla, durante las operaciones de carga, piense en aumentar el número de bytes del límite de tamaño de la cola de mensajes.