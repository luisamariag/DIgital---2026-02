# 1. Código Verilog
Añadir foto
Desgloce:
## Explicación Detallada del Código: Sumador / Restador de 4 Bits (`sumador_restador4b.v`)

### 1. Definición de la Interfaz y Puertos (Líneas 1 a 7)
- **`module sumador_restador4b (`**: Declara el inicio del módulo principal denominado `sumador_restador4b`.
- **`input wire [3:0] A,`**: Bus de entrada de 4 bits para el primer operando ($A_3, A_2, A_1, A_0$).
- **`input wire [3:0] B,`**: Bus de entrada de 4 bits para el segundo operando ($B_3, B_2, B_1, B_0$).
- **`input wire M,`**: Señal de entrada de 1 bit que actúa como selector de modo ($M = 0$ para Suma, $M = 1$ para Resta).
- **`output wire [3:0] Sum,`**: Bus de salida de 4 bits que almacena el resultado de la operación.
- **`output wire Cout`**: Salida de 1 bit para el acarreo sobrante final (o indicador de préstamo en resta).
- **`);`**: Cierra la declaración de la lista de puertos.

---

### 2. Señales Internas (Líneas 8 a 10)
- **`wire [3:0] B_xor;`**: Bus interno de 4 bits para almacenar la versión invertida condicionalmente de $B$.
- **`wire c1, c2, c3;`**: Cables internos para la propagación en cadena del acarreo entre los bloques de 1 bit (*Ripple Carry*).

---

### 3. Lógica de Complemento a 1 mediante XOR (Líneas 12 a 16)
- **`assign B_xor[0] = B[0] ^ M;`**: Aplica una operación XOR entre el bit 0 de $B$ y el modo $M$.
- **`assign B_xor[1] = B[1] ^ M;`**: Aplica una operación XOR entre el bit 1 de $B$ y el modo $M$.
- **`assign B_xor[2] = B[2] ^ M;`**: Aplica una operación XOR entre el bit 2 de $B$ y el modo $M$.
- **`assign B_xor[3] = B[3] ^ M;`**: Aplica una operación XOR entre el bit 3 de $B$ y el modo $M$.

> **Funcionamiento de las compuertas XOR:**
> - Si $M = 0$ (Suma): $B \oplus 0 = B$ (los datos pasan sin alteración).
> - Si $M = 1$ (Resta): $B \oplus 1 = \bar{B}$ (invierte cada bit de $B$, obteniendo el **Complemento a 1**).

---

### 4. Instanciación del Bit 0 - LSB (Líneas 18 a 25)
- **`sum1bcc_primitive FA0 (`**: Crea la primera instancia del sumador de 1 bit con el nombre `FA0`.
- **`.a(A[0]),`**: Conecta la entrada `a` al bit 0 de $A$.
- **`.b(B_xor[0]),`**: Conecta la entrada `b` al bit 0 condicionado de $B$.
- **`.cin(M),`**: Conecta la entrada de acarreo inicial directamente a $M$. En modo resta ($M=1$), inyecta el $+1$ necesario para completar la conversión a **Complemento a 2** ($A + \bar{B} + 1$).
- **`.sum(Sum[0]),`**: Conecta el resultado del primer bit a `Sum[0]`.
- **`.cout(c1)`**: Transmite el acarreo generado al cable interno `c1`.

---

### 5. Instanciación del Bit 1 (Líneas 27 a 34)
- **`sum1bcc_primitive FA1 (`**: Crea la segunda instancia del sumador de 1 bit (`FA1`).
- **`.a(A[1]),`**: Conecta la entrada `a` al bit 1 de $A$.
- **`.b(B_xor[1]),`**: Conecta la entrada `b` al bit 1 condicionado de $B$.
- **`.cin(c1),`**: Recibe el acarreo generado por el Bit 0 a través del cable `c1`.
- **`.sum(Sum[1]),`**: Conecta el resultado al bit 1 de la salida `Sum`.
- **`.cout(c2)`**: Transmite el acarreo al cable interno `c2`.

---

### 6. Instanciación del Bit 2 (Líneas 36 a 43)
- **`sum1bcc_primitive FA2 (`**: Crea la tercera instancia del sumador de 1 bit (`FA2`).
- **`.a(A[2]),`**: Conecta la entrada `a` al bit 2 de $A$.
- **`.b(B_xor[2]),`**: Conecta la entrada `b` al bit 2 condicionado de $B$.
- **`.cin(c2),`**: Recibe el acarreo generado por el Bit 1 a través del cable `c2`.
- **`.sum(Sum[2]),`**: Conecta el resultado al bit 2 de la salida `Sum`.
- **`.cout(c3)`**: Transmite el acarreo al cable interno `c3`.

---

### 7. Instanciación del Bit 3 - MSB (Líneas 45 a 52)
- **`sum1bcc_primitive FA3 (`**: Crea la cuarta instancia del sumador de 1 bit (`FA3`).
- **`.a(A[3]),`**: Conecta la entrada `a` al bit 3 de $A$.
- **`.b(B_xor[3]),`**: Conecta la entrada `b` al bit 3 condicionado de $B$.
- **`.cin(c3),`**: Recibe el acarreo generado por el Bit 2 a través del cable `c3`.
- **`.sum(Sum[3]),`**: Conecta el resultado al bit 3 de la salida `Sum`.
- **`.cout(Cout)`**: Conecta el acarreo final directamente a la salida externa `Cout`.

### 8. Fin del Módulo (Línea 53)
- **`endmodule`**: Concluye la definición del módulo `sumador_restador4b`.

---
# 2. Distribución de pines 
La distribución de los pines se presenta en la figura 2 a continuación:
<img width="1301" height="942" alt="image" src="https://github.com/user-attachments/assets/44ecf70b-6acd-4b70-9aca-71987d277788" />
Fig 2. Distribución de pines 

# 3. Ejemplo
# Interpretación de Resultados en Resta de 4 Bits

### 1. Si el resultado es Positivo ($A \ge B$)

El resultado se muestra directamente en binario natural y el acarreo $C_{out}$ se enciende para indicar que el resultado es válido/positivo.

- **Ejemplo:** $5 - 2 = 3$
  - **Entradas:** $A = 5$ (`0101`), $B = 2$ (`0010`), $M = 1$.
  - **LEDs de resultado ($S$):** Mostrarán `0011` (número 3). Se encienden los LEDs $S[1]$ y $S[0]$.
  - **LED de Acarreo ($C_{out}$):** Se enciende (`1`). En resta por complemento a 2, $C_{out} = 1$ confirma que $A \ge B$ (no hubo necesidad de pedir prestado).

---

### 2. Si el resultado es Negativo ($A < B$)

El resultado se muestra representado en **Complemento a 2** y el acarreo $C_{out}$ se apaga (`0`), lo que te advierte que el valor representado es un número negativo.

- **Ejemplo:** $2 - 5 = -3$
  - **Entradas:** $A = 2$ (`0010`), $B = 5$ (`0101`), $M = 1$.
  - **LEDs de resultado ($S$):** Mostrarán `1101` (que es el $-3$ codificado en complemento a 2).
  - **LED de Acarreo ($C_{out}$):** Se apaga (`0`), señalando que el resultado es menor que cero.

---

## Resumen de Interpretación para las Restas

| Condición | LED $C_{out}$ | Interpretación de LEDs $S$ |
| :--- | :--- | :--- |
| **$A \ge B$** | **Encendido (1)** | El número positivo exacto en binario. |
| **$A < B$** | **Apagado (0)** | El número negativo codificado en Complemento a 2. |



> **Nota:** Para saber el valor absoluto de un resultado negativo mostrado en los LEDs, inviertes sus bits y le sumas 1. Por ejemplo, `1101` invertido es `0010`, más 1 da `0011`, es decir, el valor absoluto es 3.





