# Trabajo Práctico 2: Diseño de ALU de 4 bits — Informe (prototipo)

> **Estado:** prototipo en Markdown. Pendiente: exportar a PDF, capturas del esquema
> Qucs y tabla de verdad de simulación.

---

## Carátula

| Campo | Valor |
|:--|:--|
| **Universidad** | Universidad Católica Argentina (UCA) |
| **Materia** | Electrónica Digital |
| **Trabajo** | Trabajo Práctico 2: Diseño de ALU de 4 bits |
| **Año** | 2026 |
| **Integrantes** | Apellido, Nombre — Legajo (completar) |
| **Fecha** | (completar) |

---

## 1. Consigna y objetivos

Diseñar una **ALU (Unidad Aritmético-Lógica) de 4 bits** que implemente las
operaciones aritméticas **suma y resta** (números en complemento a la base) y las
lógicas **AND y OR**, incluyendo el diseño de la **unidad de control**, con detalle
a nivel de **compuertas lógicas y circuitos usados**, implementada en el simulador
**Qucs**.

Diagrama en bloques de referencia (ver `Trabajo Práctico 2.pdf`):

```text
                flags
                  ^
                  |
       a ------->|         r
       b ------->|  ALU  ------->
       Operación  |
       Control -->|
```

* Entradas: `a[3:0]`, `b[3:0]`, `Operación/Control`.
* Salidas: `r[3:0]` (resultado), `flags`.

## 2. Codificación de operación

| Op1 | Op0 | Operación | Descripción |
|:---:|:---:|:----------|:------------|
|  0  |  0  | Suma  | `R = A + B` |
|  0  |  1  | Resta | `R = A − B` (B en complemento a 2) |
|  1  |  0  | AND   | `R[i] = A[i] · B[i]` bit a bit |
|  1  |  1  | OR    | `R[i] = A[i] + B[i]` bit a bit |

## 3. Arquitectura general

Se implementó la siguiente cadena de procesamiento por bit `i` (`i = 0..3`):

1. **Acondicionador del operando B** (para resta en complemento a 2):
   `B'[i] = B[i] XOR Sub`, con `Sub = Op0 AND (NOT Op1)`.
   `Sub` vale 1 solo en resta (`Op = 01`).
2. **Sumador/restador ripple-carry de 4 bits:** 4× Full-Adder de 1 bit
   (`fa1b` de Qucs), con `Cin0 = Sub` y `Cout[i] → Cin[i+1]`.
3. **Bloque lógico:** 4× AND y 4× OR bit a bit.
4. **MUX de salida:** por bit, selección entre suma/resta, AND y OR según
   `Op1 Op0`, implementado a compuertas:
   `R[i] = S[i]·¬Op1 + AND[i]·Op1·¬Op0 + OR[i]·Op1·Op0`
   (la combinación `Op = 01` con salida aritmética queda cubierta por el término
   `S[i]·¬Op1`).
5. **Generación de flags:**
   * `C = Cout3` — acarreo final (en resta equivale a `NOT borrow`).
   * `Z = NOT(R3 + R2 + R1 + R0)` — 1 si el resultado es cero.
   * `V = Cout3 XOR Cout2` — overflow en complemento a 2.
   * `N = R3` — bit de signo del resultado.

## 4. Diseño a nivel de compuertas

### 4.1. Unidad de control

* `¬Op1`, `¬Op0`: 2 inversores (`Inv`).
* `Sub = Op0 · ¬Op1`: 1 AND de 2 entradas. Activa la inversión de B y el
  `Cin0 = 1` solo en resta.

### 4.2. Slice aritmético (×4, uno por bit)

* `XB[i]`: XOR `B[i] XOR Sub` → `B'[i]`.
* `FA[i]` (`fa1b`): entradas `A[i]`, `B'[i]`, `Cin`; salidas
  `S[i] = A XOR B' XOR Cin`, `Cout = AB' + ACin + B'Cin`.
* Acarreo serie: `Cin0 = Sub`, `Cin[i+1] = Cout[i]`; `C4` es el acarreo final.

### 4.3. Slice lógico (×4)

* `AN[i]`: AND `A[i] · B[i]`.
* `OU[i]`: OR `A[i] + B[i]`.

### 4.4. MUX de salida (×4)

Por bit `i`, con compuertas de 2 y 3 entradas:

* `M0[i] = S[i] · ¬Op1` (vía aritmética).
* `M1[i] = AND[i] · Op1 · ¬Op0` (vía AND).
* `M2[i] = OR[i] · Op1 · Op0` (vía OR).
* `R[i] = M0[i] + M1[i] + M2[i]` (OR de 3 entradas).

### 4.5. Flags

* `FC`: lleva `C4` (acarreo final).
* `FN`: lleva `R3`.
* `XV + FV`: `V = C4 XOR C3` (XOR de 2 entradas).
* `OZ + NZ + FZ`: `OZ = R3+R2+R1+R0` (OR de 4 entradas),
  `FZ = NOT OZ`.

### 4.6. Inventario de componentes (`alu_4bit.sch`)

| Componente Qucs | Cantidad | Uso |
|:----------------|:--------:|:----|
| `AND` (2/3 ent.) | 17 | 4 AND bit, 1 SUB, 4 M0, 4 M1, 4 M2 |
| `OR` (2/3/4 ent.) | 9 | 4 OR bit, 4 MR, 1 OZ |
| `XOR` (2 ent.) | 5 | 4 XB, 1 XV |
| `Inv` | 3 | ¬Op1, ¬Op0, NZ |
| `fa1b` | 4 | sumador ripple-carry |
| `In` / `Out` | 10 / 8 | A[3:0], B[3:0], Op — R[3:0], C/Z/V/N |
| `DigiSim` (truth) | 1 | simulación digital |

Total: 56 componentes, 151 wires (conexión por etiquetas de red, válida en
simulación digital `truth` de Qucs).

## 5. Implementación en Qucs

* Archivo: `alu_4bit.sch` (Qucs 0.0.19).
* Pines de entrada `In`: `A0..A3`, `B0..B3`, `OP0`, `OP1`.
* Pines de salida `Out`: `R0..R3`, `FC`, `FZ`, `FV`, `FN`.
* Simulación: bloque `DigiSim` en modo `truth` (tabla de verdad).
* Conexiones internas por etiquetas de red (stubs con label); Qucs las une por
  nombre en simulación digital, lo que evita el mazo de cables cruzados.

> **TODO (antes del PDF final):** agregar captura del esquema desde Qucs y la
> tabla de verdad obtenida del simulador como Figuras 1 y 2.

## 6. Verificación

* **Chequeo estructural:** 1 driver por red, sin redes flotantes, todos los
  wires Manhattan (horizontales/verticales).
* **Verificación lógica exhaustiva** (modelo aritmético independiente,
  16×16×4 = 1024 combinaciones de `A × B × Op`):
  * Suma: `R`, `C`, `V`, `Z`, `N` correctos en 256/256 casos.
  * Resta: `R`, `C` (= NOT borrow), `V`, `Z`, `N` correctos en 256/256 casos.
  * AND/OR: `R`, `Z`, `N` correctos en 512/512 casos.
  * **Total: 1024 OK, 0 fallas.**

> **TODO:** reemplazar esta verificación por script con la tabla de verdad
> exportada de Qucs una vez corrida la simulación.

## 7. Conclusiones

La ALU cumple la consigna: implementa suma, resta (complemento a 2, que es el
complemento a la base en binario), AND y OR de 4 bits, con unidad de control
propia, descripta a nivel de compuertas e implementada en Qucs. Los flags
`C/Z/V/N` extienden la salida `flags` del diagrama de referencia con la
semántica estándar.

## Anexos

* **Anexo A:** `Trabajo Práctico 2.pdf` (consigna oficial).
* **Anexo B:** `alu_4bit.sch` (esquemático Qucs — archivo adjunto en EVA).
* **Anexo C:** `consignas_trabajo_practico_2.md` (transcripción de la consigna).
