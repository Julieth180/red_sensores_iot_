# Decisiones de diseño — Semana 3

## 1. Punto de entrada

El proyecto mantiene un único punto de entrada:

```text
IngestaSensores.main()
```

No se crean aplicaciones independientes por semana.

`BancoDePruebas` es una clase auxiliar y no contiene `main`.

## 2. Búsqueda por timestamp

Se utilizan dos estrategias:

- Búsqueda lineal: no requiere ordenamiento.
- Búsqueda binaria: requiere que el arreglo esté ordenado por timestamp.

Los datos sintéticos de `GeneradorDatos` se generan en orden cronológico, por lo que la búsqueda binaria por timestamp cumple su precondición.

## 3. Búsqueda por PM2.5

No se asume que los datos estén ordenados por PM2.5.

Por tanto, la búsqueda binaria por PM2.5 se conserva como experimento para demostrar el efecto de una precondición incumplida.

## 4. Comparación de String

Los identificadores de estación se comparan mediante:

```java
equals()
```

y no mediante:

```java
==
```

porque se necesita comparar contenido.

## 5. Medición

La comparación principal entre algoritmos utiliza el número de comparaciones.

El tiempo en milisegundos se conserva como evidencia experimental, pero no es la única medida utilizada.

## 6. Evolución del proyecto

La Semana 3 agrega una nueva capacidad a la misma plataforma:

```text
Sensores
   ↓
Ingesta
   ↓
Repositorio
   ↓
Búsqueda
   ↓
Medición de eficiencia
```

La Semana 4 podrá extender esta misma arquitectura para estudiar ordenamiento.


## DEC-04 — Pivote de QuickSort

**Semana:** 4

**Problema:**
QuickSort con pivote fijo (el primer elemento) produce particiones
desbalanceadas con datos ordenados. Con 50.000 lecturas desordenadas
hizo 900.318 comparaciones en 167 ms. Con 50.000 lecturas en orden
cronológico, que es como llegan de la red, terminó en
StackOverflowError después de 1.072.699.965 comparaciones.

**Alternativas:**
- Pivote aleatorio.
- Mediana de tres.

**Decisión:**
Usar pivote aleatorio.

**Justificación:**
Con el pivote aleatorio, 50.000 lecturas en orden cronológico se
ordenaron con 941.679 comparaciones y 538.705 intercambios, sin error,
frente a las 930.781 comparaciones del caso desordenado. Con el pivote
fijo, en datos ordenados el primer elemento era siempre el menor y cada
partición dejaba todo el trabajo de un solo lado. Elegir el pivote al
azar evita que eso ocurra de forma sistemática.

**Consecuencia:**
QuickSort ya no falla con los datos como llegan de la red. El número
exacto de comparaciones cambia entre ejecuciones, porque el pivote es
aleatorio, pero se mantiene alrededor de n log n.


## DEC-05 — Ordenamiento y búsqueda

**Semana:** 4

**Problema:**
Ordenar por PM2.5 destruye el orden por timestamp. En el experimento 5,
la búsqueda binaria por timestamp encontró la lectura en la posición
73412 con 16 comparaciones antes de ordenar. Después de ordenar por
PM2.5 devolvió -1, aunque la lectura seguía existiendo: la búsqueda
lineal la encontró en la posición 87705.

**Alternativas:**
- Copia.
- Restaurar orden.
- Índices separados.

**Decisión:**
Ordenar por PM2.5 sobre una copia del arreglo y conservar el original
en orden cronológico.

**Justificación:**
La búsqueda binaria necesita el arreglo ordenado por timestamp. Con
100.000 lecturas hizo 16 comparaciones, frente a 100.000 de la lineal.
Ordenar una copia protege esa ventaja y evita respuestas incorrectas
sin aviso.

**Consecuencia:**
Se usa memoria adicional para la copia. El ranking se calcula sobre la
copia, y las consultas por timestamp siguen usando el original.