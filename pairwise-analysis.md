# Taller de Modelado Combinatorio y All-Pairs

## 1. Matriz de parámetros y valores

El caso consiste en probar el configurador de vehículos de SmartDrive Motors. Para organizar las pruebas, identificamos estos parámetros:

| ID | Parámetro | Valores posibles | Cantidad |
|---|---|---|---:|
| P1 | Motor | Gasolina, Híbrido, Eléctrico | 3 |
| P2 | Transmisión | Manual, Automática, Monomarcha | 3 |
| P3 | Frenos | Estándar, ABS, Regenerativo | 3 |
| P4 | Mercado | América, Europa, Asia | 3 |
| P5 | Modo de conducción | Eco, Sport, Autónomo | 3 |

En total tenemos cinco parámetros con tres valores cada uno. Se toman los nombres de las columnas y los ejemplos de la tabla de salida del taller, ya que el gráfico inicial presenta etiquetas repetidas.

## 2. Explosión combinatoria

Para calcular todas las combinaciones, multiplicamos la cantidad de valores de cada parámetro:

**3 × 3 × 3 × 3 × 3 = 3⁵ = 243 combinaciones.**

Si cada prueba dura 15 minutos:

- 243 × 15 = **3645 minutos**.
- 3645 ÷ 60 = **60,75 horas**, es decir, **60 horas y 45 minutos**.
- Esto equivale a unas **7,6 jornadas de 8 horas**, sin contar pausas ni correcciones.

Probar todo tomaría bastante tiempo de trabajo y aumentaría el costo del equipo de QA; por eso conviene reducir los casos de forma organizada. Las 243 combinaciones corresponden al modelo matemático antes de aplicar las restricciones.

## 3. Suite All-Pairs sin restricciones

All-Pairs, también llamado Pairwise, busca que cada combinación de valores entre dos parámetros aparezca al menos una vez. No prueba todas las configuraciones completas, sino todas las interacciones de dos parámetros.

La siguiente tabla se obtuvo con una selección voraz: en cada paso se agregó una combinación que cubriera la mayor cantidad de pares pendientes. Después se comprobó la cobertura con un script. No se afirma que sea la suite mínima posible.

| Caso | Motor | Transmisión | Frenos | Mercado | Modo |
|---|---|---|---|---|---|
| TC-01 | Gasolina | Monomarcha | Estándar | Asia | Eco |
| TC-02 | Eléctrico | Manual | ABS | Asia | Autónomo |
| TC-03 | Híbrido | Monomarcha | Regenerativo | Europa | Autónomo |
| TC-04 | Gasolina | Automática | ABS | Europa | Sport |
| TC-05 | Eléctrico | Automática | Regenerativo | América | Eco |
| TC-06 | Híbrido | Manual | Estándar | América | Sport |
| TC-07 | Eléctrico | Manual | Estándar | Europa | Eco |
| TC-08 | Eléctrico | Monomarcha | Regenerativo | Asia | Sport |
| TC-09 | Gasolina | Manual | Regenerativo | América | Autónomo |
| TC-10 | Híbrido | Monomarcha | ABS | América | Eco |
| TC-11 | Híbrido | Automática | Estándar | Asia | Autónomo |

### Comprobación y reducción

Hay **10 parejas de parámetros** y cada una tiene **3 × 3 = 9 pares de valores**. En total se deben cubrir **10 × 9 = 90 pares**.

La tabla cubre **90 de 90 pares (100 %)** con **11 pruebas**, frente a las 243 pruebas exhaustivas. La reducción es de **95.47 %**. A 15 minutos por caso, tomaría **165 minutos**.

Esta primera tabla representa el modelo matemático puro. Contiene configuraciones que las reglas del negocio no permiten; por eso todavía no debe ejecutarse directamente como una suite de configuraciones válidas.

## 4. Constraints o restricciones lógicas

Separamos las condiciones del taller en tres reglas para validarlas individualmente:

### Regla 1: transmisión del motor eléctrico

```text
IF Motor = Eléctrico
THEN Transmisión = Monomarcha
```

En este configurador, un motor eléctrico debe llevar transmisión monomarcha.

### Regla 2: frenos del motor eléctrico

```text
IF Motor = Eléctrico
THEN Frenos = Regenerativo
```

Según las reglas del caso, el motor eléctrico debe utilizar frenos regenerativos.

### Regla 3: frenos del motor de gasolina

```text
IF Motor = Gasolina
THEN Frenos != Regenerativo
```

El configurador no permite combinar un motor de gasolina con frenos regenerativos.

Estas son reglas del modelo del taller, no afirmaciones generales sobre todos los vehículos reales.

### Suite válida después de aplicar las restricciones

No basta con eliminar las filas inválidas de la tabla anterior, porque podrían perderse pares permitidos. Por eso se generó otra suite usando solamente configuraciones válidas y se volvió a comprobar la cobertura.

| Caso | Motor | Transmisión | Frenos | Mercado | Modo |
|---|---|---|---|---|---|
| TC-01 | Híbrido | Manual | ABS | América | Autónomo |
| TC-02 | Eléctrico | Monomarcha | Regenerativo | Europa | Sport |
| TC-03 | Gasolina | Automática | ABS | Asia | Eco |
| TC-04 | Híbrido | Monomarcha | Estándar | Asia | Sport |
| TC-05 | Gasolina | Automática | Estándar | Europa | Autónomo |
| TC-06 | Híbrido | Manual | Regenerativo | Europa | Eco |
| TC-07 | Gasolina | Monomarcha | Estándar | América | Eco |
| TC-08 | Híbrido | Automática | Regenerativo | América | Sport |
| TC-09 | Eléctrico | Monomarcha | Regenerativo | Asia | Autónomo |
| TC-10 | Gasolina | Manual | Estándar | Asia | Sport |
| TC-11 | Gasolina | Monomarcha | ABS | Europa | Sport |
| TC-12 | Eléctrico | Monomarcha | Regenerativo | América | Eco |

Con las tres reglas quedan **144 configuraciones completas válidas** y **85 pares de valores permitidos**. Esta tabla contiene **12 casos**, cumple las restricciones y cubre **85 de 85 pares válidos (100 %)**. Los pares imposibles, como Eléctrico–Manual, no se incluyen en esa cobertura.

El tiempo de ejecución sería de **180 minutos**. Frente a probar las 144 configuraciones válidas, la cantidad de casos se reduce un **91.67 %**.

La cobertura se verificó enumerando las configuraciones, filtrándolas con las tres reglas y comparando los pares presentes en ellas con los de la suite. No quedaron pares válidos sin cubrir.

## 5. Preguntas de transferencia

### 1. ¿Qué riesgo financiero y operativo corremos si ignoramos los constraints lógicos y enviamos la matriz matemática pura directamente al equipo de automatización (QA)?

Gastaríamos tiempo y dinero automatizando configuraciones que el producto no permite. Por ejemplo, se intentaría probar un motor eléctrico con transmisión manual. Esto podría generar reportes de errores que en realidad vienen de datos de prueba incorrectos, retrasar el trabajo y quitar tiempo a las configuraciones válidas. Estas combinaciones pueden servir como pruebas negativas para comprobar que el sistema las rechaza, pero deben tener ese objetivo definido.

### 2. ¿Por qué la técnica All-Pairs es matemáticamente y empíricamente superior a que un tester diseñe 20 casos de prueba basándose únicamente en su intuición?

All-Pairs tiene una ventaja matemática porque permite comprobar que todos los pares de valores estén cubiertos. Si elegimos 20 casos solo por intuición, podemos repetir algunos pares y dejar otros sin probar. Además, el material del taller señala que los estudios del NIST encontraron muchos fallos relacionados con pocos parámetros, lo que apoya este enfoque. Aun así, cubrir todos los pares no garantiza encontrar todos los errores: pueden existir fallos que dependan de tres o más parámetros. Por eso la técnica complementa la experiencia del tester.

## 6. Conclusión

El ejercicio muestra que cinco parámetros sencillos ya producen 243 combinaciones. All-Pairs ayuda a reducir las pruebas manteniendo una cobertura comprobable de pares. También aprendimos que primero hay que entender las reglas del producto: una tabla puede estar completa matemáticamente y aun así contener configuraciones que no deberían aceptarse.

## Material de apoyo

- Documento de clase: *Taller Autónomo: Modelado Combinatorio y All-Pairs*, caso SmartDrive Motors.
