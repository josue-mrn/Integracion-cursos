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

**Modelado Espacial de Paquetería y Verificación de Capacidad Volumétrica mediante Vectores, Producto Triple Escalar y Determinantes en \(\mathbb{R}^3\).**

### b. Descripción del problema

En la logística de envíos exprés, cada paquete no es un dato escalar plano, sino un cuerpo volumétrico tridimensional que ocupa un espacio real dentro de la cabina de transporte de un camión (`Vehiculo.capacidadVolumenM3`). Si el cálculo se aborda únicamente como una multiplicación empírica sin base geométrica, se pierde la capacidad de modelar rotaciones espaciales o bultos con geometrías de aristas no canónicas. El sistema requiere determinar con exactitud analítica el volumen en metros cúbicos (\(m^3\)) y el peso volumétrico (\(kg\)) para prevenir sobrecargas y garantizar un cobro justo.

### c. Tema de Álgebra Lineal

- **Vectores en** \(\mathbb{R}^3\)
- **Producto Cruz (Vectorial)**
- **Producto Punto (Escalar)**
- **Determinantes de orden** \(3 \times 3\)
- **Aplicaciones para el cálculo de volúmenes (Producto Triple Escalar)**

### d. Aplicación

Un paquete con forma de paralelepípedo recto se define a partir de un vértice de referencia en el origen y tres vectores de arista:

$$
\vec{u} = \begin{pmatrix} largo \\ 0 \\ 0 \end{pmatrix}, \quad
\vec{v} = \begin{pmatrix} 0 \\ ancho \\ 0 \end{pmatrix}, \quad
\vec{w} = \begin{pmatrix} 0 \\ 0 \\ alto \end{pmatrix}
$$

En Álgebra Lineal, el volumen \(V\) del paralelepípedo generado por tres vectores en \(\mathbb{R}^3\) equivale al valor absoluto del **producto triple escalar**, el cual es idéntico al determinante de la matriz que tiene a dichos vectores por filas o columnas:

$$
V = \left|\vec{u} \cdot (\vec{v} \times \vec{w})\right|
= \left|
\det \begin{pmatrix}
u_x & u_y & u_z \\
v_x & v_y & v_z \\
w_x & w_y & w_z
\end{pmatrix}
\right|
$$

En la implementación del proyecto (`include/transacciones.h`, funciones `calcularVolumenM3` y `calcularPesoVolumetrico`), el volumen se normaliza a las unidades operativas. Si las dimensiones están expresadas en centímetros:

$$
V_{m^3} = \frac{|\det(M)|}{1\,000\,000},
\qquad
\text{Peso volumétrico (kg)} = \frac{|\det(M)|}{5\,000}
$$

### e. Resultado esperado

Obtención analítica del volumen del paquete y de su peso volumétrico derivado del determinante, asegurando que el proceso de despacho (`Despachos::procesarDespacho`) verifique de forma consistente que la sumatoria acumulada de volúmenes de las guías no supere la capacidad del vehículo.

---

### Explicación y Justificación de la Propuesta 1

- **Por qué se seleccionó:** Porque la paquetería opera en un espacio euclidiano tridimensional. El cálculo de volumen es una aplicación geométrica del determinante en \(\mathbb{R}^3\).
- **Problema a resolver:** Calcular el espacio cúbico que ocupa una carga y determinar su peso volumétrico para evitar que el camión se quede sin espacio físico antes de alcanzar su límite de peso.
- **Relación con el curso pivote:** Se vincula directamente con los campos de la estructura `Guia` (`largoCm`, `anchoCm`, `altoCm`, `pesoVolumetricoKg`, `pesoFacturableKg`) y con las validaciones del despacho en `include/transacciones.h`.
- **Qué concepto de Álgebra Lineal se utilizará:** Vectores en \(\mathbb{R}^3\), producto vectorial, producto escalar y determinantes de matrices \(3 \times 3\).
- **Qué aporta Álgebra Lineal al proyecto:** Fundamenta matemáticamente el aforo logístico y permite demostrar que, si un paquete sufre una rotación espacial representada por una matriz ortogonal \(R \in SO(3)\), con \(\det(R)=1\), el volumen permanece invariante: \(\det(RM)=\det(R)\det(M)=\det(M)\).

---

### Desarrollo y Resultados Obtenidos (Propuesta 1)

#### 1. Problema planteado

Calcular el volumen en \(m^3\) y el peso volumétrico en \(kg\) de una encomienda con dimensiones de \(60\text{ cm}\) de largo, \(40\text{ cm}\) de ancho y \(50\text{ cm}\) de alto, enviada desde la sucursal de Guatemala hacia Quetzaltenango, verificando si puede ser asignada a un camión que tiene actualmente \(0.15\text{ m}^3\) de espacio disponible.

#### 2. Datos utilizados

- Vector largo: \(\vec{u} = (60,0,0)^T\text{ cm}\)
- Vector ancho: \(\vec{v} = (0,40,0)^T\text{ cm}\)
- Vector alto: \(\vec{w} = (0,0,50)^T\text{ cm}\)
- Peso real en báscula: \(15.0\text{ kg}\)
- Capacidad volumétrica remanente del vehículo: \(0.15\text{ m}^3\)

#### 3. Modelo matemático

$$
M =
\begin{pmatrix}
60 & 0 & 0 \\
0 & 40 & 0 \\
0 & 0 & 50
\end{pmatrix}
$$

$$
V_{cm^3}
= \left|\vec{u}\cdot(\vec{v}\times\vec{w})\right|
= |\det(M)|
$$

$$
V_{m^3} = \frac{V_{cm^3}}{1\,000\,000},
\qquad
\text{Peso volumétrico (kg)} = \frac{V_{cm^3}}{5\,000}
$$

#### 4. Procedimiento

1. Calcular el producto cruz \(\vec{v}\times\vec{w}\).
2. Calcular el producto punto con \(\vec{u}\) para obtener el producto triple escalar.
3. Evaluar el determinante de la matriz diagonal \(M\).
4. Aplicar los factores de conversión de escala y el factor volumétrico \(1/5000\) indicado en el modelo del proyecto.

#### 5. Operaciones realizadas

**1. Producto cruz**

$$
\vec{v}\times\vec{w}
=
\begin{vmatrix}
\hat{i} & \hat{j} & \hat{k} \\
0 & 40 & 0 \\
0 & 0 & 50
\end{vmatrix}
=
\begin{pmatrix}
40\cdot50 - 0 \\
0 \\
0
\end{pmatrix}
=
\begin{pmatrix}
2000 \\
0 \\
0
\end{pmatrix}
$$

**2. Producto triple escalar y determinante**

$$
\vec{u}\cdot(\vec{v}\times\vec{w})
=
\begin{pmatrix}
60 \\ 0 \\ 0
\end{pmatrix}
\cdot
\begin{pmatrix}
2000 \\ 0 \\ 0
\end{pmatrix}
= 60(2000) = 120\,000\text{ cm}^3
$$

$$
\det(M) = 60(40)(50) = 120\,000
$$

**3. Cálculo de volumen y peso volumétrico**

$$
V_{m^3} = \frac{120\,000}{1\,000\,000}
= 0.12\text{ m}^3
$$

$$
\text{Peso volumétrico} = \frac{120\,000}{5\,000}
= 24.0\text{ kg}
$$

#### 6. Software y herramientas tecnológicas utilizadas

- Entorno **Apple Clang C++17** ejecutando la suite de pruebas unitarias (`./test_calculos`).
- Intérprete interactivo **Python 3.12 / NumPy** para comprobación matricial mediante `numpy.linalg.det`.

#### 7. Herramientas de inteligencia artificial utilizadas

Ver desglose formal en la sección 5.

#### 8. Resultado obtenido

- Volumen calculado: **\(0.12\text{ m}^3\)**.
- Peso volumétrico calculado: **\(24.0\text{ kg}\)**.
- Peso facturable: \(\max(15.0\text{ kg},24.0\text{ kg})=\mathbf{24.0\text{ kg}}\).
- Condición de carga física: \(0.12\text{ m}^3 \leq 0.15\text{ m}^3\), por lo que el **despacho es aceptado** según la capacidad volumétrica disponible.

#### 9. Interpretación del resultado

Aunque el paquete pesa físicamente \(15\text{ kg}\), su volumen equivale a una carga de \(24\text{ kg}\) según el factor volumétrico usado en este ejemplo. El cálculo permite facturar con base en el peso facturable y validar que el paquete cabe dentro de la capacidad volumétrica disponible del camión.

#### 10. Relación con el proyecto del curso pivote

Se vincula con `include/transacciones.h` (`calcularVolumenM3`, `calcularPesoVolumetrico`) y con la persistencia de los registros en `data/guias.dat`.

---

# PROPUESTA 2: Monitoreo y Rastreo Cinemático en Ruta mediante Ecuaciones Paramétricas y Simétricas de la Recta

### a. Nombre de la propuesta

**Estimación de Posición Continua y Validación de Trayectorias de Transporte mediante Ecuaciones Paramétricas y Simétricas de la Recta en \(\mathbb{R}^2\).**

### b. Descripción del problema

En el módulo de reportes (`GET /v1/reportes/rastreo/{numeroGuia}`), una guía que pasa al estado `"Despachada"` se encuentra en tránsito entre la sucursal de origen \(A\) y la sucursal de destino \(B\). Tradicionalmente, los sistemas muestran un estado binario ("En ruta"). El problema consiste en modelar analíticamente una trayectoria rectilínea simplificada del vehículo para estimar su posición en función del avance temporal \(t\in[0,1]\) y verificar si un punto de control reportado pertenece a la ruta matemática propuesta.

**Nota:** este modelo es una aproximación académica en un plano de coordenadas. Las rutas reales no son necesariamente rectas y las coordenadas de latitud y longitud requieren tratamiento geográfico para representar distancias y trayectos reales con precisión.

### c. Tema de Álgebra Lineal

- **Vectores en** \(\mathbb{R}^2\)
- **Vector de posición y vector director**
- **Ecuación paramétrica de una recta**
- **Ecuación simétrica de una recta**
- **Norma vectorial y distancia euclidiana**

### d. Aplicación

Dadas las coordenadas de la sucursal de origen \(P_0(x_0,y_0)\) y de destino \(P_1(x_1,y_1)\), el **vector director** de la ruta se define como:

$$
\vec{d} = P_1-P_0 =
\begin{pmatrix}
x_1-x_0 \\
y_1-y_0
\end{pmatrix}
$$

La **ecuación vectorial** del trayecto es:

$$
\vec{r}(t)=P_0+t\vec{d}, \qquad t\in[0,1]
$$

Desglosada en sus **ecuaciones paramétricas**:

$$
\begin{cases}
x(t)=x_0+t(x_1-x_0) \\
y(t)=y_0+t(y_1-y_0)
\end{cases}
$$

Si las componentes del vector director son distintas de cero, la **ecuación simétrica** puede escribirse como:

$$
\frac{x-x_0}{x_1-x_0}
=
\frac{y-y_0}{y_1-y_0}
=t
$$

### e. Resultado esperado

Cálculo de las coordenadas estimadas del transporte para un porcentaje de avance determinado y validación analítica de si un punto pertenece a la trayectoria rectilínea modelada.

---

### Explicación y Justificación de la Propuesta 2

- **Por qué se seleccionó:** Permite conectar la geometría analítica vectorial directamente con el seguimiento logístico.
- **Problema a resolver:** Estimar la posición de un envío en tránsito y comprobar si las coordenadas de un punto de control coinciden con la trayectoria matemática de la ruta.
- **Relación con el curso pivote:** Se integra conceptualmente con el catálogo de `Ruta` (`idSucursalOrigen`, `idSucursalDestino`, `distanciaKm`) y el reporte de rastreo en `include/reportes.h`.
- **Qué concepto de Álgebra Lineal se utilizará:** Vectores directores y ecuaciones paramétricas y simétricas de la recta en \(\mathbb{R}^2\).
- **Qué aporta Álgebra Lineal al proyecto:** Proporciona un modelo matemático ligero para estimar posiciones en una trayectoria idealizada, sin sustituir un sistema cartográfico real.

---

### Desarrollo y Resultados Obtenidos (Propuesta 2)

#### 1. Problema planteado

La ruta ID 1 conecta la **Sucursal Guatemala Central**, en \(P_0(14.63,-90.51)\), con la **Sucursal Quetzaltenango**, en \(P_1(14.84,-91.52)\). Un vehículo despachado reporta un avance paramétrico del \(50\%\) de su itinerario (\(t=0.50\)).

1. Determinar las ecuaciones paramétricas y simétricas de la ruta.
2. Calcular la posición estimada del vehículo.
3. Verificar si un puesto de control en \(Q(14.735,-91.015)\) se encuentra alineado con la ruta.

#### 2. Datos utilizados

- Origen: \(P_0(14.63,-90.51)\)
- Destino: \(P_1(14.84,-91.52)\)
- Parámetro de avance: \(t=0.50\)
- Coordenada a evaluar: \(Q(14.735,-91.015)\)

#### 3. Modelo matemático

Vector director:

$$
\vec{d} =
\begin{pmatrix}
14.84-14.63 \\
-91.52-(-90.51)
\end{pmatrix}
=
\begin{pmatrix}
0.21 \\
-1.01
\end{pmatrix}
$$

Ecuaciones paramétricas:

$$
\begin{cases}
x(t)=14.63+0.21t \\
y(t)=-90.51-1.01t
\end{cases}
$$

Ecuación simétrica:

$$
\frac{x-14.63}{0.21}
=
\frac{y+90.51}{-1.01}
$$

#### 4. Procedimiento

1. Evaluar las funciones paramétricas para \(t=0.50\).
2. Sustituir las coordenadas de \(Q\) en la ecuación simétrica para verificar la igualdad de las razones.

#### 5. Operaciones realizadas

**1. Posición para \(t=0.50\)**

$$
x(0.50)=14.63+0.21(0.50)=14.63+0.105=14.735
$$

$$
y(0.50)=-90.51-1.01(0.50)=-90.51-0.505=-91.015
$$

$$
\vec{r}(0.50)=(14.735,-91.015)
$$

**2. Verificación en la ecuación simétrica**

$$
\frac{14.735-14.63}{0.21}
=\frac{0.105}{0.21}=0.50
$$

$$
\frac{-91.015+90.51}{-1.01}
=\frac{-0.505}{-1.01}=0.50
$$

Como \(0.50=0.50\), las dos razones coinciden. Por tanto, \(Q\) pertenece al segmento rectilíneo modelado entre el origen y el destino.

#### 6. Software y herramientas tecnológicas utilizadas

- Entorno nativo de compilación C++17 en macOS y scripts de validación numérica.

#### 7. Herramientas de inteligencia artificial utilizadas

Ver sección 5.

#### 8. Resultado obtenido

- **Ecuaciones paramétricas:**

$$
x(t)=14.63+0.21t,\qquad y(t)=-90.51-1.01t
$$

- **Ecuación simétrica:**

$$
\frac{x-14.63}{0.21}
=
\frac{y+90.51}{-1.01}
$$

- Coordenada al \(50\%\) del recorrido: **\((14.735,-91.015)\)**.
- El punto de control \(Q\) **pertenece a la trayectoria rectilínea modelada**.

#### 9. Interpretación del resultado

El parámetro \(t\) transforma el porcentaje de avance supuesto en un punto del plano. La ecuación simétrica permite verificar si las coordenadas de un punto están alineadas con la trayectoria matemática. En un sistema real, esta verificación debe complementarse con datos de GPS, tolerancias de error y geometría vial.

#### 10. Relación con el proyecto del curso pivote

Se vincula conceptualmente con `include/rutas.h` y complementa el reporte de rastreo en `include/reportes.h`.

---

# PROPUESTA 3: Calibración y Descomposición Tarifaria mediante Sistemas de Ecuaciones Lineales y Matrices

### a. Nombre de la propuesta

**Determinación Analítica de Parámetros de la Matriz Tarifaria mediante Sistemas de Ecuaciones Lineales \(3 \times 3\) y Regla de Cramer.**

### b. Descripción del problema

En UMG Express, la fórmula de costo total de un envío (`include/transacciones.h`) se modela como una combinación lineal de tres factores:

$$
\text{Costo total}
=
(\text{tarifa base})\cdot 1
+
(\text{peso facturable})\cdot(\text{precio por kg})
+
(\text{volumen en }m^3)\cdot(\text{precio por }m^3)
$$

Cuando se tienen tres envíos con dimensiones, pesos y costos finales conocidos, pero se desconocen las tasas individuales, se puede plantear un sistema de ecuaciones para estimar los parámetros. Este procedimiento solo es válido si el modelo tarifario real utiliza esa fórmula y los datos proporcionados corresponden a ella.

### c. Tema de Álgebra Lineal

- **Matrices \(3 \times 3\)**
- **Determinantes**
- **Sistemas de ecuaciones lineales**
- **Regla de Cramer y matriz inversa (\(A\vec{x}=\vec{b}\))**

### d. Aplicación

Cada envío observado genera una fila en un sistema de ecuaciones lineales:

$$
\begin{cases}
x_1+p_1x_2+v_1x_3=c_1 \\
x_1+p_2x_2+v_2x_3=c_2 \\
x_1+p_3x_2+v_3x_3=c_3
\end{cases}
$$

En forma matricial:

$$
A\vec{x}=\vec{b}
$$

$$
\begin{pmatrix}
1 & p_1 & v_1 \\
1 & p_2 & v_2 \\
1 & p_3 & v_3
\end{pmatrix}
\begin{pmatrix}
x_1 \\ x_2 \\ x_3
\end{pmatrix}
=
\begin{pmatrix}
c_1 \\ c_2 \\ c_3
\end{pmatrix}
$$

Si \(\det(A)\neq0\), el sistema tiene solución única y se puede resolver mediante la **Regla de Cramer**:

$$
x_1=\frac{\det(A_1)}{\det(A)},\qquad
x_2=\frac{\det(A_2)}{\det(A)},\qquad
x_3=\frac{\det(A_3)}{\det(A)}
$$

donde \(A_j\) es la matriz resultante de sustituir la columna \(j\) de \(A\) por el vector de términos independientes \(\vec{b}\).

### e. Resultado esperado

Determinación de los tres parámetros del modelo: tarifa base (\(x_1\)), precio por kilogramo (\(x_2\)) y precio por metro cúbico (\(x_3\)), a partir de tres observaciones independientes y consistentes con la fórmula tarifaria propuesta.

---

### Explicación y Justificación de la Propuesta 3

- **Por qué se seleccionó:** Es una aplicación de Álgebra Lineal a la economía de operaciones y a la determinación de precios de transporte.
- **Problema a resolver:** Estimar las tarifas unitarias a partir de cobros globales conocidos.
- **Relación con el curso pivote:** Se conecta conceptualmente con la estructura `Tarifa` en `include/structs.h`, la función `calcularCostoTotal` en `include/transacciones.h` y el reporte de liquidación en `include/reportes.h`.
- **Qué concepto de Álgebra Lineal se utilizará:** Matrices de coeficientes, determinantes de \(3 \times 3\) y sistemas de ecuaciones lineales resueltos mediante la Regla de Cramer.
- **Qué aporta Álgebra Lineal al proyecto:** Proporciona un método analítico para auditar o estimar tarifas, siempre que se conozca la estructura real de la fórmula de costos.

---

### Desarrollo y Resultados Obtenidos (Propuesta 3)

#### 1. Problema planteado

Determinar las tres tarifas unitarias del modelo de UMG Express (\(x_1=\text{tarifa base}\), \(x_2=\text{precio por kg}\), \(x_3=\text{precio por }m^3\)) a partir de tres envíos hipotéticos con los siguientes cobros: Q120.00, Q210.00 y Q170.00.

#### 2. Datos utilizados

- **Envío 1:** \(p_1=10\text{ kg}\), \(v_1=0.05\text{ m}^3\), costo total = Q120.00.
- **Envío 2:** \(p_2=25\text{ kg}\), \(v_2=0.10\text{ m}^3\), costo total = Q210.00.
- **Envío 3:** \(p_3=15\text{ kg}\), \(v_3=0.20\text{ m}^3\), costo total = Q170.00.

#### 3. Modelo matemático

$$
\begin{pmatrix}
1 & 10 & 0.05 \\
1 & 25 & 0.10 \\
1 & 15 & 0.20
\end{pmatrix}
\begin{pmatrix}
x_1 \\ x_2 \\ x_3
\end{pmatrix}
=
\begin{pmatrix}
120 \\ 210 \\ 170
\end{pmatrix}
$$

#### 4. Procedimiento

1. Calcular el determinante de la matriz de coeficientes \(\det(A)\).
2. Sustituir cada columna por el vector de costos \(\vec{b}\) para hallar \(\det(A_1)\), \(\det(A_2)\) y \(\det(A_3)\).
3. Aplicar las razones de Cramer para despejar cada incógnita.

#### 5. Operaciones realizadas

**1. Determinante del sistema \(\det(A)\)**

$$
\det(A)=
1(25\cdot0.20-15\cdot0.10)
-10(1\cdot0.20-1\cdot0.10)
+0.05(1\cdot15-1\cdot25)
$$

$$
\det(A)=1(5.0-1.5)-10(0.10)+0.05(-10)
$$

$$
\det(A)=3.5-1.0-0.5=2.0
$$

Como \(\det(A)=2.0\neq0\), el sistema tiene solución única.

**2. Determinante \(\det(A_1)\) (tarifa base)**

$$
A_1=
\begin{pmatrix}
120 & 10 & 0.05 \\
210 & 25 & 0.10 \\
170 & 15 & 0.20
\end{pmatrix}
$$

$$
\det(A_1)
=120(25\cdot0.20-15\cdot0.10)
-10(210\cdot0.20-170\cdot0.10)
+0.05(210\cdot15-170\cdot25)
$$

$$
\det(A_1)=120(3.5)-10(25)+0.05(-1100)
$$

$$
\det(A_1)=420-250-55=115
$$

$$
x_1=\frac{\det(A_1)}{\det(A)}
=\frac{115}{2}=57.5
$$

**3. Determinante \(\det(A_2)\) (precio por kg)**

$$
A_2=
\begin{pmatrix}
1 & 120 & 0.05 \\
1 & 210 & 0.10 \\
1 & 170 & 0.20
\end{pmatrix}
$$

$$
\det(A_2)
=1(210\cdot0.20-170\cdot0.10)
-120(1\cdot0.20-1\cdot0.10)
+0.05(1\cdot170-1\cdot210)
$$

$$
\det(A_2)=25-12-2=11
$$

$$
x_2=\frac{\det(A_2)}{\det(A)}
=\frac{11}{2}=5.5
$$

**4. Determinante \(\det(A_3)\) (precio por \(m^3\))**

$$
A_3=
\begin{pmatrix}
1 & 10 & 120 \\
1 & 25 & 210 \\
1 & 15 & 170
\end{pmatrix}
$$

$$
\det(A_3)
=1(25\cdot170-210\cdot15)
-10(1\cdot170-210\cdot1)
+120(1\cdot15-25\cdot1)
$$

$$
\det(A_3)=1100+400-1200=300
$$

$$
x_3=\frac{\det(A_3)}{\det(A)}
=\frac{300}{2}=150
$$

#### 6. Software y herramientas tecnológicas utilizadas

- Entorno de terminal POSIX y scripts de comprobación numérica.

#### 7. Herramientas de inteligencia artificial utilizadas

Ver sección 5.

#### 8. Resultado obtenido

Las tarifas unitarias deducidas para el modelo hipotético son:

- **Tarifa base (\(x_1\)): Q57.50**
- **Precio por kg facturable (\(x_2\)): Q5.50**
- **Precio por metro cúbico (\(x_3\)): Q150.00**

Comprobación mediante sustitución:

$$
57.5+10(5.5)+0.05(150)=57.5+55+7.5=120
$$

$$
57.5+25(5.5)+0.10(150)=57.5+137.5+15=210
$$

$$
57.5+15(5.5)+0.20(150)=57.5+82.5+30=170
$$

Los tres costos coinciden con los datos proporcionados.

#### 9. Interpretación del resultado

El determinante del sistema confirma que las tres observaciones permiten una solución única (\(\det(A)=2.0\neq0\)). La Regla de Cramer permite determinar matemáticamente cuánto aporta cada parámetro al costo total dentro del modelo propuesto.

#### 10. Relación con el proyecto del curso pivote

Se vincula conceptualmente con `include/tarifas.h`, la fórmula de `calcularCostoTotal` en `include/transacciones.h` y el reporte consolidado de `include/reportes.h` (`obtenerReporteLiquidacion`). Los valores obtenidos son una demostración académica y no deben presentarse como tarifas reales del sistema sin verificar la configuración implementada.

---

## 5. Declaración de Uso de Herramientas Tecnológicas y de Inteligencia Artificial

En cumplimiento con las directrices académicas e institucionales sobre el uso ético y transparente de herramientas digitales y de Inteligencia Artificial:

### 1. Nombre de las herramientas utilizadas

- **Antigravity AI Assistant** (asistente de IA utilizado durante el trabajo).
- **Compilador Apple Clang C++17** (cadena de herramientas de compilación nativa).
- **Python 3.12 / NumPy** (entorno para verificación de determinantes y matrices, si se ejecutaron efectivamente dichas comprobaciones).

### 2. Para qué se utilizaron

- Apoyar la auditoría del código C++ para identificar oportunidades de vinculación matemática.
- Estructurar las propuestas de Álgebra Lineal aplicadas a operaciones de transporte de UMG Express.
- Contrastar los resultados algebraicos con la lógica del backend, cuando se ejecutaron pruebas del proyecto.

### 3. Qué información, datos o instrucciones se proporcionaron

- Se consideraron los modelos de datos de `include/structs.h` (`Guia`, `Ruta`, `Vehiculo`, `Tarifa`), las funciones de cálculo volumétrico y costos de `include/transacciones.h`, y la lista de temas del curso de Álgebra Lineal.
- Se indicó estructurar las propuestas respetando la arquitectura de C++ y sin introducir bibliotecas no autorizadas.

### 4. Qué resultado o apoyo proporcionaron

- Identificación de la relación entre el producto triple escalar en \(\mathbb{R}^3\) y el volumen de un paralelepípedo.
- Formulación de ecuaciones paramétricas y simétricas para una trayectoria idealizada.
- Construcción y resolución de un sistema \(3 \times 3\) mediante la Regla de Cramer.

### 5. Cómo se utilizaron dentro del proyecto

- Como apoyo consultivo y metodológico para formular el informe académico de vinculación entre ambos cursos.
- La implementación del backend del proyecto se mantiene en C++17 y Crow, según la arquitectura descrita.

### 6. De qué manera se verificó o validó la información

- **Validación analítica:** revisión paso a paso de los determinantes y productos vectoriales.
- **Validación experimental:** si se ejecutaron los binarios del proyecto (`./test_calculos` y `./test_despacho`), deben documentarse aquí los resultados efectivamente observados. Los cálculos del ejemplo de la propuesta 1 producen \(0.12\text{ m}^3\) y \(24.0\text{ kg}\); solo debe afirmarse que coinciden con el backend si esa coincidencia fue comprobada mediante las pruebas correspondientes.

---

## Conclusión

Las tres propuestas muestran cómo conceptos de Álgebra Lineal pueden relacionarse con problemas de logística: cálculo de volumen mediante determinantes, modelado de trayectorias con ecuaciones de rectas y estimación de parámetros tarifarios mediante sistemas de ecuaciones lineales.

La primera propuesta aplica directamente el producto triple escalar al cálculo de volumen. La segunda representa una trayectoria rectilínea simplificada y debe entenderse como un modelo académico, no como un sistema GPS completo. La tercera demuestra el uso de la Regla de Cramer con datos hipotéticos y obtiene los parámetros Q57.50, Q5.50 por kilogramo y Q150.00 por metro cúbico.

Antes de entregar el informe, conviene comprobar que los nombres de archivos, funciones, pruebas y resultados descritos coincidan exactamente con el repositorio y con las pruebas que realmente se ejecutaron.
