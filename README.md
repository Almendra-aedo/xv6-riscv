Tarea 3 — Grupo H
Integrantes: Almendra Aedo — Gabriel Basualto
Introducción

El sistema operativo xv6, desarrollado para la arquitectura RISC-V, implementa administración de memoria mediante tablas de páginas y un conjunto de bits que definen los permisos asociados a cada página: lectura, escritura, ejecución y accesibilidad desde modo usuario.

El objetivo de esta tarea fue extender xv6 incorporando un mecanismo de protección de lectura para páginas de usuario, mediante la creación de dos nuevas llamadas al sistema:

mrdprotect(void *addr, int len)

munrdprotect(void *addr, int len)

Estas funciones modifican los Page Table Entries (PTE) correspondientes al rango entregado por el usuario, permitiendo deshabilitar y posteriormente restaurar el permiso de lectura sin afectar otros permisos. Finalmente, se implementó un programa de prueba para validar el correcto funcionamiento de la extensión.

Conceptos teóricos
Syscall

Mecanismo mediante el cual un programa en modo usuario solicita servicios del kernel. xv6 implementa un pipeline específico para registrar, declarar y exponer syscalls hacia programas de usuario.

Tabla de páginas

Estructura jerárquica que mapea direcciones virtuales a marcos físicos. Cada entrada (PTE) contiene:

PTE_V: marca la entrada como válida.

PTE_R: habilita lectura.

PTE_W: habilita escritura.

PTE_X: habilita ejecución.

PTE_U: permite acceso desde modo usuario.

Protección por hardware

Si un proceso intenta leer una página cuyo PTE no contiene PTE_R, la MMU genera un page fault y xv6 finaliza el proceso.

Alineación de páginas

Dado que xv6 trabaja con páginas de tamaño fijo (PGSIZE = 4096 bytes), toda dirección utilizada para protección debe estar exactamente alineada al inicio de una página.

Archivos modificados en xv6
Archivo	Modificación realizada
syscall.h	Se agregaron SYS_mrdprotect y SYS_munrdprotect.
syscall.c	Declaración extern y registro en la tabla syscalls[].
sysproc.c o archivo equivalente	Implementación de la lógica de ambas syscalls.
vm.c	Uso de funciones de apoyo como walk() para obtener PTE.
user.h	Prototipos accesibles a programas de usuario.
usys.pl	Generación de wrappers con entry("mrdprotect") y entry("munrdprotect").
Makefile	Inclusión del ejecutable de prueba _rdprotect_test.
Desarrollo
Implementación de mrdprotect(void *addr, int len)

Esta función recorre todas las páginas en el rango [addr, addr + len * PGSIZE) y elimina el permiso de lectura de cada PTE.

Pasos realizados en el kernel:

Obtención de parámetros desde espacio usuario mediante argaddr() y argint().

Validación de condiciones de error:

addr debe estar alineada a PGSIZE.

len debe ser mayor que cero.

Recorrido página por página:

Obtener la entrada con walk(pagetable, va, 0).

Verificar que la PTE exista (PTE_V) y sea accesible desde usuario (PTE_U).

Modificación del flag:

*pte &= ~PTE_R;

Retorno de 0 en caso de éxito o -1 si ocurre algún error.

Implementación de munrdprotect(void *addr, int len)

Funciona de forma simétrica a la anterior, restaurando el permiso de lectura mediante:

*pte |= PTE_R;


Con las mismas validaciones y estructura general.

Programa de prueba

El programa de usuario realiza la siguiente secuencia:

Reserva una página con sbrk(PGSIZE).

Escribe un valor en ella (debe funcionar).

Llama a mrdprotect(addr, 1) para deshabilitar lectura.

Intenta escribir nuevamente (debe seguir funcionando).

Intenta leer (debe causar un page fault).

Llama a munrdprotect(addr, 1) y verifica que la lectura vuelva a ser posible.

Este flujo valida que la protección afecta exclusivamente a la operación read y no a write.

Ejecución del programa en xv6
Compilación y ejecución:
make qemu


Dentro de xv6:

rdprotect_test

Resultado esperado:

Escrituras permitidas bajo protección.

Lectura prohibida, generando un page fault.

Lectura restaurada luego de munrdprotect.

Problemas comunes y soluciones
Problema	Causa	Solución
Page fault antes de aplicar protección	PTE inválida	Validar PTE_V y PTE_U antes de modificar flags
Syscall no disponible	Falta de registro en usys.pl, syscall.h o syscall.c	Verificar que aparezca en los tres archivos
Error de alineación	Dirección no múltiplo de PGSIZE	Validar addr % PGSIZE == 0
Rango fuera del espacio del proceso	Página no mapeada	Comprobar retorno de walk()
Conclusión

Se extendió el sistema operativo xv6 agregando un mecanismo de protección de lectura por página mediante las syscalls mrdprotect() y munrdprotect().

La implementación respeta el modelo de memoria virtual de xv6, manteniendo escritura permitida sobre páginas protegidas y generando una excepción de hardware ante intentos de lectura. El programa de prueba confirma el comportamiento esperado, demostrando el correcto funcionamiento de las modificaciones introducidas.

Evidencia

<img width="738" height="310" alt="image" src="https://github.com/user-attachments/assets/d0761d43-3ea2-4566-9998-6e9af651865c" />

<img width="647" height="845" alt="image" src="https://github.com/user-attachments/assets/3f1f1535-6de2-4e11-a01a-0d3a807e2c3e" />
<img width="1005" height="968" alt="image" src="https://github.com/user-attachments/assets/91b68836-c0f3-4cbd-b774-95bf48d2e59c" />


<img width="567" height="787" alt="image" src="https://github.com/user-attachments/assets/29848696-a981-49ca-ba6c-dffe82702023" />

<img width="742" height="571" alt="image" src="https://github.com/user-attachments/assets/d6f019ac-ec8d-4896-850c-6c1868442dcf" />
<img width="227" height="173" alt="image" src="https://github.com/user-attachments/assets/e538c290-ea2f-40d8-bd7c-a727d423b5ee" />

<img width="1027" height="700" alt="image" src="https://github.com/user-attachments/assets/f9df5af0-d440-4042-aa6b-7cef57fc49e0" />










