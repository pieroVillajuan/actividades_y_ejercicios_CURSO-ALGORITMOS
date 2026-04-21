
# Resolución de Ejercicios: Semana 2

## Ejercicio 1. Tiempo promedio vs. tiempo amortizado

### 1. Diferencia entre tiempo promedio y amortizado

- **Tiempo Promedio (Average Case):** Se basa en una esperanza probabilística. Se asume una distribución de entradas (por ejemplo, que todos los números tienen la misma probabilidad de ser buscados) y se calcula el tiempo medio.
    
- **Tiempo Amortizado:** Es una garantía sobre el **peor caso de una secuencia**. No depende de probabilidades. Asegura que, aunque una operación aislada sea muy lenta, la suma de los costos de $N$ operaciones dividida entre $N$ será pequeña.
    

### 2. Secuencia concreta (DengVector)

Supongamos un `DengVector` con `capacity = 4` y `size = 4`.

1. `push_back(10)` -> **Costo alto ($O(n)$):** Se dispara `expand()`, se crea un arreglo de tamaño 8 y se copian los 4 elementos previos + el nuevo.
    
2. `push_back(20)` -> **Costo bajo ($O(1)$):** Hay espacio libre.
    
3. `push_back(30)` -> **Costo bajo ($O(1)$):** Hay espacio libre.
    

### 3. ¿Por qué no contradice el O(1) amortizado?

Porque para que ocurra una operación de costo $n$ (expandir), el vector tuvo que haber realizado previamente muchas operaciones de costo 1 que "ahorraron" tiempo. El costo total de las copias en las expansiones forma una serie geométrica que suma aproximadamente $2n$. Al dividir $2n / n$, el resultado es una constante.

### 4. Papel de la duplicación

La duplicación garantiza que el intervalo entre expansiones crezca exponencialmente. Esto permite que el costo de la mudanza de datos se distribuya entre una cantidad de operaciones cada vez mayor, manteniendo el promedio constante.

### 5. Comparación con crecimiento por cantidad fija

Si crecemos por una constante $k$ (ej. sumar 10 espacios), después de $k$ inserciones siempre habrá un costo $O(n)$. Esto resulta en un costo amortizado de $O(n)$, lo cual es ineficiente para grandes volúmenes de datos.

---

## Ejercicio 2. Copia profunda y seguridad de `copyFrom()`

### 1. Revisión de `DengVector.h`

La función `copyFrom` utiliza el operador `new` para reservar un bloque de memoria totalmente nuevo:


``` C++
_elem = new T[_capacity]; // Reserva física independiente
```

### 2. Independencia física

Si el arreglo destino no fuera independiente, ambos vectores compartirían el mismo puntero. Si el Vector A modifica el elemento en la posición 0, el Vector B vería el cambio sin quererlo (**aliasing**). Además, al destruir uno, el otro se quedaría con un puntero a memoria liberada (**dangling pointer**).

### 3. Diseño de prueba (C++)


``` C++
ods::DengVector<int> v1;
v1.insert(0, 100);
ods::DengVector<int> v2 = v1; // Copia profunda
v1[0] = 500; // Modificamos el original
assert(v2[0] == 100); // La copia debe mantener el valor original
```

### 4. Ejecución lógica

Al ejecutar lo anterior, la aserción pasa con éxito porque `v2` tiene su propio arreglo `_elem`. Modificar la memoria de `v1` no afecta la dirección de memoria de `v2`.

### 5. Problema de compartir arreglo

Si compartieran el arreglo, ocurriría una **corrupción de memoria**. Al llamar al destructor del primer objeto, se liberaría la memoria; cuando el segundo objeto intente liberar la misma memoria al morir, el programa fallará (Double Free Error).

---

## Ejercicio 3. Crecimiento por constante vs. duplicación

### 1. Variante experimental


``` C++
void expand_constante(int c) {
    if (_size < _capacity) return;
    _capacity += c; // Crecimiento constante
    T* viejo = _elem;
    _elem = new T[_capacity];
    for(int i=0; i<_size; i++) _elem[i] = viejo[i];
    delete[] viejo;
}
```

### 2. Interfaz pública

La interfaz se mantiene idéntica (`insert`, `remove`, `get`), ya que el cambio es puramente interno (**encapsulamiento**).

### 3. Comparación conceptual

La versión original prioriza el **tiempo** (menos expansiones). La versión constante prioriza el **espacio** (no reserva tanta memoria extra), pero castiga severamente el rendimiento.

### 4. Empeoramiento del costo amortizado

Al crecer por constante, para llegar a $N$ elementos realizamos $N/c$ expansiones. El costo total es $c + 2c + 3c + ... + N$, lo cual es una serie aritmética de orden $O(n^2)$. El costo amortizado es $O(n^2) / n = O(n)$, perdiendo la eficiencia del vector.

### 5. Ejemplo evidente

Si $c=2$ e insertamos 1000 elementos, el vector constante copiará datos 500 veces. El vector original (duplicación) solo copiará datos unas 10 veces ($2^{10} = 1024$).

---

## Ejercicio 4. `shrink()` y reducción amortizada

### 1. Revisión de reducción

En `DengVector`, la reducción ocurre cuando el tamaño es significativamente menor que la capacidad. En `ArrayStack` de Morin, se suele redimensionar cuando `size < capacity / 4`.

### 2. Condición razonable

La reducción debe ocurrir solo cuando el **factor de carga** sea muy bajo (típicamente 25% o menos).

### 3. ¿Por qué no reducir tras cada eliminación?

Si reducimos justo al 50% y el usuario inserta un elemento inmediatamente, dispararíamos un `expand()`. Si luego borra, un `shrink()`. Esto se llama **histéresis** o "oscilación": el sistema pasaría todo el tiempo copiando datos de un lado a otro por un solo cambio.

### 4. Justificación del O(1) amortizado

Al reducir a la mitad solo cuando estamos al 25% de capacidad, garantizamos que después de la reducción el vector esté al 50% de su nueva capacidad. Esto deja un "margen de maniobra" de muchas eliminaciones o inserciones antes de que ocurra otro redimensionamiento.

### 5. Expand vs. Shrink

- `expand()`: Protege contra el **Overflow** (falta de espacio).
    
- `shrink()`: Protege contra el **Underflow espacial** (desperdicio de memoria).
    
    Ambas mantienen el equilibrio entre ser rápidos y no ser "glotones" de RAM.
    

---

## Ejercicio 5. Búsqueda secuencial en `DengVector`

### 1. Implementación de `find`

Busca desde `hi-1` hacia abajo hasta `lo`.


``` C++
while ((lo < hi--) && !(e == _elem[hi])) { }
return hi;
```

### 2. Complejidad

- **Mejor caso O(1):** El elemento buscado es el último del vector. El bucle termina en la primera comparación.
    
- **Peor caso O(n):** El elemento no existe o está al puro inicio. Debemos revisar todos los elementos.
    

### 3. Múltiples coincidencias

La implementación busca de atrás hacia adelante (`hi--`). Por lo tanto, si hay duplicados, devuelve el **índice de la ocurrencia de mayor rango** (la más cercana al final).

### 4. Pruebas de caso

- **Al final:** `v = [1, 2, 3]`, `find(3)` -> retorna 2 (Rápido).
    
- **Ausente:** `v = [1, 2, 3]`, `find(5)` -> retorna -1 (Lento).
    
- **Repeticiones:** `v = [7, 8, 7]`, `find(7)` -> retorna 2 (ignora el del índice 0).
    

### 5. Sensibilidad a la entrada

Es sensible porque el tiempo no depende solo de cuántos elementos hay ($n$), sino de **qué valores** hay y **en qué orden** están. Dos vectores del mismo tamaño pueden tardar tiempos muy distintos en encontrar un elemento dependiendo de su posición.

---

## Tabla de Conceptos (Método Feynman)

| **Concepto Técnico** | **Explicación Sencilla**                                                          | **Importancia**                                            |
| -------------------- | --------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| **Aliasing**         | Dos personas usando el mismo cepillo de dientes por error.                        | Causa errores de datos corruptos.                          |
| **Factor de Carga**  | Qué tan lleno está el autobús en relación a sus asientos.                         | Decide cuándo necesitamos un vehículo más grande.          |
| **Copia Profunda**   | Sacar una fotocopia de un examen para que cada alumno raye la suya.               | Mantiene la independencia de la información.               |
| **Histéresis**       | No cambiar de opinión (o de capacidad) ante el mínimo cambio para evitar el caos. | Previene el desperdicio de tiempo en procesos repetitivos. |

---

## Ejercicio 6. Inserción y desplazamientos en memoria contigua

### 1. Revisión de código

En `DengVector::insert(r, e)` y `ArrayStack::add(i, x)`, el flujo es idéntico: primero se asegura el espacio con `expand()` o `resize()`, y luego se ejecuta un bucle de desplazamiento.

### 2. Obligación de desplazar el sufijo

La **memoria contigua** exige que no haya "huecos" entre elementos. Si insertamos en la posición $r$, el elemento que estaba en $r$ y todos los que le siguen deben moverse una posición a la derecha para abrir espacio físico. Si no desplazáramos, sobrescribiríamos el dato existente en $r$, rompiendo la integridad de la lista.

### 3. Recorrido de atrás hacia adelante

El desplazamiento debe empezar por el último elemento (`_size`).

- **Razón:** Si empezamos desde $r$ moviendo el elemento a $r+1$, sobrescribiríamos el valor original de $r+1$ antes de haberlo movido a $r+2$.
    
- **Mantenimiento:** Al mover de atrás hacia adelante, siempre movemos un dato hacia una celda que ya ha sido "vaciada" (copiada) o que es nueva.
    

### 4. Costo promedio O(n)

Si la inserción es uniforme, la posición $r$ puede ser cualquier índice desde $0$ hasta $n$.

- En promedio, insertaremos en el medio ($n/2$).
    
- El número de desplazamientos es $n - r$.
    
- El promedio de desplazamientos es $\frac{1}{n+1} \sum_{r=0}^{n} (n-r) \approx \frac{n}{2}$.
    
    En notación asintótica, omitimos la constante $1/2$, resultando en $O(n)$.
    

### 5. Comparación de casos extremos

- **Al final ($r = n$):** Costo $O(1)$ amortizado. No hay sufijo que desplazar. Es la operación más eficiente.
    
- **Al inicio ($r = 0$):** Costo $O(n)$. Obliga a mover **absolutamente todos** los elementos actuales. Es el peor escenario.
    

---

## Ejercicio 7. Eliminación por intervalo y orden correcto

### 1. Revisión de `remove(lo, hi)`

En `DengVector`, esta función elimina un rango. La lógica clave es `_size -= (hi - lo)`.

### 2. Ineficiencia de llamadas repetidas

Si llamamos a `remove(r)` $k$ veces (donde $k = hi - lo$):

- Cada llamada individual desplazaría el sufijo completo hacia la izquierda.
    
- Si el intervalo es grande, estaríamos moviendo los mismos elementos del final muchas veces innecesariamente. El costo sería $O(k \cdot n)$.
    

### 3. Estrategia de compactación (Bloque)

La estrategia correcta es mover el sufijo que empieza en `hi` directamente a la posición `lo`.


``` C++
// Pseudocódigo de la estrategia correcta
while (hi < _size) 
    _elem[lo++] = _elem[hi++];
```

Esto realiza **un solo desplazamiento** por cada elemento del sufijo, sin importar qué tan grande sea el hueco eliminado.

### 4. Error por orden incorrecto (Compactación)

Si intentáramos compactar de atrás hacia adelante para una eliminación (al revés que en la inserción):

- **Ejemplo:** `[A, B, C, D, E]`, eliminar `[1, 3)` (B y C).
    
- Queremos que D pase a la posición 1.
    
- Si empezamos por el final e intentamos mover E a una posición calculada erróneamente, podríamos dejar huecos o sobrescribir datos que aún no se han movido hacia la izquierda. En eliminación, el destino siempre está a la "izquierda" del origen, por lo que el recorrido **debe ser de adelante hacia atrás**.
    

### 5. Comparación de costos

- **Elemento por elemento:** $O(k \cdot n)$.
    
- **En bloque:** $O(n - hi)$. Es decir, solo depende de cuántos elementos sobreviven al final, no de cuántos eliminas.
    

---

## Ejercicio 8. Recorrido uniforme con `traverse()`

### 1. Revisión de `traverse()`

Utiliza un puntero a función o un objeto función para aplicar una tarea a cada `_elem[i]`.

### 2 y 3. Implementación de operaciones


``` C++
// 2. Operación decrease
void decrease(int &n) { n -= 1; }

// 3. Operación double (usando objeto función para variedad)
struct DoubleOp {
    void operator()(int &n) { n *= 2; }
};
```

### 4. Ejemplo de prueba

- **Estado Inicial:** `[10, 20, 30]`
    
- **Después de `v.traverse(decrease)`:** `[9, 19, 29]`
    
- **Después de `v.traverse(DoubleOp())`:** `[18, 38, 58]`
    

### 5. Separación de incumbencias

`traverse()` es una excelente interfaz porque el **recorrido** (la lógica de cómo visitar cada celda en memoria contigua) se escribe una sola vez. El programador que usa el vector solo se preocupa por la **operación** (la lógica de negocio). Esto cumple con el principio de responsabilidad única.

---

## Ejercicio 9. Inserción múltiple eficiente `addAll`

### 1. Base de diseño

Usamos `DengVector` como base. Imaginamos que queremos insertar un `std::vector` de tamaño $m$ en la posición $i$.

### 2. Ineficiencia de la solución ingenua

Si llamamos a `add(i, x)` $m$ veces:

1. Podríamos disparar múltiples `expand()` (redimensionamientos) costosos.
    
2. Desplazaríamos el mismo sufijo $m$ veces.
    
    Costo: $O(m \cdot n)$.
    

### 3. Versión eficiente (Paso a paso)

1. **Calcular espacio:** Verificar si `_size + m > _capacity`.
    
2. **Redimensionar una vez:** Si hace falta, crear un arreglo que quepa todo de un solo golpe.
    
3. **Desplazamiento grande:** Mover el sufijo de $i$ a $n$ hacia la posición $i+m$ una sola vez.
    
4. **Copia de bloque:** Copiar los $m$ elementos nuevos en el hueco creado.
    

### 4. Implementación sencilla


``` C++
void addAll(int i, const std::vector<T>& c) {
    int m = c.size();
    while (_size + m > _capacity) expand(); // Un solo ciclo de expansión real
    
    // Desplazar sufijo una sola vez
    for (int j = _size - 1; j >= i; --j)
        _elem[j + m] = _elem[j];
        
    // Insertar el bloque nuevo
    for (int j = 0; j < m; ++j)
        _elem[i + j] = c[j];
        
    _size += m;
}
```

### 5. Comparación conceptual

- **Ingenua:** Muchos redimensionamientos y desplazamientos redundantes ($O(m \cdot n)$).
    
- **Mejorada:** Máximo un redimensionamiento y un desplazamiento por cada elemento del sufijo ($O(n + m)$). Es significativamente más rápida para inserciones masivas.
    

---

## Tabla de Conceptos: Método Feynman

|**Concepto Técnico**|**Explicación Simple**|**Utilidad Clave**|
|---|---|---|
|**Sufijo**|Los elementos que están después de la posición donde quieres trabajar.|Identifica qué datos se verán afectados por un movimiento.|
|**Compactación**|Empujar los libros hacia un lado para que no queden huecos en la repisa.|Mantiene la contigüidad necesaria para el acceso rápido.|
|**Iterador / Functor**|Un cartero que sabe qué casa visitar, pero la carta la escribes tú.|Separa el "cómo llegar" del "qué hacer".|
|**Invariante de Contigüidad**|Regla: "No puede haber un asiento vacío entre dos personas sentadas".|Es lo que permite que el acceso por índice sea $O(1)$.|