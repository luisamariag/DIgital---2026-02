# Parte 1: Sumador de 1 bit

Contenido:

## 1. [Código]
El código en Verilog que se utilizo para el circuito sumador de 1 bit se presenta a continuación en la figura 1.

<img width="965" height="621" alt="image" src="https://github.com/user-attachments/assets/def14b62-f783-4502-9d61-805c7d734189" />

Fig 1. Código verilog sumador 1 bit

Donde:
- module sumador_1bit( ... );: Define la estructura del bloque lógico del sumador de 1 bit.
- input A, input B, input Cin,: Declara las tres entradas del bloque: los bits a sumar (A y B) y el acarreo proveniente del bit anterior (Cin).
- output Sum, output Cout: Declara las dos salidas: el resultado de la suma (Sum) y el acarreo generado para la siguiente etapa (Cout).
- assign Sum = A ^ B ^ Cin;: Implementa la lógica de la suma utilizando dos compuertas XOR. La salida será 1 cuando un número impar de entradas sea 1.
- assign Cout = (A & B) | (Cin & (A ^ B));: Implementa la lógica del acarreo de salida. Genera un acarreo 1 si ambos bits A y B son 1, o si el acarreo previo Cin es 1 y uno de los operandos es 1
  
## 2. [Distribución pines]
La distribución de los pines del circuito se presentan a acontinuación en la figura 2.

<img width="1437" height="941" alt="image" src="https://github.com/user-attachments/assets/edfb75a3-4be9-4417-9a9b-7458cfac9c73" />


Fig 2. Pines circuito sumador de 1 bit

## 3. [Tabla de verdad]

La tabla de verdad del circuito se presenta en la figura 3 a continuación.

<img width="331" height="450" alt="image" src="https://github.com/user-attachments/assets/c710ffba-02ce-42d7-8de8-6944d0833475" />

Fig 3. Tabla de verdad sumador de 1 bit


# 4. [Ejemplo]

Por último, en la imagen 4 se muestra un ejemplo del funcionamiento del circuito.

<img width="180" height="177" alt="image" src="https://github.com/user-attachments/assets/e650bde1-68f9-4b15-a765-219c4b174ab8" />

Fig 4. Funcionamiento del circuito

Tal y como se observa en el ejemplo del punto 4; 0+0=0 donde tanto Sum como Cout son 0 ; 0+1=1/1+0=1 donde 1 es Sum y 0 es el Cout ; 1+1=10 donde 0 es Sum y 1 es Cout

    
