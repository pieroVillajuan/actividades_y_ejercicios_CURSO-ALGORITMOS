# Estructuras de Datos Lineales - Semana 2


## Bloque 1 - Núcleo conceptual de la semana
### Análisis Profundo de Estructuras de Datos Lineales

#### 1. Memoria Contigua
**Definición simple:** Es cuando los datos se guardan uno al lado del otro en la memoria de la computadora, sin espacios vacíos entre ellos.

**Explicación con ejemplo:** Imagina que vas al cine con amigos y tienen que sentarse todos juntos, en la misma fila, sin asientos vacíos en medio. Así funciona la memoria contigua: todos los elementos "viven" pegaditos.

**¿Para qué sirve?** La computadora sabe exactamente dónde está cada dato sin tener que buscar por todas partes.

**Importancia:** Como los datos están juntos, el procesador los lee muy rápido (esto se llama "localidad de referencia").

**Analogía:** Es como una calle con casas numeradas 101, 102, 103... Si sabes dónde está la 101 y cuánto mide cada terreno, puedes llegar a la 105 caminando en línea recta sin perderte.

---

#### 2. Acceso Directo y Complejidad $O(1)$
**Definición simple:** Acceder a cualquier posición de un arreglo toma siempre el mismo tiempo, sin importar qué tan grande sea la lista. Esto se escribe como $O(1)$ (tiempo constante).

**¿Por qué pasa esto?** Porque la computadora usa una fórmula matemática simple para calcular dónde está el elemento: 
$$Dirección(i) = Base + (i \times tamaño\_del\_tipo)$$

**Explicación con ejemplo:** No importa si tu lista tiene 10 elementos o 10 millones; llegar al elemento en la posición 5,500 toma el mismo tiempo que llegar al primero.

**Importancia:** Evita tener que revisar uno por uno desde el inicio (como pasaría en una lista enlazada).

**Analogía:** Es como un edificio con ascensor inteligente. Si conoces el número del piso (índice), presionas el botón y llegas directo. No tienes que abrir puerta por puerta.

---

#### 3. Diferencia entre Tamaño (Size) y Capacidad (Capacity)
| Concepto | Definición simple | Analogía |
|----------|------------------|----------|
| **Size (tamaño)** | Cuántos elementos hay actualmente guardados | Cuántos pasajeros hay en el autobús |
| **Capacity (capacidad)** | Cuántos elementos caben en total en la memoria reservada | Cuántos asientos tiene el autobús |

**Regla importante:** Siempre se cumple que `size ≤ capacity`.

**¿Por qué importa esta diferencia?** Porque nos permite agregar elementos nuevos sin tener que buscar más memoria cada vez.

**Analogía del vaso:** La capacidad es cuánto líquido puede contener el vaso; el tamaño es cuánta agua tiene ahora mismo. No necesitas vaso nuevo por cada gota que agregas.

---

#### 4. El "Porqué" del Nuevo Bloque en Resize
**Problema:** Cuando un arreglo se llena y necesita crecer, no siempre hay espacio libre justo después en la memoria.

**Solución (paso a paso):**
1. Reservar un bloque de memoria nuevo y más grande
2. Copiar todos los elementos del bloque viejo al nuevo
3. Liberar (borrar) el bloque antiguo

**Explicación con ejemplo:** Imagina que estás en un restaurante con una mesa para 4 y llega un quinto amigo. No puedes "estirar" la mesa porque hay otros clientes al lado. Entonces, los meseros buscan una mesa más grande en otro lugar, llevan tus platos allí, y limpian la mesa vieja.

**Analogía:** Es como mudarse de casa. Si tu familia crece, no puedes estirar las paredes (hay vecinos). Tienes que comprar una casa más grande, llevar tus cosas y dejar la vieja.

---

#### 5. Costo Amortizado $O(1)$ por Duplicación
**Concepto clave:** Aunque expandir el arreglo cuesta mucho ($O(n)$), si duplicamos la capacidad cada vez, ese costo se "reparte" entre muchas inserciones futuras.

**Fórmula intuitiva:** 
$$Costo\_promedio = \frac{Costo\_total\_de\_todas\_las\_inserciones}{Número\_de\_inserciones} = O(1)$$

**Explicación con ejemplo:** Si cada vez que invitas un amigo a tu casa tuvieras que mudarte, sería un caos. Pero si cada mudanza compras una casa con el doble de espacio, te mudarás muy pocas veces. La mudanza es cara, pero como ocurre tan raramente, el esfuerzo "por día" es mínimo.

**Analogía:** Comprar comida al por mayor. Ir al supermercado toma tiempo (costo alto), pero si vas una vez al mes y compras para 30 días, el "costo de transporte" por cada comida es casi cero.

---

#### 6. Comparación: ArrayStack vs. DengVector

| Característica | ArrayStack (Morin) | DengVector (Deng) |
|---------------|-------------------|-------------------|
| **Enfoque** | Implementar la interfaz `List` (lista) | Abstracción basada en "Rango" (posición) |
| **Didáctica** | Se centra en comportamiento de Pila/Lista y redimensionamiento | Se centra en ciclo de vida del objeto y copia profunda |
| **Operaciones** | Usa `get(i)`, `set(i)`, `add(i)`, `remove(i)` | Usa sobrecarga de operadores `[]`, `insert`, `remove`, `find` |
| **Reducción** | Reduce cuando `capacity ≥ 3 × size` | Reduce cuando `size × 4 ≤ capacity` |

**Lo que comparten:** Ambos usan un arreglo de respaldo y duplican la capacidad al crecer.

**Lo que cambia:** `DengVector` busca ser un contenedor robusto (estilo C++ STL), mientras que `ArrayStack` es más académico para demostrar optimizaciones.

---

#### 7. Optimización en FastArrayStack
**Idea principal:** Usa funciones de bajo nivel como `std::copy` y `std::copy_backward` que mueven bloques de memoria de forma más eficiente.

**Diferencia en código:**
```cpp
// ArrayStack (más lento: elemento por elemento)
for (int j = n; j > i; j--) a[j] = a[j-1];

// FastArrayStack (más rápido: bloque completo)
std::copy_backward(a+i, a+n, a+n+1);
```

**¿Por qué es más rápido?** Porque estas funciones usan instrucciones especiales del procesador (SIMD) que mueven varios datos a la vez.

**Analogía:** Es la diferencia entre llenar una piscina con un balde (bucle `for`) vs. usar una manguera de bomberos (copia optimizada). El resultado es el mismo, pero el tiempo real es mucho menor.

---

#### 8. Idea Espacial de RootishArrayStack
**Concepto central:** En lugar de un solo bloque grande, usa varios bloques pequeños que crecen en tamaño: el bloque $b$ tiene capacidad $b+1$ (bloque 0 → 1 espacio, bloque 1 → 2 espacios, bloque 2 → 3 espacios, etc.).

**Ventaja clave:** Nunca hay que mover todos los elementos cuando crecemos; solo añadimos un bloque nuevo al final.

**Fórmula de mapeo:** Para saber en qué bloque está el índice $i$, usamos:
$$i = \frac{b(b+1)}{2} \quad \text{(suma de progresión aritmética)}$$

**Analogía:** Es como una biblioteca que añade estantes. En lugar de comprar un mueble nuevo y mover todos los libros cada vez, simplemente compras un estante adicional y lo pones al lado.

---

#### 9. Bloques de Tamaños 1, 2, 3... en RootishArrayStack
**¿Por qué esta secuencia?** Para minimizar el espacio desperdiciado (waste).

**Comparación de desperdicio:**

| Estructura | Desperdicio máximo |
|------------|-------------------|
| Arreglo dinámico tradicional | $O(n)$ (puede desperdiciar casi tanto como usa) |
| RootishArrayStack | $O(\sqrt{n})$ (mucho menos desperdicio) |

**Ejemplo numérico:** Si tienes 1,000,000 de datos:
- Arreglo normal: podría desperdiciar espacio para ~1,000,000 más
- Rootish: solo desperdicia espacio para ~1,414 datos

**Analogía:** Como comprar azulejos. Si compras cajas de 100 y solo necesitas 101, te sobran 99. Pero si las cajas vienen en tamaños 1, 2, 3, 4... compras casi exactamente lo que necesitas.

---

#### 10. Representación, Costo y Desperdicio (Trade-offs)
**Compromiso fundamental en diseño:**

| Estrategia | Ventaja | Desventaja |
|-----------|---------|------------|
| **Memoria contigua** | Acceso rápido $O(1)$ | Inserción lenta $O(n)$ por desplazamientos |
| **Sobre-aprovisionamiento (duplicación)** | Costo amortizado $O(1)$ | Desperdicio espacial temporal |
| **RootishArrayStack** | Desperdicio reducido $O(\sqrt{n})$ | Cálculo de índice más complejo (usa `sqrt`) |

**Mensaje clave:** En computación, nada es gratis. Si quieres velocidad extrema para leer, debes aceptar gastar memoria extra o perder tiempo moviendo cosas al insertar.

**Analogía del equipaje:**
- Maleta rígida (Arreglo): Todo ordenado y rápido de encontrar, pero si se llena necesitas maleta nueva.
- Bolsas de tela separadas (Rootish): Puedes añadir bolsas pequeñas poco a poco, pero encontrar algo requiere saber en qué bolsa buscar.

---

### Tabla de Conceptos Clave (para repasar)

| Término Técnico | Explicación Sencilla | Utilidad Clave |
|----------------|---------------------|----------------|
| Memoria Contigua | Como una fila de soldados agarrados de la mano | Permite encontrar a cualquiera en la fila instantáneamente |
| Acceso $O(1)$ | Saber dónde está el juguete sin revolver toda la caja | Velocidad de lectura inigualable |
| Análisis Amortizado | Pagar suscripción anual: duele una vez, pero el resto del año parece gratis | Justifica por qué el `resize` no arruina la eficiencia |
| Aritmética Modular | Como un reloj: después de las 12, vuelve a empezar en la 1 | Permite que las colas sean circulares sin desperdiciar espacio |
| Invariante | Regla que nunca se puede romper (ej: "siempre size ≤ capacity") | Garantiza que el programa no falle ni se cierre solo |
| Desplazamiento | Cuando alguien se sienta en medio de una fila y todos se mueven | Es el "precio" que pagamos por mantener el orden en un arreglo |

---

## Bloque 2 - Demostración y trazado guiado
### Análisis Avanzado: Demos y Evidencia Empírica

#### I. Matriz de Análisis de Demos (Semana 2)

| Archivo | Observable importante | Idea estructural | Argumento de costo/espacio |
|---------|----------------------|-----------------|---------------------------|
| `demo_array_basico.cpp` | Transferencia de contenido de `a` hacia `b` | Memoria Contigua y transferencia de propiedad | Acceso directo $O(1)$. Espacio estático definido al inicio |
| `demo_arraystack_explicado.cpp` | Visualización de desplazamientos en `add(1, 15)` | Lista sobre arreglo con gestión de extremos | Costo $O(n-i)$ por desplazamientos de elementos |
| `demo_fastarraystack.cpp` | Ejecución acelerada de `add` y `resize` | Optimización mediante copia de bloques | Mantiene $O(n)$, pero reduce el factor constante $c$ |
| `demo_rootisharraystack_explicado.cpp` | Mapeo: índice 5 → bloque 2, offset 2 | Estructura en bloques triangulares (1, 2, 3...) | Desperdicio espacial reducido a $O(\sqrt{n})$ |
| `demo_deng_vector.cpp` | Incremento de `capacity` de 3 a 8 y luego a 16 | Crecimiento geométrico (duplicación) | Costo amortizado $O(1)$ para inserciones |
| `demo_stl_vector_contraste.cpp` | Evolución de `capacity` en C++ estándar | Interfaz ADT robusta y encapsulamiento | Estrategia de realocación para minimizar `reallocs` |

---

#### II. Respuestas Detalladas y Pedagógicas

##### 1. Arreglo, Longitud y Asignación (`demo_array_basico.cpp`)
**Enunciado simple:** El arreglo tiene tamaño fijo. La operación `b = a` no copia el contenido, sino que `b` apunta al mismo bloque de memoria que `a` (transferencia de propiedad).

**Explicación:** Un arreglo es como una caja con compartimentos fijos. Si compras una caja de 5 espacios (`length = 5`), no puedes meter 6 cosas. Cuando hacemos `b = a`, le damos a `b` la llave de la caja de `a`.

**Importancia:** Entender quién es el "dueño" de la memoria evita que dos partes del programa intenten borrar la misma caja (error de memoria).

**Analogía:** Es como pasar un contrato de alquiler: ahora la otra persona es dueña del espacio y tú ya no tienes dónde vivir. El tamaño no cambia, solo quién tiene las llaves.

---

##### 2. Costo por Desplazamientos (`demo_arraystack_explicado.cpp`)
**Enunciado simple:** Insertar en posición intermedia (`add(1, 15)`) requiere mover todos los elementos que están después para hacer espacio. El costo es proporcional a $n - i$.

**Explicación:** Si ya tenías datos en posiciones 1, 2 y 3, y quieres meter algo en la 1, ¡tienes que empujar a todos hacia la derecha! (desplazamiento).

**Función:** Abrir un hueco vacío en la memoria para que el nuevo dato tenga donde sentarse.

**Analogía:** Imagina una fila para entrar al cine. Si alguien intenta "colarse" en el segundo lugar, todas las personas detrás tienen que dar un paso hacia atrás. Si te pones al final, nadie se mueve.

---

##### 3. Implementación vs. Complejidad Asintótica (`demo_fastarraystack.cpp`)
**Enunciado simple:** `FastArrayStack` usa `std::copy_backward` para mover bloques completos. La complejidad sigue siendo $O(n)$, pero el tiempo real es menor porque el procesador mueve varios datos a la vez.

**Explicación:** En lugar de un bucle `for` que mueve elementos uno por uno, usamos una función "relámpago" que mueve el bloque entero.

**Analogía:** Es la diferencia entre mover 100 ladrillos uno por uno con la mano vs. usar una cinta transportadora. En ambos casos moviste 100 ladrillos, pero la cinta es mucho más eficiente.

---

##### 4. Mapeo de Índice en RootishArrayStack (`demo_rootisharraystack_explicado.cpp`)
**Enunciado simple:** Para encontrar en qué bloque está el índice $i$, resolvemos: $\frac{b(b+1)}{2} \leq i$. La fórmula es: $b = \lceil \frac{-3 + \sqrt{9 + 8i}}{2} \rceil$.

**Explicación:** En un arreglo normal, el elemento 500 está en la posición 500. Pero en Rootish, el elemento 500 está escondido en algún bloque. La función `i2b(i)` es un "GPS matemático" que te dice instantáneamente en qué bloque está.

**Analogía:** Es como buscar una página en un libro donde cada capítulo es más largo que el anterior. Usas una fórmula mágica que te dice: "Esa página está en el Capítulo 7".

---

##### 5. Crecimiento de Capacity (`demo_deng_vector.cpp`)
**Observable clave:** La capacidad salta: 3 → 8 → 16... (duplicación geométrica).

**Explicación:** Al duplicar la capacidad, nos aseguramos de que no tendremos que volver a "mudarnos" (hacer `expand`) por mucho tiempo.

**Analogía:** Es como comprar comida para la semana. Podrías ir a la tienda cada vez que tienes hambre (muy ineficiente), o puedes comprar el doble de lo que necesitas hoy para no volver en mucho tiempo.

---

##### 6. Similitud Conceptual con STL (`demo_stl_vector_contraste.cpp`)
**Enunciado simple:** Tanto `std::vector` como `DengVector` usan:
- Arreglo de respaldo contiguo
- Crecimiento exponencial al llenarse
- Acceso mediante operador `[]`

**Explicación:** Son como dos marcas distintas de maletas expandibles. Por fuera se ven y usan igual, y ambas tienen un cierre que permite que la maleta se haga más grande si ya no cabe nada más.

---

##### 7. Amortización vs. Uso de Espacio
| Concepto | Mejor demo para defenderlo | ¿Por qué? |
|----------|---------------------------|-----------|
| **Amortización** (ahorro de tiempo) | `demo_deng_vector.cpp` | Muestra cómo la capacidad crece exponencialmente para diluir el costo de la copia |
| **Uso de Espacio** (no desperdiciar) | `demo_rootisharraystack_explicado.cpp` | Su estructura de bloques crecientes limita el desperdicio a $O(\sqrt{n})$ |

**Analogía adicional:**
- Amortización: Como comprar un pase anual de transporte. Pagas mucho al inicio, pero cada viaje individual después te sale "gratis".
- Uso de Espacio: Como usar tupperwares de diferentes tamaños para las sobras, en lugar de poner un poquito de arroz en una olla gigante.

---

### Tabla Técnica (para repasar)

| Concepto Técnico | Explicación "Para un amigo" | ¿Por qué es clave? |
|-----------------|----------------------------|-------------------|
| Costo Amortizado | El promedio del costo de muchas operaciones a lo largo del tiempo | Permite usar estructuras que a veces son lentas pero usualmente rápidas |
| Arreglo de Respaldo | El lugar real (el "garage") donde la computadora guarda los datos | Es la base física de toda la estructura contigua |
| Invariante | Una regla que siempre es verdad (ej: "el tamaño nunca supera la capacidad") | Si la regla se rompe, el programa "explota" (falla) |
| Factor de Carga | Qué tan lleno está el recipiente (`size / capacity`) | Nos dice cuándo es momento de crecer o encoger la estructura |
| Underflow Espacial | Cuando tienes un estadio gigante para solo 2 personas | Es el principal enemigo de la eficiencia de memoria |
| Aritmética Modular | La lógica del reloj para que el final conecte con el inicio | Permite crear colas circulares sin mover datos de lugar |

---

## Bloque 3 - Pruebas públicas, stress y correctitud
### Análisis de Verificación y Validación

#### I. Análisis de Pruebas Públicas y Estrés

##### 1. Operaciones en `ArrayStack`
**Qué valida la prueba pública:** Que la estructura sepa hacer lo mínimo necesario:
- `add`: Guardar un dato
- `get`: Devolvértelo cuando lo pides
- `remove`: Borrarlo correctamente

**Explicación:** La prueba pública es como un examen de entrada básico. Si estas operaciones fallan, la estructura ni siquiera "arranca".

**Analogía:** Es como probar un bolígrafo nuevo en la tienda. Solo rayas un poco el papel para ver si sale tinta. No estás probando si durará 10 años, solo si escribe ahora.

---

##### 2. Operaciones en `FastArrayStack`
**Qué valida:** Que, por ser más rápido, no "rompimos" nada. Como usa copia de bloques, la prueba verifica que el resultado final sea el mismo que si lo hubiéramos movido elemento por elemento.

**Analogía:** Es como probar un tren de alta velocidad en una vía normal. Quieres verificar que, aunque vaya más rápido, se detenga en las mismas estaciones que el tren lento.

---

##### 3. Operaciones en `RootishArrayStack`
**Qué valida:** Que las operaciones `add(i, x)`, `get(i)` y `remove(i)` funcionen de forma transparente, ocultando la complejidad del mapeo de índices a bloques.

**Explicación:** Tú, como programador, no te das cuenta del lío interno de muchos bloques. Tú solo pides el elemento 5 y él te lo da, sin importar en qué bloque esté.

**Analogía:** Es como una biblioteca con estantes de distintos tamaños. A ti no te importa si el libro está en un estante pequeño o grande; tú solo quieres que el bibliotecario te traiga el libro que pediste.

---

##### 4. ¿Qué demuestra una prueba pública?
**Respuesta simple:** Demuestra que el código "funciona para lo que fue diseñado" en situaciones tranquilas. Confirma que:
- Los nombres de las funciones son correctos
- La lógica básica no tiene errores evidentes

**Analogía:** Es como el encendido de un auto. Si el motor arranca y las luces prenden, el auto pasó la prueba pública. Sabemos que el sistema básico está conectado correctamente.

---

##### 5. ¿Qué NO demuestra una prueba pública?
**Limitaciones importantes:**
- ❌ No garantiza rapidez (puede pasar la prueba y ser lento)
- ❌ No garantiza estabilidad con muchos datos (puede colapsar con 3 millones)
- ❌ No valida comportamiento en casos extremos o fugas de memoria

**Analogía:** Que un auto prenda en el garaje no demuestra que pueda cruzar el desierto a 100 km/h sin recalentarse. La prueba pública no mide resistencia ni rendimiento a largo plazo.

---

##### 6. Comportamiento en `resize_stress_week2.cpp`
**Propósito del archivo:** Estresar el ciclo de vida de la memoria, específicamente el mecanismo de redimensionamiento (`resizing`).

**Qué hace:** Realiza cientos de inserciones y eliminaciones para forzar a la estructura a:
- Expandir (`expand()`) cuando se llena
- Contraer (`shrink()`) cuando se vacía

**Analogía:** Es como meter a 50 personas en un ascensor diseñado para 10, y luego sacarlas a todas de golpe mientras el ascensor sube y baja. Estamos viendo si los cables aguantan y si el motor no se quema.

---

##### 7. ¿Por qué las pruebas no reemplazan la teoría?
**Respuesta clave:** Las pruebas son empíricas y finitas; los invariantes y la complejidad son propiedades estructurales y universales.

| Aspecto | Pruebas (empírico) | Teoría (invariantes) |
|---------|-------------------|---------------------|
| **Cobertura** | Solo casos específicos | Cualquier estado legal posible |
| **Garantía** | "Funcionó hoy" | "Siempre funcionará si se respetan las reglas" |
| **Escalabilidad** | No predice comportamiento con billones de datos | La complejidad $O(f(n))$ sí lo hace |

**Analogía:** Puedes cruzar la calle con los ojos cerrados y no morir (pasar la prueba), pero eso no significa que sea una estrategia segura. La teoría de invariantes es abrir los ojos y entender las leyes del tráfico para nunca ser atropellado.

---

### Tabla de Conceptos (para repasar)

| Término Técnico | Explicación para Principiantes | Utilidad Clave |
|----------------|-------------------------------|----------------|
| Prueba Unitaria (Unit Test) | Un pequeño examen para una sola pieza del código | Asegura que cada pieza individual funcione bien antes de armar todo |
| Prueba de Estrés | Llevar el código al límite para ver cuándo se rompe | Evita que el sistema colapse cuando hay muchos usuarios reales |
| Invariante de Estructura | Una regla que siempre debe ser verdad (ej: "el agua siempre está dentro del vaso") | Garantiza que los datos no se pierdan o se mezclen por error |
| Correctitud (Correctness) | Que el programa haga exactamente lo que se le pidió | Es el requisito mínimo para que un software sea útil |
| Capa de Abstracción | Un "muro" que separa cómo se usa algo de cómo está construido por dentro | Permite cambiar el código interno sin arruinarle la vida al usuario |
| Caso de Borde (Edge Case) | Situaciones raras (ej: intentar borrar algo de una lista que ya está vacía) | Evita que el programa se cierre inesperadamente por errores tontos |

---

## Bloque 4 - Vector como puente entre teoría y código
### Anatomía y Lógica del Vector: Análisis de la Implementación "Deng"

#### I. Respuestas Técnicas y Pedagógicas

##### 1. El Trío Fundamental: `_size`, `_capacity` y `_elem`
| Miembro | Qué es | Analogía del estacionamiento |
|---------|--------|----------------------------|
| `_elem` | Puntero al bloque de memoria contigua | El terreno pavimentado donde estacionas los autos |
| `_capacity` | Límite superior de almacenamiento físico reservado | Los espacios marcados en el suelo (ej: 10 cajones) |
| `_size` | Número de elementos lógicamente válidos | Los autos presentes ahora mismo (ej: 4 autos) |

**Regla de oro:**  $$ 0 \leq \text{\_size} \leq \text{\_capacity} $$

**Analogía de la caja de chocolates:** La caja tiene huecos para 12 piezas (capacidad). Si solo te quedan 5 chocolates (tamaño), los otros 7 huecos están vacíos pero listos para ser usados. El cartón de la caja es el soporte físico (`_elem`).

---

##### 2. Disparo de la Función `expand()`
**¿Cuándo se llama?** Obligatoriamente antes de insertar cuando `_size == _capacity` (el arreglo está lleno).

**Código clave:**
```cpp
void expand() {
    if (_size < _capacity) return; // Si aún hay espacio, no hagas nada.
    // ... de lo contrario, ¡crece! (reservar nuevo bloque, copiar, liberar viejo)
}
```

**Función:** Asegurar que siempre haya un "asiento vacío" disponible antes de que alguien llegue.

**Importancia:** Si intentas meter un dato cuando `_size` ya alcanzó a `_capacity`, invadirás memoria que no te pertenece → el programa colapsa (`segmentation fault`).

**Analogía:** Es como un vaso de agua. Debes cambiar el agua a una jarra más grande justo antes de que la última gota haga que el vaso se derrame.

---

##### 3. Necesidad de Desplazamiento en `insert(r, e)`
**¿Por qué hay que mover elementos?** Porque el vector garantiza acceso aleatorio mediante memoria contigua. Para insertar en posición $r < n$, debemos preservar el orden relativo de los elementos existentes.

**Costo:** $O(n-r)$ (lineal en el número de elementos a desplazar).

**Explicación:** Como todos los datos deben estar pegaditos, uno tras otro, no puedes simplemente "flotar" un dato en el medio. Si quieres meter un libro en medio de una repisa llena, tienes que empujar todos los libros de la derecha hacia el final para hacerle un hueco al nuevo.

**Analogía:** Imagina una fila de personas agarradas de los hombros. Si una persona nueva quiere entrar en la segunda posición, todas las personas desde la segunda hasta la última deben dar un paso hacia atrás para que la nueva pueda "engancharse" en la cadena sin romper la unión.

---

##### 4. Diferencia Conceptual entre `remove(r)` y `remove(lo, hi)`
| Operación | Qué hace | Costo | Eficiencia |
|-----------|----------|-------|------------|
| `remove(r)` | Elimina un solo elemento en posición $r$ | $O(n-r)$ | Mueve elementos una vez por cada eliminación |
| `remove(lo, hi)` | Elimina un rango de elementos $[lo, hi)$ | $O(n-lo)$ | Mueve elementos una sola vez para cerrar la brecha completa |

**Diferencia clave:** Si sacas a 5 personas una por una, haces que la gente de atrás se mueva 5 veces. Si los sacas a los 5 al mismo tiempo, los de atrás solo se mueven una vez para llenar el gran hueco.

**Analogía:** Es como limpiar un tren de carga. Puedes quitar un vagón a la vez (lento) o desenganchar una sección entera de 10 vagones de un solo golpe (eliminación de rango).

---

##### 5. Evidencia de Copia Profunda (Deep Copy)
**¿Dónde se ve?** En el constructor de copia y el operador de asignación que invocan a `copyFrom()`.

**Código clave:**
```cpp
_elem = new T[_capacity]; // Crea una NUEVA casa (nuevo bloque de memoria)
while (lo < hi) {
    _elem[_size++] = A[lo++]; // Copia los muebles uno por uno
}
```

**Diferencia crucial:**
- ❌ Copia superficial: Copia solo el puntero → dos objetos apuntan al mismo bloque (peligroso)
- ✅ Copia profunda: Reserva nuevo bloque y copia cada elemento → objetos independientes

**Analogía:** Es la diferencia entre compartir un documento de Google Docs (si yo borro algo, tú también lo pierdes) vs. mandarte una copia de un archivo de Word por correo (puedes borrar lo que quieras en tu archivo y el mío seguirá intacto).

---

##### 6. Valor Didáctico de `traverse()`
**Qué hace:** Recorre todos los elementos del vector y aplica una función (o "regla") a cada uno.

**Ventaja pedagógica:** Separa la lógica de "cómo recorrer la estructura" de la lógica de "qué hacer con cada dato".

**Explicación:** En lugar de escribir un `for` aburrido cada vez que quieres imprimir o sumar los datos, usas `traverse`. Tú solo le dices a la estructura: "Oye, aplica esta regla a todos tus hijos".

**Analogía:** Es como un guía de turistas en un museo. El guía sabe el camino (recorrido), pero tú decides si quieres tomar fotos, dibujar o simplemente mirar cada cuadro (función aplicada).

---

##### 7. Ventaja de Implementar un Vector Propio
**¿Por qué hacerlo si existe `std::vector`?** Porque te da una "visión de rayos X" sobre lo que sucede internamente:

| Lo que aprendes | Por qué importa |
|----------------|---------------|
| Gestión de recursos (RAII) | Entiendes cuándo se reserva y libera memoria |
| Manejo de memoria dinámica en el heap | Sabes cómo crecen las estructuras "infinitas" |
| Compromisos tiempo/espacio | Comprendes por qué a veces el programa se vuelve lento |
| Manejo de punteros | Es lo que separa programadores novatos de ingenieros de élite |

**Analogía:** Es la diferencia entre saber conducir un auto (usar `std::vector`) y ser un mecánico de Fórmula 1 que sabe cómo funciona cada pistón del motor (implementar tu propio vector).

---

### Tabla Maestra de Conceptos

| Concepto Técnico | Explicación "En Sencillo" | ¿Por qué es vital? |
|-----------------|--------------------------|-------------------|
| Heap (Montículo) | Un almacén gigante de memoria donde pides espacio "a medida" | Permite que tus listas crezcan infinitamente mientras haya RAM |
| Puntero (`_elem`) | Una flecha que apunta a la dirección de una casa | Sin la flecha, el programa olvida dónde guardó tus datos |
| Costo Lineal $O(n)$ | El tiempo de trabajo crece igual que la cantidad de datos | Te avisa que, si tienes billones de datos, la operación será lenta |
| Realocación | Moverse de un departamento pequeño a uno grande | Es el "mal necesario" para tener arreglos que crecen |
| Constructor de Copia | El "botón de clonar" un objeto | Evita que dos partes del programa se peleen por el mismo dato |
| Sobrecarga de Operador | Enseñarle a un objeto nuevo trucos viejos (como usar `[]`) | Hace que usar tu propia estructura sea tan fácil como un arreglo de C |

---

## Bloque 5 - RootishArrayStack: espacio y mapeo
### Análisis Avanzado: Eficiencia Espacial y Mapeo No Lineal

#### I. Respuestas Técnicas y Pedagógicas

##### 1. Distribución de Elementos entre Bloques
**Estructura:** El `RootishArrayStack` organiza los elementos en bloques donde:
- Bloque 0 → capacidad 1
- Bloque 1 → capacidad 2
- Bloque 2 → capacidad 3
- ...
- Bloque $b$ → capacidad $b+1$

**Ventaja principal:** Fragmentar la memoria para que el crecimiento sea gradual. No hay que "duplicar" un millón de espacios de golpe si solo necesitamos uno más.

**Analogía:** Es como una pirámide invertida de vasos. En la cima hay 1 vaso, en el siguiente nivel hay 2, luego 3... Para guardar 6 canicas, llenas el nivel 1, luego el 2 y finalmente el 3.

---

##### 2. Capacidad Total: La Serie Aritmética $r(r+1)/2$
**Fórmula:** La capacidad total con $r$ bloques es:
$$C = \sum_{i=1}^{r} i = \frac{r(r+1)}{2}$$

**Verificación con ejemplo:** Si tienes 4 bloques (tamaños 1, 2, 3, 4):
- Suma manual: $1+2+3+4 = 10$
- Fórmula: $\frac{4(4+1)}{2} = \frac{20}{2} = 10$ ✓

**Utilidad:** Nos permite saber cuántos elementos podemos guardar en total si conocemos cuántos bloques tenemos abiertos.

**Analogía:** Imagina que estás armando un triángulo con bolitas de billar. Si el triángulo tiene 5 filas de base, la cantidad total de bolitas siempre seguirá esa fórmula. Es la geometría de la eficiencia.

---

##### 3. El Problema que Resuelve `i2b(i)`
**Problema:** En un arreglo normal, el elemento 500 está en la posición 500. Pero en Rootish, el elemento 500 está escondido en algún bloque. ¿En cuál?

**Solución:** Resolver la ecuación cuadrática $\frac{b(b+1)}{2} \geq i$ para $b$:
$$b = \left\lceil \frac{-3 + \sqrt{9 + 8i}}{2} \right\rceil$$

**Función:** Es un "GPS matemático" que, dándole el número del elemento, te dice instantáneamente en qué bloque está (tiempo constante $O(1)$).

**Analogía:** Es como buscar una página en un libro donde cada capítulo es más largo que el anterior. Si buscas la página 100, no quieres ojear todo el libro; usas una fórmula mágica que te dice: "Esa página está en el Capítulo 7".

---

##### 4. Información de `locate(i)` (Mapeo de Índice)
**Qué devuelve:** Un par ordenado $(b, j)$ donde:
- $b$ = índice del bloque objetivo (¿en qué pasillo está mi dato?)
- $j$ = offset intra-bloque (¿en qué posición dentro de ese pasillo?)

**Cálculo del offset:**
$$j = i - \frac{b(b+1)}{2}$$

**Código ilustrativo:**
```cpp
int b = i2b(i);                    // ¿En qué bloque?
int j = i - b*(b+1)/2;            // ¿En qué posición dentro del bloque?
return blocks.get(b)[j];          // ¡Acceso directo!
```

**Analogía:** Es como una dirección de ciudad. No basta con decir "Calle 5". Necesitas decir "Calle 5 (bloque), Casa 2 (offset)". `locate` te da ambos datos para que llegues directo al objetivo.

---

##### 5. Ganancia de Espacio frente a `ArrayStack`
**Comparación de desperdicio:**

| Estructura | Desperdicio máximo | Ejemplo con 1,000,000 datos |
|-----------|-------------------|----------------------------|
| ArrayStack | $O(n)$ | Podría desperdiciar ~1,000,000 espacios |
| RootishArrayStack | $O(\sqrt{n})$ | Solo desperdicia ~1,414 espacios |

**¿Por qué Rootish desperdicia menos?** Porque si se llena el último bloque, solo añades un bloque un poquito más grande. Nunca sobran demasiados espacios.

**Analogía:** Es como comprar zapatos para un niño que crece rápido. El `ArrayStack` es comprarle hoy unos zapatos talla 20 y mañana unos talla 40 por si acaso. El `Rootish` es comprarle unos talla 21, luego 22... Siempre le quedan casi perfectos.

---

##### 6. Lo que se Conserva en la Interfaz
**Transparencia para el usuario:** A pesar de la complejidad interna, el usuario sigue usando los mismos métodos:
- `size()`, `get(i)`, `set(i, x)`, `add(i, x)`, `remove(i)`

**Encapsulamiento:** La complejidad del almacenamiento fragmentado queda oculta. No tienes que saber matemáticas raras para usarlo.

**Analogía:** Es como un automóvil híbrido. Por fuera tiene volante, pedales y palanca de cambios igual que uno de gasolina. No necesitas saber física cuántica sobre baterías para conducirlo; la complejidad está "bajo el capó".

---

##### 7. El Reto de la Defensa Oral: ¿Qué es más difícil?
**Desde la perspectiva de arquitectura de software:**

| Aspecto | Por qué es difícil de defender | Cómo explicarlo |
|---------|-------------------------------|----------------|
| **Mapeo `i2b(i)`** | Parece "mágico" usar `sqrt` para buscar en una lista | `sqrt` en CPUs modernas es muy rápido; el ahorro de memoria compensa esos microsegundos |
| **Análisis espacial $O(\sqrt{n})$** | Requiere justificación matemática (series de Gauss) bajo presión | Mostrar con ejemplos numéricos: 1M datos → solo ~1,414 espacios desperdiciados |

**Consejo para defensa:** Enfócate en el trade-off: "Rootish sacrifica un poquito de velocidad en el acceso (cálculo de bloque) para ahorrar muchísima memoria. En aplicaciones grandes, ese ahorro vale la pena".

---

### Tabla Maestra de Conceptos

| Concepto Técnico | Explicación para un "No-Programador" | Utilidad Clave |
|-----------------|-------------------------------------|----------------|
| Bloque Triangular | Cajas que se vuelven más grandes conforme las vas necesitando | Evita comprar un almacén gigante que vas a dejar medio vacío |
| Mapeo `i2b` | Una fórmula matemática que adivina dónde está tu dato sin buscarlo | Mantiene la velocidad de la estructura aunque esté partida en trozos |
| Desperdicio $O(\sqrt{n})$ | El "aire" que queda en el paquete es mínimo comparado con el producto | Ahorra muchísima memoria RAM en aplicaciones grandes |
| Punteros a Bloques | Una lista de direcciones que te dicen dónde están guardadas las cajas | Es el "índice" que mantiene unida a toda la estructura |
| Gasto Amortizado | El costo de limpiar la casa dividido entre todos los días que estuvo limpia | Explica por qué añadir elementos sigue siendo rápido en promedio |
| Encapsulamiento | Esconder el motor complejo bajo un capó simple y elegante | Permite que otros programadores usen tu código sin romperse la cabeza |

---

## Bloque 6 - Refuerzo de lectura
### Profundización en el Vector: Análisis de la Lectura 4 (Deng)

#### I. Análisis de Conceptos y Respuestas

##### 1. El papel del `operator[]` en la idea de vector
**Función principal:** Proporcionar "azúcar sintáctico" (código más legible) para el acceso por rango, manteniendo la encapsulación.

**Ventaja práctica:**
```cpp
v[i]        // ← Natural, intuitivo, como arreglo de C
v.get(i)    // ← Funcional, pero más verboso
```

**Explicación:** Imagina que has construido un robot muy complejo para organizar tus libros. Para pedirle un libro, podrías tener que escribir `robot.obtenerLibroEnLaPosicion(5)`. Eso es funcional, pero aburrido. El `operator[]` es como ponerle un teclado simple al robot donde solo presionas el número `[5]`.

**Analogía:** Es como el mando a distancia de una TV. No necesitas saber qué circuitos se activan dentro para cambiar al canal 7; solo presionas el botón `[7]`. El botón es la "notación natural" que oculta la complejidad interna.

---

##### 2. Suposiciones de `find(e)` sobre la igualdad
**Requisito clave:** El tipo de dato `T` debe tener definido el operador de igualdad (`operator==`).

**Cómo funciona:** El algoritmo de búsqueda secuencial compara exhaustivamente entre el argumento `e` y los elementos del vector hasta que `(e == _elem[i])` sea verdadero.

**Explicación:** Cuando le pides al vector que "busque el chocolate de marca X", el vector necesita saber qué significa que dos chocolates sean "iguales". ¿Es por el sabor? ¿Por el color? ¿Por la marca?

**Analogía:** Imagina que buscas a un amigo en una multitud. Tu cerebro tiene un "algoritmo de búsqueda" que compara cada cara que ves con la imagen de tu amigo. La igualdad es el reconocimiento de que la cara que ves coincide con la que buscas. Si no supieras cómo es tu amigo, jamás podrías "encontrarlo".

---

##### 3. Procesamiento uniforme mediante `traverse()`
**Qué hace:** Itera exhaustivamente sobre el intervalo $[0, \text{\_size})$ y aplica un functor (objeto función) a cada elemento.

**Patrón de diseño:** Manifestación del patrón Iterador + aplicación de funciones sobre colecciones.

**Ventaja:** Separa la lógica de "cómo recorrer" (el camino) de "qué hacer" (la tarea).

**Analogía:** Imagina un camión de riego que pasa por una calle llena de árboles. El camión no decide a qué árbol regar y a cuál no; simplemente sigue la calle (recorrido uniforme) y suelta agua en cada uno (acción aplicada). No importa si hay 10 o 100 árboles, el proceso es el mismo para todos.

---

##### 4. La lectura como refuerzo de `DengVector`
**Relación lectura-código:**

| En el código (`DengVector.h`) | En la lectura (marco teórico) |
|------------------------------|------------------------------|
| Ves QUÉ pasa (los datos se mueven) | Entiendes el PORQUÉ (para mantener contigüidad y eficiencia) |
| Implementas `expand()` | Justificas la duplicación geométrica con análisis amortizado |
| Usas `copyFrom()` | Comprendes copia profunda vs. superficial y gestión de ciclo de vida |

**Explicación:** Piensa en el `DengVector` como el laboratorio y en esta lectura como el manual de física. En el código ves qué pasa, pero en la lectura entiendes el porqué.

**Analogía:** Es como aprender a cocinar siguiendo una receta (`DengVector`) y luego leer un libro sobre química culinaria (Lectura 4). La receta te da el plato, pero la química te explica por qué el pan sube o por qué la carne se dora. Ambos son necesarios para ser un maestro chef.

---

### Tabla Técnica: Método Feynman

| Término Técnico | Explicación "Para un amigo" | Función / Utilidad Clave |
|----------------|----------------------------|-------------------------|
| Rango (Range) | El número de personas que están delante de ti en una fila | Identifica de forma única a un elemento por su posición |
| Copia Profunda (Deep Copy) | No solo prestar la llave de tu casa, sino construir una casa idéntica para tu amigo | Evita que dos partes del programa borren el mismo dato por accidente |
| Factor de Carga | El porcentaje de ocupación de un autobús (pasajeros / asientos) | Indica cuándo es hora de buscar un transporte más grande o más pequeño |
| Análisis Amortizado | Pagar una cuota anual de gimnasio: es un gasto grande un día, pero los otros 364 días parece "gratis" | Explica por qué el vector es rápido en promedio aunque a veces deba crecer |
| Contigüidad Física | Vivir en una casa pegada a la de tu vecino, sin espacios vacíos en medio | Es lo que permite que la computadora encuentre cualquier dato al instante |
| Sensible a la Entrada | Un algoritmo que termina rápido si tiene suerte, pero tarda si los datos son difíciles | Ayuda a entender el "Mejor Caso" vs. el "Peor Caso" en búsquedas |

---

## Bloque 7 - Cierre comparativo
### La Transición del Arreglo a la Estructura Dinámica

#### I. Respuesta Formal y Rigurosa (sintetizada)
El paso de un arreglo estático a una estructura dinámica representa un cambio de paradigma:

| Aspecto | Arreglo Estático | Estructura Dinámica |
|---------|-----------------|-------------------|
| **Representación** | Solo dirección base | Incluye metadatos: `_size`, `_capacity`, `_elem` |
| **Correctitud** | Programador verifica límites | Estructura preserva invariantes automáticamente |
| **Costo** | Falla ante desbordamiento | Crecimiento geométrico → costo amortizado $O(1)$ |
| **Espacio** | Exacto, pero inflexible | Trade-off: desperdicio temporal por flexibilidad |

---

#### II. Explicación Didáctica y Analógica
Pasar de usar un arreglo a diseñar una estructura es como dejar de cargar tus pertenencias en las manos para empezar a usar una mochila inteligente expandible.

1. **Representación (El Contenedor vs. El Contenido)**
   - Arreglo simple: Tú tienes que recordar cuántas cosas guardaste.
   - Estructura dinámica: La estructura "sabe" quién es. Tiene `_elem` (el suelo), `_capacity` (espacios totales) y `_size` (usados realmente).

2. **Correctitud (Las Reglas del Juego)**
   - Arreglo: Fácil cometer errores y escribir fuera de los límites.
   - Estructura: Tiene reglas irrompibles (invariantes). Si intentas meter algo donde no cabe, ella misma llama a `expand()`.

3. **Costo Amortizado (El Ahorro a Largo Plazo)**
   - Cada vez que tu mochila se llena, compras una nueva y pasas todo. Suena lento, ¿verdad?
   - Pero si cada vez compras una mochila el doble de grande, cada vez te mudarás con menos frecuencia.
   - Al final, el tiempo "por cosa guardada" es mínimo → costo amortizado $O(1)$.

4. **Uso de Espacio (El Aire en la Caja)**
   - Arreglo estático: Perfecto, usa exactamente el espacio que le pides.
   - Estructura dinámica: "Generosa", a veces tiene espacios vacíos listos.
   - Trade-off: Pequeño sacrificio de memoria por gran velocidad de ejecución.

---

#### III. Comparativa de Estructuras Dinámicas

| Estructura | Filosofía de Diseño | Ventaja Clave | Desventaja / Costo |
|-----------|-------------------|--------------|-------------------|
| **ArrayStack** | La forma más pura y simple de lista sobre arreglo | Simplicidad extrema y acceso directo $O(1)$ | Desplazamientos lentos en inserciones intermedias |
| **FastArrayStack** | Optimización de bajo nivel de ArrayStack | Usa copia de bloques de memoria → más rápida en la práctica | Mantiene misma complejidad teórica $O(n)$ en peor caso |
| **RootishArrayStack** | Fragmentación inteligente en bloques crecientes | Reduce desperdicio de espacio a $O(\sqrt{n})$ | Acceso un poco más complejo por mapeo matemático (`i2b`) |

---

#### IV. Resumen de Conceptos (Método Feynman)

| Concepto | ¿Cómo explicarlo a un niño? | ¿Por qué es importante? |
|----------|----------------------------|------------------------|
| Invariante | Es como una regla de un juego que NUNCA puede romperse | Evita que el software falle de forma impredecible |
| Memoria Contigua | Es como una fila de casas en la misma calle, sin terrenos baldíos en medio | Permite encontrar cualquier dato instantáneamente sabiendo el número de casa |
| Crecimiento Geométrico | Cada vez que necesites más espacio, duplica el tamaño de tu casa | Es el secreto para que el costo de mudarse no arruine la velocidad del programa |
| Desplazamiento | Si alguien se sienta en medio de un banco ocupado, todos los demás deben moverse un asiento | Es el costo real que pagamos por mantener a todos los datos juntos y en orden |

> [!TIP] Analogía Final
> Usar un arreglo es como tener una hoja de papel fija: si se te acaba el espacio, tienes que tirarla y empezar en otra. Diseñar una estructura dinámica es como tener un cuaderno infinito que añade hojas mágicamente cuando llegas al final de la página. El cuaderno es más complejo de fabricar, pero infinitamente más útil para escribir una historia larga.

---

## Autoevaluación breve (para repasar antes de una sustentación)

### ✅ Qué podemos defender con seguridad:
- **Dualidad Tamaño/Capacidad:** `_size` (elementos lógicos) vs `_capacity` (espacio físico reservado).
- **Mecánica de la Amortización:** Duplicar capacidad en `expand()` convierte operación costosa $O(n)$ en costo promedio constante $O(1)$.
- **Acceso Directo:** Memoria contigua permite calcular direcciones físicas instantáneamente ($A + i \times s$).
- **Trade-off Espacial:** `RootishArrayStack` es superior en eficiencia de memoria ($O(\sqrt{n})$ de desperdicio) vs desperdicio lineal de `ArrayStack`.

### ⚠️ Qué todavía confundimos (y debemos repasar):
- **Factor Constante vs. Complejidad:** `FastArrayStack` y `ArrayStack` son ambos $O(n)$ en inserciones; lo que cambia es la velocidad real (constante $c$) gracias a copia de bloques por hardware.
- **Mapeo de Índices en Rootish:** La derivación exacta de la fórmula cuadrática en `i2b(i)` puede ser confusa sin apoyo visual de la serie aritmética.
- **Copia Profunda vs. Superficial:** Riesgo de que dos objetos apunten al mismo `_elem` si no se implementa correctamente el constructor de copia.

### 🎯 Qué evidencia usaríamos en una sustentación:
| Tipo de evidencia | Ejemplo concreto | Qué demuestra |
|------------------|-----------------|--------------|
| **Código** | `if (_size == _capacity) expand();` | Guardia de correctitud para evitar overflow |
| **Empírica** | Resultados de `resize_stress_week2.cpp` | Estructura no corrompe memoria tras cientos de expansiones/reducciones |
| **Matemática** | Sumatoria de Gauss $\sum_{i=1}^{r} i = \frac{r(r+1)}{2}$ | Capacidad del `RootishArrayStack` |
| **Diseño** | Implementación de `traverse()` | Desacoplamiento entre estructura (recorrido) y lógica de aplicación (functor) |

---

> [!SUCCESS] Criterio general de trabajo (Conexión Explícita)
> Para dominar este bloque, la conexión entre los conceptos debe ser total e invisible:
> - **Representación → Correctitud:** La tripleta `(_elem, _size, _capacity)` mantiene el invariante de nunca acceder a memoria no reservada.
> - **Costo → Amortización:** No evaluamos `insert()` por su peor caso aislado, sino por el beneficio que la duplicación geométrica otorga a las siguientes $n$ operaciones.
> - **Uso de Espacio → Estructura:** El diseño de bloques en `RootishArrayStack` es la respuesta de ingeniería al problema del desperdicio de memoria.
> - **Implementación → Rendimiento:** El uso de `std::copy_backward` en lugar de un bucle `for` manual separa una implementación académica de una industrial de alto rendimiento.

> ***Conclusión***
> No se trata de "hacer que el código compile", sino de entender que cada línea de código en `DengVector` o `RootishArrayStack` es una decisión deliberada de diseño para optimizar la CPU, la RAM o la estabilidad del sistema.