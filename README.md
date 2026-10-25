# Electrónica Digital - Trabajo Práctico 2
### Universidad Católica Argentina (UCA) - Facultad de Ingeniería y Ciencias Agrarias

Este repositorio contiene la resolución del **Trabajo Práctico 2** de la materia **Electrónica Digital** (Carrera de Ingeniería Informática, UCA, 2026).

El proyecto consiste en el diseño e implementación en **Qucs** de una **ALU de 4 bits** capaz de realizar suma, resta en complemento a 2, AND y OR. Incluye el informe técnico académico y los esquemas del circuito a nivel de compuertas lógicas.

**Integrantes:** Lucas Di Tomase y Valentín Michelic.

## Consulta Rápida del Informe

El [informe técnico en PDF](TP2%20-%20ALU%20de%204%20bits%20-%20Informe.pdf) presenta la consigna, la arquitectura, la unidad de control, las ecuaciones, los esquemas y los resultados de la simulación.

Se puede consultar directamente desde GitHub o descargar para su lectura. Para revisar el informe no es necesario instalar el simulador.

## Simulación Local

### Requisitos Previos

* **Qucs 0.0.19**, versión utilizada para la validación transitoria en Windows con qucsator.
* Para la variante digital opcional, un entorno de simulación digital operativo con FreeHDL y su compilador.

### Apertura y Ejecución

1. Clona o descarga este repositorio:
   ```bash
   git clone https://github.com/lucasditomase/Trabajo-Practico-2-de-Electronica-Digital.git
   ```
2. Abre **`alu_completo.sch`** en Qucs. Es el esquema principal de referencia.
3. Ejecuta el análisis transitorio configurado entre **0 y 51,2 µs**, con **5121 puntos** guardados.
4. Consulta las señales de entrada `A0`…`A3`, `B0`…`B3`, `OP0`, `OP1`, las salidas `R0`…`R3` y los flags `C4`, `FZ`, `FV` y `R3` en el dataset generado.

La simulación completa puede tardar. Los resultados obtenidos se presentan en el informe; el repositorio publica el PDF y los esquemas necesarios para repetirla.

| Esquema | Descripción |
| :--- | :--- |
| `alu_completo.sch` | Simulación transitoria principal de las cuatro operaciones: 1024 combinaciones. |
| `alu_tran_or.sch` | Variante auxiliar para la ventana OR, de 0 a 12,8 µs. |
| `alu_tran_and.sch` | Variante auxiliar para la ventana AND, de 12,8 a 25,6 µs. |
| `alu_tran_resta.sch` | Variante auxiliar para la ventana de resta, de 25,6 a 38,4 µs. |
| `alu_tran_suma.sch` | Variante auxiliar para la ventana de suma, de 38,4 a 51,2 µs. |
| `alu_4bit.sch` | Alternativa digital configurada para tabla de verdad. |

La evidencia de validación corresponde a la corrida transitoria principal. Las variantes auxiliares y la alternativa digital no se presentan como ejecuciones nativas adicionales.

## Estructura del Repositorio

```text
.
├── TP2 - ALU de 4 bits - Informe.pdf  # Informe técnico académico
├── alu_completo.sch                  # Esquema principal validado en Qucs
├── alu_tran_or.sch                   # Ventana transitoria de OR
├── alu_tran_and.sch                  # Ventana transitoria de AND
├── alu_tran_resta.sch                # Ventana transitoria de resta
├── alu_tran_suma.sch                 # Ventana transitoria de suma
├── alu_4bit.sch                      # Variante digital de la ALU
└── README.md                        # Descripción e instrucciones del proyecto
```

## Contenido del Trabajo Práctico

### Operaciones y Unidad de Control

La ALU recibe dos operandos de cuatro bits, `A` y `B`, y dos señales de selección, `OP1` y `OP0`. El resultado `R` conserva cuatro bits; las operaciones aritméticas se truncan módulo 16. En complemento a 2, el rango representable es **−8 a 7**.

| OP1 | OP0 | Operación |
| :---: | :---: | :--- |
| 0 | 0 | Suma: `A + B` |
| 0 | 1 | Resta: `A − B` |
| 1 | 0 | AND bit a bit |
| 1 | 1 | OR bit a bit |

La señal `SUB = OP0 AND NOT OP1` controla la inversión de `B` y el acarreo inicial. Así, suma y resta comparten cuatro sumadores completos conectados en ripple-carry. El bloque lógico calcula AND y OR, y un multiplexor a compuertas selecciona la salida.

### Indicadores de Estado

| Flag | Señal en Qucs | Significado |
| :---: | :---: | :--- |
| C | `C4` | Acarreo final; en resta indica ausencia de préstamo sin signo. |
| Z | `FZ` | El resultado seleccionado es cero. |
| V | `FV` | Overflow aritmético con signo: `C4 XOR C3`. |
| N | `R3` | Bit más significativo del resultado seleccionado. |

En AND y OR, **C y V reflejan el sumador A+B que funciona en paralelo**; no se interpretan como acarreo u overflow de la operación lógica. Z y N dependen del resultado seleccionado en las cuatro operaciones.

### Implementación y Validación

* **54 compuertas elementales**, con los sumadores completos desarrollados a nivel de compuertas.
* Unidad de control, acondicionamiento de B, sumador/restador, bloque lógico, selección de salida y generación de flags descritos en el informe.
* **1024 combinaciones distintas** verificadas en el dataset de Qucs: **256 por operación**, sin discrepancias en resultado ni flags.
* Informe con ecuaciones, esquemas, gráficos de simulación y casos representativos de acarreo, préstamo, cero y overflow.
