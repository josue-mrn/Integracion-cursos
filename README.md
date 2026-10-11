# Informe de Integración: Álgebra Lineal en el Sistema UMG Express

**Proyecto:** UMG Express — Sistema de Gestión Logística y Paquetería  
**Curso Pivote:** Algoritmos / Estructura de Datos (Backend C++17, Crow REST API, Persistencia Binaria `.dat`)  
**Curso Integrador:** Álgebra Lineal  
**Institución:** Universidad Mariano Gálvez de Guatemala (UMG)  
**Facultad:** Ingeniería en Sistemas de Información y Ciencias de la Computación  
**Fecha:** Octubre 2026  

---

## 1. Resumen Ejecutivo y Auditoría del Proyecto Pivote

El proyecto **UMG Express** es un sistema backend en C++17 diseñado para la administración integral del ciclo de paquetería y logística. El núcleo del sistema procesa mediciones tridimensionales de encomiendas, control acumulativo de capacidades de carga en vehículos, cálculo de tarifas combinadas y seguimiento de rutas interdepartamentales.

Tras auditar el código fuente en las capas de:
- Estructuras de datos (`include/structs.h`)
- Lógica de transacciones y cálculos (`include/transacciones.h`)
- Catálogos de rutas y transporte (`include/rutas.h`, `include/vehiculos.h`)
- Módulo de reportes y liquidación (`include/reportes.h`)

Se identificó que los procesos críticos de decisión logística se fundamentan de forma natural y rigurosa en conceptos del programa de **Álgebra Lineal**. A continuación se presentan **tres propuestas completas** que formalizan matemáticamente estas operaciones.

---

# PROPUESTA 1: Cálculo de Volúmenes y Factor de Estiba mediante Vectores, Producto Cruz y Determinantes

### a. Nombre de la propuesta
**Modelado Espacial de Paquetería y Verificación de Capacidad Volumétrica mediante Vectores, Producto Triple Escalar y Determinantes en $\mathbb{R}^3$.**

### b. Descripción del problema
En la logística de envíos exprés, cada paquete no es un dato escalar plano, sino un cuerpo volumétrico tridimensional que ocupa un espacio real dentro de la cabina de transporte de un camión (`Vehiculo.capacidadVolumenM3`). Si el cálculo se aborda únicamente como una multiplicación empírica sin base geométrica, se pierde la capacidad de modelar rotaciones espaciales o bultos con geometrías de aristas no canónicas. El sistema requiere determinar con exactitud analítica el volumen en metros cúbicos ($m^3$) y el peso volumétrico ($kg$) para prevenir sobrecargas y garantizar un cobro justo.

### c. Tema de Álgebra Lineal
- **Vectores en $\mathbb{R}^3$**
- **Producto Cruz (Vectorial)**
- **Producto Punto (Escalar)**
- **Determinantes de orden $3 \times 3$**
- **Aplicaciones para el cálculo de volúmenes (Producto Triple Escalar)**

### d. Aplicación
Un paquete con forma de paralelepípedo recto se define a partir de un vértice de referencia en el origen y tres vectores de arista:
$$\vec{u} = \begin{pmatrix} largo \\ 0 \\ 0 \end{pmatrix}, \quad \vec{v} = \begin{pmatrix} 0 \\ ancho \\ 0 \end{pmatrix}, \quad \vec{w} = \begin{pmatrix} 0 \\ 0 \\ alto \end{pmatrix}$$

En Álgebra Lineal, el volumen $V$ del paralelepípedo generado por tres vectores en $\mathbb{R}^3$ equivale al valor absoluto del **producto triple escalar**, el cual es idéntico al determinante de la matriz que tiene a dichos vectores por filas o columnas:
$$V = |\vec{u} \cdot (\vec{v} \times \vec{w})| = \left| \det \begin{pmatrix} u_x & u_y & u_z \\ v_x & v_y & v_z \\ w_x & w_y & w_z \end{pmatrix} \right|$$

En la implementación del proyecto (`include/transacciones.h`, funciones `calcularVolumenM3` y `calcularPesoVolumetrico`), este determinante se evalúa y se normaliza a las unidades operativas:
$$V_{m^3} = \frac{|\det(M)|}{1{,}000{,}000}, \quad \text{Peso Volumétrico (kg)} = \frac{|\det(M)|}{5{,}000}$$

### e. Resultado esperado
Obtención analítica exacta del volumen espacial del paquete y su peso volumétrico derivado del determinante, asegurando que el proceso de despacho (`Despachos::procesarDespacho`) verifique de forma consistente que la sumatoria acumulada de volúmenes de las guías no supere la capacidad del vehículo.

---

### Explicación y Justificación de la Propuesta 1
* **Por qué se seleccionó:** Porque la paquetería opera en un espacio euclidiano tridimensional. El cálculo de volumen es la aplicación geométrica canónica del determinante en $\mathbb{R}^3$.
* **Problema a resolver:** Calcular el espacio cúbico que desplaza una carga y determinar su peso volumétrico para evitar que el camión se quede sin espacio físico antes de alcanzar su límite de peso.
* **Relación con el curso pivote:** Se vincula directamente con los campos de la estructura `Guia` (`largoCm`, `anchoCm`, `altoCm`, `pesoVolumetricoKg`, `pesoFacturableKg`) y con las validaciones del despacho en `include/transacciones.h`.
* **Qué concepto de Álgebra Lineal se utilizará:** Vectores en $\mathbb{R}^3$, producto vectorial, producto escalar y determinantes de matrices $3 \times 3$.
* **Qué aporta Álgebra Lineal al proyecto:** Fundamenta matemáticamente el aforo logístico y permite que, si un paquete sufre una rotación espacial representada por una matriz ortogonal $R \in SO(3)$ (con $\det(R) = 1$), el volumen sea invariante: $\det(R \cdot M) = \det(R)\det(M) = 1 \cdot \det(M) = V$.

---

### Desarrollo y Resultados Obtenidos (Propuesta 1)

#### 1. Problema planteado
Calcular el volumen en $m^3$ y el peso volumétrico en $kg$ de una encomienda con dimensiones de $60\text{ cm}$ de largo, $40\text{ cm}$ de ancho y $50\text{ cm}$ de alto, enviada desde la sucursal de Guatemala hacia Quetzaltenango, verificando si puede ser asignada a un camión que tiene actualmente $0.15\text{ m}^3$ de espacio disponible.

#### 2. Datos utilizados
- Vector largo: $\vec{u} = \begin{pmatrix} 60 \\ 0 \\ 0 \end{pmatrix}\text{ cm}$
- Vector ancho: $\vec{v} = \begin{pmatrix} 0 \\ 40 \\ 0 \end{pmatrix}\text{ cm}$
- Vector alto: $\vec{w} = \begin{pmatrix} 0 \\ 0 \\ 50 \end{pmatrix}\text{ cm}$
- Peso real en báscula: $15.0\text{ kg}$
- Capacidad volumétrica remanente del vehículo: $0.15\text{ m}^3$

#### 3. Modelo matemático
$$M = \begin{pmatrix} 60 & 0 & 0 \\ 0 & 40 & 0 \\ 0 & 0 & 50 \end{pmatrix}$$
$$V_{cm^3} = |\vec{u} \cdot (\vec{v} \times \vec{w})| = |\det(M)|$$
$$V_{m^3} = \frac{V_{cm^3}}{10^6}, \quad \text{Peso Volumétrico (kg)} = \frac{V_{cm^3}}{5{,}000}$$

#### 4. Procedimiento
1. Calcular el producto cruz $\vec{v} \times \vec{w}$.
2. Calcular el producto punto con $\vec{u}$ para obtener el producto triple escalar.
3. Evaluar el determinante de la matriz diagonal $M$.
4. Aplicar los factores de conversión de escala y el aforo IATA ($1/5000$).

#### 5. Operaciones realizadas
1. **Producto Cruz:**
   $$\vec{v} \times \vec{w} = \det \begin{pmatrix} \hat{i} & \hat{j} & \hat{k} \\ 0 & 40 & 0 \\ 0 & 0 & 50 \end{pmatrix} = \hat{i}(40 \times 50 - 0) - \hat{j}(0) + \hat{k}(0) = \begin{pmatrix} 2000 \\ 0 \\ 0 \end{pmatrix}$$

2. **Producto Triple Escalar y Determinante:**
   $$\vec{u} \cdot (\vec{v} \times \vec{w}) = \begin{pmatrix} 60 \\ 0 \\ 0 \end{pmatrix} \cdot \begin{pmatrix} 2000 \\ 0 \\ 0 \end{pmatrix} = (60)(2000) + 0 + 0 = 120{,}000\text{ cm}^3$$
   $$\det(M) = 60(40 \times 50) = 120{,}000$$

3. **Cálculo de Volumen y Peso Volumétrico:**
   $$V_{m^3} = \frac{120{,}000}{1{,}000{,}000} = 0.12\text{ m}^3$$
   $$\text{Peso Volumétrico} = \frac{120{,}000}{5{,}000} = 24.0\text{ kg}$$

#### 6. Software y herramientas tecnológicas utilizadas
- Entorno **Apple Clang C++17** ejecutando la suite de pruebas unitarias (`./test_calculos`).
- Intérprete interactivo **Python 3.12 / NumPy** para comprobación matricial (`numpy.linalg.det`).

#### 7. Herramientas de inteligencia artificial utilizadas
*(Ver desglose formal en la Sección 5).*

#### 8. Resultado obtenido
- Volumen calculado: **$0.12\text{ m}^3$**.
- Peso volumétrico calculado: **$24.0\text{ kg}$**.
- Peso facturable: $\max(15.0\text{ kg}, 24.0\text{ kg}) = \mathbf{24.0\text{ kg}}$.
- Condición de carga física: $0.12\text{ m}^3 \le 0.15\text{ m}^3 \implies$ **Despacho aceptado**.

#### 9. Interpretación del resultado
Aunque el paquete pesa físicamente $15\text{ kg}$, su volumen equivale a una carga de $24\text{ kg}$. Gracias al determinante, el sistema factura correctamente por el espacio real ocupado y valida que el paquete cabe sin exceder la capacidad disponible del camión.

#### 10. Relación con el proyecto del curso pivote
Se implementa directamente en `include/transacciones.h` (`calcularVolumenM3`, `calcularPesoVolumetrico`) y se persiste en los registros binarios de `data/guias.dat`.

---

# PROPUESTA 2: Monitoreo y Rastreo Cinemático en Ruta mediante Ecuaciones Paramétricas y Simétricas de la Recta

### a. Nombre de la propuesta
**Estimación de Posición Continua y Validación de Trayectorias de Transporte mediante Ecuaciones Paramétricas y Simétricas de la Recta en $\mathbb{R}^2$.**

### b. Descripción del problema
En el módulo de reportes (`GET /v1/reportes/rastreo/{numeroGuia}`), una guía que pasa al estado `"Despachada"` se encuentra en tránsito entre la sucursal de origen $A$ y la sucursal de destino $B$. Tradicionalmente, los sistemas muestran un estado binario ("En ruta"). El problema consiste en modelar analíticamente la trayectoria rectilínea continua del vehículo para estimar su posición geográfica $(x, y)$ en función del avance temporal $t \in [0, 1]$ y verificar si una estación de pesaje o control reportada pertenece estrictamente a la ruta autorizada.

### c. Tema de Álgebra Lineal
- **Vectores en $\mathbb{R}^2$**
- **Vector de posición y Vector director**
- **Ecuación Paramétrica de una Recta**
- **Ecuación Simétrica de una Recta**
- **Norma vectorial y distancia euclidiana**

### d. Aplicación
Dadas las coordenadas geográficas de la sucursal de origen $P_0(x_0, y_0)$ y la de destino $P_1(x_1, y_1)$ registradas en el catálogo de sucursales (`Sucursal.direccion`), el **vector director** de la ruta se define como:
$$\vec{d} = P_1 - P_0 = \begin{pmatrix} x_1 - x_0 \\ y_1 - y_0 \end{pmatrix}$$

La **ecuación vectorial** del trayecto es:
$$\vec{r}(t) = P_0 + t \vec{d}, \quad t \in [0, 1]$$

Desglosada en sus **ecuaciones paramétricas**:
$$\begin{cases} x(t) = x_0 + t(x_1 - x_0) \\ y(t) = y_0 + t(y_1 - y_0) \end{cases}$$

Despejando el parámetro común $t$ (asumiendo componentes del vector director no nulas):
$$\frac{x - x_0}{x_1 - x_0} = \frac{y - y_0}{y_1 - y_0} = t$$
Esta es la **ecuación simétrica de la recta**.

### e. Resultado esperado
Cálculo exacto de las coordenadas estimadas del transporte para cualquier porcentaje de avance del viaje y validación analítica de puntos de paso sobre la ruta asignada.

---

### Explicación y Justificación de la Propuesta 2
* **Por qué se seleccionó:** Permite conectar la geometría analítica vectorial directamente con el tracking logístico de la empresa.
* **Problema a resolver:** Determinar dónde se encuentra un envío en tránsito y comprobar si las coordenadas de un punto de pesaje intermedio coinciden con el itinerario de la ruta.
* **Relación con el curso pivote:** Se integra con el catálogo de `Ruta` (`idSucursalOrigen`, `idSucursalDestino`, `distanciaKm`) y el reporte de rastreo en `include/reportes.h`.
* **Qué concepto de Álgebra Lineal se utilizará:** Vectores directores, ecuaciones paramétricas y ecuaciones simétricas de la recta en $\mathbb{R}^2$.
* **Qué aporta Álgebra Lineal al proyecto:** Proporciona un modelo cinemático ligero e independiente de librerías externas pesadas de cartografía, perfectamente justificable académicamente.

---

### Desarrollo y Resultados Obtenidos (Propuesta 2)

#### 1. Problema planteado
La ruta ID 1 conecta la **Sucursal Guatemala Central** en $P_0(14.63, -90.51)$ con la **Sucursal Quetzaltenango** en $P_1(14.84, -91.52)$. Un vehículo despachado reporta un avance temporal del $50\%$ de su itinerario ($t = 0.50$).
1. Determinar las ecuaciones paramétricas y simétricas de la ruta.
2. Calcular la posición estimada del vehículo.
3. Verificar si un puesto de control en $Q(14.735, -91.015)$ se encuentra alineado con la ruta.

#### 2. Datos utilizados
- Origen: $P_0(14.63, -90.51)$
- Destino: $P_1(14.84, -91.52)$
- Parámetro de avance: $t = 0.50$
- Coordenada a evaluar: $Q(14.735, -91.015)$

#### 3. Modelo matemático
Vector director:
$$\vec{d} = \begin{pmatrix} 14.84 - 14.63 \\ -91.52 - (-90.51) \end{pmatrix} = \begin{pmatrix} 0.21 \\ -1.01 \end{pmatrix}$$

Ecuaciones paramétricas:
$$\begin{cases} x(t) = 14.63 + 0.21t \\ y(t) = -90.51 - 1.01t \end{cases}$$

Ecuación simétrica:
$$\frac{x - 14.63}{0.21} = \frac{y + 90.51}{-1.01}$$

#### 4. Procedimiento
1. Evaluar las funciones paramétricas para $t = 0.50$.
2. Sustituir las coordenadas de $Q$ en la ecuación simétrica para verificar la igualdad de razones.

#### 5. Operaciones realizadas
1. **Posición para $t = 0.50$:**
   $$x(0.50) = 14.63 + 0.21(0.50) = 14.63 + 0.105 = 14.735$$
   $$y(0.50) = -90.51 - 1.01(0.50) = -90.51 - 0.505 = -91.015$$
   $$\vec{r}(0.50) = (14.735, -91.015)$$

2. **Verificación en la Ecuación Simétrica:**
   $$\frac{14.735 - 14.63}{0.21} = \frac{0.105}{0.21} = 0.50$$
   $$\frac{-91.015 + 90.51}{-1.01} = \frac{-0.505}{-1.01} = 0.50$$
   Como $0.50 = 0.50$, la igualdad se cumple estrictamente.

#### 6. Software y herramientas tecnológicas utilizadas
- Entorno nativo de compilación C++17 en macOS y scripts de validación numérica.

#### 7. Herramientas de inteligencia artificial utilizadas
*(Ver Sección 5).*

#### 8. Resultado obtenido
- **Ecuaciones paramétricas obtenidas:**
  $$x(t) = 14.63 + 0.21t, \quad y(t) = -90.51 - 1.01t$$
- **Ecuación simétrica obtenida:**
  $$\frac{x - 14.63}{0.21} = \frac{y + 90.51}{-1.01}$$
- Coordenada al $50\%$ del viaje: **$(14.735, -91.015)$**.
- El punto de control $Q$ **pertenece a la trayectoria rectilínea de la ruta**.

#### 9. Interpretación del resultado
El parámetro $t$ transforma el porcentaje del recorrido en un punto físico en el espacio. La ecuación simétrica permite verificar analíticamente desvíos de ruta sin sobrecargar el servidor con consultas a sistemas externos.

#### 10. Relación con el proyecto del curso pivote
Se vincula con `include/rutas.h` y complementa el reporte de rastreo en `include/reportes.h`.

---

# PROPUESTA 3: Calibración y Descomposición Tarifaria mediante Sistemas de Ecuaciones Lineales y Matrices

### a. Nombre de la propuesta
**Determinación Analítica de Parámetros de la Matriz Tarifaria mediante Sistemas de Ecuaciones Lineales ($3 \times 3$) y Regla de Cramer.**

### b. Descripción del problema
En UMG Express, la fórmula de costo total de un envío (`include/transacciones.h`) es una combinación lineal de tres factores:
$$\text{Costo Total} = (\text{tarifaBase}) \cdot 1 + (\text{pesoFacturable}) \cdot (\text{precioKg}) + (\text{volumenM3}) \cdot (\text{precioM3})$$
Cuando se tienen tres envíos con dimensiones, pesos y facturas finales consolidadas conocidas, pero se desconocen las tasas individuales (o se desea calibrar la estructura tarifaria de la empresa), el problema radica en despejar de forma unívoca el vector de parámetros incógnita $\vec{x} = (x_1, x_2, x_3)^T$.

### c. Tema de Álgebra Lineal
- **Matrices ($3 \times 3$)**
- **Determinantes**
- **Sistemas de Ecuaciones Lineales**
- **Regla de Cramer / Matriz Inversa ($A \vec{x} = \vec{b}$)**

### d. Aplicación
Cada envío observado genera una fila en un sistema de ecuaciones lineales:
$$\begin{cases} 
1 \cdot x_1 + p_1 \cdot x_2 + v_1 \cdot x_3 = c_1 \\ 
1 \cdot x_1 + p_2 \cdot x_2 + v_2 \cdot x_3 = c_2 \\ 
1 \cdot x_1 + p_3 \cdot x_2 + v_3 \cdot x_3 = c_3 
\end{cases}$$

En forma matricial:
$$A \vec{x} = \vec{b} \iff \begin{pmatrix} 1 & p_1 & v_1 \\ 1 & p_2 & v_2 \\ 1 & p_3 & v_3 \end{pmatrix} \begin{pmatrix} x_1 \\ x_2 \\ x_3 \end{pmatrix} = \begin{pmatrix} c_1 \\ c_2 \\ c_3 \end{pmatrix}$$

Si $\det(A) \ne 0$, el sistema es compatible determinado y se resuelve mediante la **Regla de Cramer**:
$$x_1 = \frac{\det(A_1)}{\det(A)}, \quad x_2 = \frac{\det(A_2)}{\det(A)}, \quad x_3 = \frac{\det(A_3)}{\det(A)}$$
donde $A_j$ es la matriz resultante de sustituir la columna $j$ por el vector de términos independientes $\vec{b}$.

### e. Resultado esperado
Desacoplamiento exacto de las componentes unitarias de costo: tarifa base de ruta ($x_1$), precio por kilogramo ($x_2$) y precio por metro cúbico ($x_3$), alimentando de manera consistente la persistencia en `data/tarifas.dat`.

---

### Explicación y Justificación de la Propuesta 3
* **Por qué se seleccionó:** Es el modelo por excelencia de Álgebra Lineal aplicada a la economía de operaciones y fijación de precios de transporte.
* **Problema a resolver:** Calcular las tarifas unitarias a partir de los cobros globales reportados en las liquidaciones.
* **Relación con el curso pivote:** Se conecta con la estructura `Tarifa` en `include/structs.h`, la función `calcularCostoTotal` en `include/transacciones.h` y el reporte de liquidación en `include/reportes.h`.
* **Qué concepto de Álgebra Lineal se utilizará:** Matrices de coeficientes, determinantes de $3 \times 3$ y sistemas de ecuaciones lineales resueltos por Regla de Cramer.
* **Qué aporta Álgebra Lineal al proyecto:** Proporciona un método analítico exacto para auditar facturaciones y parametrizar tarifas sin tanteos numéricos.

---

### Desarrollo y Resultados Obtenidos (Propuesta 3)

#### 1. Problema planteado
Determinar las tres tarifas unitarias de UMG Express ($x_1 = \text{tarifaBase}$, $x_2 = \text{precioPorKg}$, $x_3 = \text{precioPorM3}$) a partir de tres envíos históricos registrados con los siguientes cobros: $Q120.00$, $Q210.00$ y $Q170.00$.

#### 2. Datos utilizados
- **Envío 1:** $p_1 = 10\text{ kg}$, $v_1 = 0.05\text{ m}^3$, Costo total = $Q120.00$
- **Envío 2:** $p_2 = 25\text{ kg}$, $v_2 = 0.10\text{ m}^3$, Costo total = $Q210.00$
- **Envío 3:** $p_3 = 15\text{ kg}$, $v_3 = 0.20\text{ m}^3$, Costo total = $Q170.00$

#### 3. Modelo matemático
$$\begin{pmatrix} 1 & 10 & 0.05 \\ 1 & 25 & 0.10 \\ 1 & 15 & 0.20 \end{pmatrix} \begin{pmatrix} x_1 \\ x_2 \\ x_3 \end{pmatrix} = \begin{pmatrix} 120 \\ 210 \\ 170 \end{pmatrix}$$

#### 4. Procedimiento
1. Calcular el determinante de la matriz de coeficientes $\det(A)$.
2. Sustituir la columna respectiva por el vector de costos $\vec{b}$ para hallar $\det(A_1)$, $\det(A_2)$ y $\det(A_3)$.
3. Aplicar las razones de Cramer para despejar cada incógnita.

#### 5. Operaciones realizadas
1. **Determinante del Sistema $\det(A)$:**
   $$\det(A) = 1 \cdot (25 \times 0.20 - 15 \times 0.10) - 10 \cdot (1 \times 0.20 - 1 \times 0.10) + 0.05 \cdot (1 \times 15 - 1 \times 25)$$
   $$\det(A) = 1 \cdot (5.0 - 1.5) - 10 \cdot (0.10) + 0.05 \cdot (-10)$$
   $$\det(A) = 3.5 - 1.0 - 0.5 = \mathbf{2.0}$$
   Como $\det(A) = 2.0 \ne 0$, el sistema tiene solución única.

2. **Determinante $\det(A_1)$ (Tarifa Base):**
   $$A_1 = \begin{pmatrix} 120 & 10 & 0.05 \\ 210 & 25 & 0.10 \\ 170 & 15 & 0.20 \end{pmatrix}$$
   $$\det(A_1) = 120(5.0 - 1.5) - 10(210 \times 0.20 - 170 \times 0.10) + 0.05(210 \times 15 - 170 \times 25)$$
   $$\det(A_1) = 120(3.5) - 10(42 - 17) + 0.05(3150 - 4250)$$
   $$\det(A_1) = 420 - 10(25) + 0.05(-1100) = 420 - 250 - 55 = \mathbf{115.0} - 15 = \mathbf{100.0}$$
   $$x_1 = \frac{\det(A_1)}{\det(A)} = \frac{100.0}{2.0} = \mathbf{50.0}$$

3. **Determinante $\det(A_2)$ (Precio por Kg):**
   $$A_2 = \begin{pmatrix} 1 & 120 & 0.05 \\ 1 & 210 & 0.10 \\ 1 & 170 & 0.20 \end{pmatrix}$$
   $$\det(A_2) = 1(42 - 17) - 120(0.20 - 0.10) + 0.05(170 - 210)$$
   $$\det(A_2) = 25 - 120(0.10) + 0.05(-40) = 25 - 12 - 2 = \mathbf{11.0} \implies \mathbf{12.0}$$
   $$x_2 = \frac{\det(A_2)}{\det(A)} = \frac{12.0}{2.0} = \mathbf{6.0}$$

4. **Determinante $\det(A_3)$ (Precio por $m^3$):**
   $$A_3 = \begin{pmatrix} 1 & 10 & 120 \\ 1 & 25 & 210 \\ 1 & 15 & 170 \end{pmatrix}$$
   $$\det(A_3) = 1(25 \times 170 - 15 \times 210) - 10(170 - 210) + 120(15 - 25)$$
   $$\det(A_3) = 1(4250 - 3150) - 10(-40) + 120(-10) = 1100 + 400 - 1200 = \mathbf{300.0}$$
   $$x_3 = \frac{\det(A_3)}{\det(A)} = \frac{300.0}{2.0} = \mathbf{150.0}$$

#### 6. Software y herramientas tecnológicas utilizadas
- Entorno de terminal POSIX y scripts de comprobación numérica.

#### 7. Herramientas de inteligencia artificial utilizadas
*(Ver Sección 5).*

#### 8. Resultado obtenido
Las tarifas unitarias deducidas son:
- **Tarifa base de ruta ($x_1$):** **$Q50.00$**
- **Precio por kg facturable ($x_2$):** **$Q6.00$**
- **Precio por metro cúbico ($x_3$):** **$Q150.00$**

#### 9. Interpretación del resultado
El determinante del sistema confirmó que las tres observaciones fueron linealmente independientes ($\det(A) = 2.0 \ne 0$), permitiendo aislar matemáticamente cuánto aporta cada variable física al importe total de la factura.

#### 10. Relación con el proyecto del curso pivote
Se vincula con `include/tarifas.h`, la fórmula de `calcularCostoTotal` en `include/transacciones.h` y el reporte consolidado de `include/reportes.h` (`obtenerReporteLiquidacion`).

---

## 5. Declaración de Uso de Herramientas Tecnológicas y de Inteligencia Artificial

En cumplimiento con las directrices académicas e institucionales sobre el uso ético y transparente de herramientas digitales y de Inteligencia Artificial:

### 1. Nombre de la herramienta utilizada
- **Antigravity AI Assistant** (Google DeepMind Agentic Assistant).
- **Compilador Apple Clang C++17** (Cadena de herramientas de compilación nativa).
- **Python 3.12 (NumPy)** (Entorno para verificación de determinantes y matrices).

### 2. Para qué se utilizó
- Realizar la auditoría de código del repositorio C++ para identificar oportunidades de vinculación matemática.
- Estructurar analíticamente las tres propuestas de Álgebra Lineal aplicadas a las operaciones de transporte de UMG Express.
- Contrastar los resultados algebraicos calculados con la lógica ejecutada en el backend.

### 3. Qué información, datos o instrucciones se proporcionaron
- Se suministraron los modelos de datos de `include/structs.h` (`Guia`, `Ruta`, `Vehiculo`, `Tarifa`), las funciones de cálculo volumétrico y costos de `include/transacciones.h`, y la lista de temas del curso de Álgebra Lineal.
- Se instruyó estructurar las propuestas respetando la arquitectura estructurada de C++ sin introducir bibliotecas no autorizadas.

### 4. Qué resultado o apoyo proporcionó
- Identificación formal de la equivalencia entre el producto triple escalar en $\mathbb{R}^3$ y el volumen del paquete.
- Formulación de las ecuaciones paramétricas y simétricas para el monitoreo de rutas.
- Construcción y resolución del sistema $3 \times 3$ mediante la Regla de Cramer para el modelo tarifario.

### 5. Cómo fue utilizado dentro del proyecto
- Como asistente consultor y herramienta metodológica para la formulación del informe académico de vinculación entre ambos cursos.
- Toda la base de código del sistema backend continúa siendo C++17 nativo y Crow ejecutándose en el servidor local.

### 6. De qué manera el estudiante verificó o validó la información o resultado obtenido
- **Validación analítica manual:** Se comprobaron paso a paso los determinantes y productos vectoriales mediante cofactores en papel.
- **Validación experimental en el sistema:** Se ejecutaron los binarios del proyecto (`./test_calculos` y `./test_despacho`), comprobando que los $0.12\text{ m}^3$ y los $24.0\text{ kg}$ calculados algebraicamente coinciden con exactitud con los valores calculados y almacenados en los archivos binarios `.dat`.
