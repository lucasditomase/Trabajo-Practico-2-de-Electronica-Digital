# Trabajo Práctico 2: Diseño de ALU de 4 bits

**Materia:** Electrónica Digital
**Institución:** Universidad Católica Argentina (UCA) – 2026

> Fuente: transcripción fiel de `Trabajo Práctico 2.pdf` (1 página).
> Nota: los metadatos internos del PDF dicen `Title: Trabajo Práctico 1`,
> pero el encabezado y contenido corresponden al Trabajo Práctico 2.

---

## Consigna Principal

Diseñar una **ALU (Unidad Aritmético-Lógica) de 4 bits**. La misma debe implementar
las operaciones aritméticas **suma y resta** (números en complemento a la base) y las
lógicas **AND y OR**. Se da un posible diagrama en bloques en la siguiente figura:

```text
                flags
                  ^
                  |
       a ------->|         r
       b ------->|  ALU  ------->
       Operación  |         ^
       Control -->|         |
```

* Entradas: `a[3:0]`, `b[3:0]`, `Operación/Control`.
* Salidas: `r[3:0]` (resultado), `flags`.

El trabajo práctico incluye el diseño de la **unidad de control** y debe ser detallado
a nivel de **compuertas lógicas y circuitos usados**, y debe ser implementado en el
simulador **Qucs**.

---

## Requisitos de Entrega

El trabajo deberá ser entregado por modalidad **tarea en EVA**. Eventualmente podrán
ser requeridas aclaraciones sobre lo realizado y solicitarse correcciones si fuera
necesario. Se deberá entregar un **informe** de lo realizado. El informe debe contar con
una **carátula** con el título del trabajo y los datos completos de los integrantes del grupo.

* Se debe dar una **descripción** de los circuitos implementados en el diseño.
* Se debe incluir en el informe el **esquema del circuito**.
* Se deben adjuntar los **archivos Qucs creados** (`.sch`).

> El trabajo es con aprobación y **nota individual**.

---

## Diseño implementado en `alu_4bit.sch`

Codificación de operación (`Op1 Op0`):

| Op1 | Op0 | Operación |
|:---:|:---:|:----------|
|  0  |  0  | Suma  `R = A + B` |
|  0  |  1  | Resta `R = A - B` (B en complemento a 2) |
|  1  |  0  | AND bit a bit `R[i] = A[i]·B[i]` |
|  1  |  1  | OR bit a bit `R[i] = A[i]+B[i]` |

Arquitectura:

1. **Acondicionador de B:** `B'[i] = B[i] XOR Sub`, con `Sub = Op0 AND (NOT Op1)`
   (vale 1 solo en resta).
2. **Sumador/restador ripple-carry de 4 bits:** 4× Full-Adder de 1 bit
   (`S = A XOR B' XOR Cin`, `Cout = AB' + ACin + B'Cin`), `Cin0 = Sub`,
   `Cout[i] -> Cin[i+1]`.
3. **Bloque lógico:** 4× AND + 4× OR bit a bit.
4. **MUX de salida:** 1× MUX 4:1 por bit (select `Op1 Op0`) entre
   suma/resta, AND, OR.
5. **Flags:**
   * `C = Cout3` (acarreo; en resta es `NOT borrow`).
   * `Z = NOT(R3+R2+R1+R0)`.
   * `V = Cout3 XOR Cout2` (overflow complemento a 2).
   * `N = R3`.
