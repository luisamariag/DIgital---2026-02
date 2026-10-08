# 1. [Código]
El código en Verilog que se utilizo para el circuito sumador de 4 bit se presenta a continuación en la figura 1.
<img width="377" height="442" alt="image" src="https://github.com/user-attachments/assets/a26b4a8a-b597-43e0-9957-05f7188e5a28" />

Fig 1. Código verilog sumador 4 bit

## Desglose Línea a Línea: Sumador de 4 Bits (`sumador4b.v`)

## 1. Definición de Interfaz y Puertos (Líneas 1-7)

- **`module sumador4b (`**: Declara el inicio del módulo del sumador de 4 bits.
- **`input wire [3:0] A,`**: Bus de entrada de 4 bits para el primer número (`A[3]`, `A[2]`, `A[1]`, `A[0]`).
- **`input wire [3:0] B,`**: Bus de entrada de 4 bits para el segundo número (`B[3]`, `B[2]`, `B[1]`, `B[0]`).
- **`input wire Cin,`**: Entrada de acarreo inicial de 1 bit (`Cin`).
- **`output wire [3:0] Sum,`**: Bus de salida de 4 bits para el resultado (`Sum[3]`, `Sum[2]`, `Sum[1]`, `Sum[0]`).
- **`output wire Cout`**: Salida de 1 bit para el acarreo sobrante final.
- **`);`**: Cierra la lista de puertos del módulo.

---

## 2. Señales Internas (Línea 9)

- **`wire c1, c2, c3;`**: Declara tres cables internos (*wires*) para conectar en cadena el acarreo de salida de cada sumador con el acarreo de entrada del siguiente (*Ripple Carry*).

---

## 3. Instanciación del Bit 0 - LSB (Líneas 12-18)

- **`sum1bcc_primitive FA0 (`**: Crea la primera instancia del sumador de 1 bit y la nombra `FA0`.
- **`.a(A[0]),`**: Conecta la entrada `a` del submódulo al bit 0 de $A$.
- **`.b(B[0]),`**: Conecta la entrada `b` del submódulo al bit 0 de $B$.
- **`.cin(Cin),`**: Conecta el acarreo de entrada `cin` al puerto de entrada principal `Cin`.
- **`.sum(Sum[0]),`**: Conecta el resultado de la suma del primer bit a `Sum[0]`.
- **`.cout(c1)`**: Guarda el acarreo generado por el Bit 0 en el cable interno `c1`.

---

## 4. Instanciación del Bit 1 (Líneas 21-27)

- **`sum1bcc_primitive FA1 (`**: Crea la segunda instancia del sumador de 1 bit y la nombra `FA1`.
- **`.a(A[1]),`**: Conecta la entrada `a` del submódulo al bit 1 de $A$.
- **`.b(B[1]),`**: Conecta la entrada `b` del submódulo al bit 1 de $B$.
- **`.cin(c1),`**: Recibe el acarreo proveniente del Bit 0 a través del cable `c1`.
- **`.sum(Sum[1]),`**: Conecta el resultado de la suma al bit 1 del bus `Sum`.
- **`.cout(c2)`**: Guarda el acarreo generado por el Bit 1 en el cable interno `c2`.

---

## 5. Instanciación del Bit 2 (Líneas 30-36)

- **`sum1bcc_primitive FA2 (`**: Crea la tercera instancia del sumador de 1 bit y la nombra `FA2`.
- **`.a(A[2]),`**: Conecta la entrada `a` del submódulo al bit 2 de $A$.
- **`.b(B[2]),`**: Conecta la entrada `b` del submódulo al bit 2 de $B$.
- **`.cin(c2),`**: Recibe el acarreo proveniente del Bit 1 a través del cable `c2`.
- **`.sum(Sum[2]),`**: Conecta el resultado de la suma al bit 2 del bus `Sum`.
- **`.cout(c3)`**: Guarda el acarreo generado por el Bit 2 en el cable interno `c3`.

---

## 6. Instanciación del Bit 3 - MSB (Líneas 39-45)

- **`sum1bcc_primitive FA3 (`**: Crea la cuarta instancia del sumador de 1 bit y la nombra `FA3`.
- **`.a(A[3]),`**: Conecta la entrada `a` del submódulo al bit 3 de $A$.
- **`.b(B[3]),`**: Conecta la entrada `b` del submódulo al bit 3 de $B$.
- **`.cin(c3),`**: Recibe el acarreo proveniente del Bit 2 a través del cable `c3`.
- **`.sum(Sum[3]),`**: Conecta el resultado de la suma al bit 3 del bus `Sum`.
- **`.cout(Cout)`**: Conecta el acarreo final directamente a la salida externa `Cout`.

---

## 7. Fin del Módulo (Línea 46)

- **`endmodule`**: Concluye la descripción en Verilog del módulo `sumador4b`.
  
# 2. [Distribución pines]
La distribución de los pines del circuito se presentan a acontinuación en la figura 2.
<img width="986" height="747" alt="image" src="https://github.com/user-attachments/assets/890a7ddc-ef49-46b2-a717-4cc97dec4172" />

Fig 2. Pines circuito sumador de 4 bit



# 3. [Ejemplo]
# Ejemplo de Suma de 2 Números de 4 Bits (`sumador4b.v`)

Este documento analiza el comportamiento del hardware al sumar dos números binarios de 4 bits ($A$ y $B$) junto con un acarreo de entrada ($C_{in}$).

---

## 1. Planteamiento de la Operación

Evaluamos la suma de $9_{10}$ y $5_{10}$ sin acarreo inicial:

- **Operando A:** $9_{10} = 1001_2$ (`A[3]=1`, `A[2]=0`, `A[1]=0`, `A[0]=1`)
- **Operando B:** $5_{10} = 0101_2$ (`B[3]=0`, `B[2]=1`, `B[1]=0`, `B[0]=1`)
- **Acarreo Inicial ($C_{in}$):** `0`

---

## 2. Propagación de Acarreos en Paralelo (*Ripple Carry*)

```text
Acarreos internos:   0  0  1  0  [0] <- Cin (Entrada)
Operando A:             1  0  0  1   (9 en decimal)
Operando B:          +  0  1  0  1   (5 en decimal)
----------------------------------
Resultado:           0  1  1  1  0   (14 en decimal)
                     |  \______/
                   Cout   Sum
