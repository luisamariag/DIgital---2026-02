# Laboratorio 4: Cerradura Electrónica Digital con FPGA Cyclone IV

Diseño, simulación e implementación en hardware de una cerradura electrónica digital en **Verilog HDL** sobre la tarjeta **Intel/Altera Cyclone IV (EP4CE10E22C8)**. El sistema integra generación de señales PWM para servomotor, escaneo de teclado matricial $4\times4$, registro de desplazamiento de 16 bits y una Máquina de Estados Finitos (FSM) de Moore.

---

## 📁 Estructura del Repositorio

```text
Lab_4/
├── src/                  # Módulos sintetizables en Verilog HDL
│   ├── cerradura_top.v   # Módulo Top-Level integrador
│   ├── fsm_cerradura.v   # Máquina de estados finitos (Moore)
│   ├── pwm_servo.v       # Generador PWM de 50 Hz para servomotor
│   ├── registro_pass.v   # Registro de desplazamiento de 4 dígitos BCD
│   └── teclado.v         # Escáner y decodificador de teclado matricial 4x4
├── testbench/            # Bancos de prueba para simulación funcional
│   └── tb_cerradura.v    # Testbench del sistema completo
├── sim/                  # Archivos de ondas y capturas de GTKWave
│   ├── cerradura.vcd     # Trazas de simulación
│   ├── sim_pwm.png       # Captura módulo PWM
│   ├── sim_teclado.png   # Captura módulo Teclado
│   ├── sim_registro.png  # Captura módulo Registro
│   ├── sim_fsm.png       # Captura módulo FSM
│   └── sim_top.png       # Captura sistema completo
└── README.md             # Documentación del proyecto
```

---

## 🛠️ Descripción del Sistema y Módulos

El sistema opera con un reloj base de **$50\text{ MHz}$** y permite alternar mediante el interruptor `modo` entre control manual por switches (`modo = 0`) y control automático por contraseña (`modo = 1`, clave por defecto `1234`):

1. **`pwm_servo.v` (Parte 1 - Control del Actuador):** Genera una señal periódica de $50\text{ Hz}$ ($20\text{ ms}$). Convierte una entrada de 4 bits (`pos`) en un ancho de pulso en alto entre $1\text{ ms}$ ($0^\circ$, puerta cerrada) y $2\text{ ms}$ ($180^\circ$, puerta abierta).
2. **`teclado.v` (Parte 2 - Interfaz Matricial):** Realiza un barrido secuencial activo en bajo sobre las 4 filas (`rows[3:0]`) y lee las 4 columnas (`cols[3:0]`) con resistencias *Weak Pull-Up*, aplicando filtrado antirrebote (*debounce*) y entregando el código BCD (`key_code`) junto con un pulso `key_valid` de un ciclo de reloj.
3. **`registro_pass.v` (Parte 3 - Memoria de Contraseña):** Almacena hasta 4 dígitos numéricos (`0`–`9`) mediante desplazamiento de 4 bits (`{pass_in[11:0], key_code}`). Activa la bandera `completo = 1` al cuarto dígito y reinicia la memoria cuando recibe `borrar = 1`.
4. **`fsm_cerradura.v` (Parte 3 - Control Principal):** FSM de Moore con 5 estados (`S_IDLE`, `S_INPUT`, `S_VERIFY`, `S_OPEN`, `S_ERROR`). Valida la clave ingresada frente a `PASSWORD`, temporiza $5\text{ s}$ de apertura (`T_ABIERTO_MS`) y $2\text{ s}$ de error (`T_ERROR_MS`), y usa la tecla `*` (`4'hE`) para cancelar digitación o cerrar manualmente.
5. **`cerradura_top.v` (Integrador Superior):** Interconecta todos los submódulos, multiplexa el control manual/automático y adapta la polaridad del hardware *Active-Low* de la placa (reset e indicadores LED).

---

## 💻 Comandos para Simulación

Desde la raíz del proyecto (`Lab_4/`), ejecute los siguientes comandos con **Icarus Verilog** y **GTKWave**:

```bash
# 1. Compilar los módulos fuente y el testbench
iverilog -o sim/sim.vvp testbench/tb_cerradura.v src/*.v

# 2. Ejecutar la simulación y generar el archivo de ondas (.vcd)
vvp sim/sim.vvp

# 3. Abrir las formas de onda en GTKWave
gtkwave cerradura.vcd
```

---

## 📜 Código HDL (`src/`)

### Módulo Superior (`src/cerradura_top.v`)

```verilog
module cerradura_top #(
    parameter integer CLK_FREQ     = 50_000_000,   // Hz, oscilador de la tarjeta
    parameter [15:0]  PASSWORD     = 16'h1234,     // Clave en BCD
    parameter integer T_ABIERTO_MS = 5000,
    parameter integer T_ERROR_MS   = 2000
)(
    input  wire       clk,
    input  wire       rst,          // Reset desde la placa (Active-Low)
    input  wire       modo,         // 0: servo manual, 1: cerradura FSM
    input  wire [3:0] sw,           // Posición manual del servo
    input  wire [3:0] cols,         // Columnas del teclado (con Weak Pull-Up)
    output wire [3:0] rows,         // Filas del teclado
    output wire [3:0] leds,         // Código BCD de la última tecla
    output wire       led_abierta,
    output wire       led_error,
    output wire       servo_pwm
);

    // Adaptación de reset Active-Low de la tarjeta a lógica positiva interna
    wire rst_int = ~rst;

    // Señales de interconexión interna
    wire [3:0]  key_code;
    wire        key_valid;
    wire [15:0] pass_in;
    wire        completo, borrar, habilitar;
    wire [3:0]  pos_fsm, pos_servo;
    wire        led_abierta_fsm;
    wire        led_error_fsm;

    // Interfaz con teclado matricial 4x4
    teclado #(.CLK_FREQ(CLK_FREQ)) u_teclado (
        .clk(clk), .rst(rst_int), .cols(cols), .rows(rows),
        .key_code(key_code), .key_valid(key_valid)
    );

    // Registro de desplazamiento + contador de dígitos
    registro_pass u_registro (
        .clk(clk), .rst(rst_int), .borrar(borrar), .habilitar(habilitar),
        .key_code(key_code), .key_valid(key_valid),
        .pass_in(pass_in), .completo(completo)
    );

    // Máquina de estados finitos de la cerradura
    fsm_cerradura #(
        .CLK_FREQ(CLK_FREQ), .PASSWORD(PASSWORD),
        .T_ABIERTO_MS(T_ABIERTO_MS), .T_ERROR_MS(T_ERROR_MS)
    ) u_fsm (
        .clk(clk), .rst(rst_int), .key_code(key_code), .key_valid(key_valid),
        .pass_in(pass_in), .completo(completo),
        .borrar(borrar), .habilitar(habilitar),
        .pos(pos_fsm),
        .led_abierta(led_abierta_fsm),
        .led_error(led_error_fsm)
    );

    // Selector de modo: 0 = manual (switches), 1 = cerradura (FSM)
    assign pos_servo = modo ? pos_fsm : sw;

    // Generador PWM del servomotor (50 Hz)
    pwm_servo #(.CLK_FREQ(CLK_FREQ)) u_servo (
        .clk(clk), .rst(rst_int), .pos(pos_servo), .servo_pwm(servo_pwm)
    );

    // Inversión de salidas para LEDs Active-Low de la tarjeta
    assign led_abierta = ~led_abierta_fsm;
    assign led_error   = ~led_error_fsm;
    assign leds        = ~key_code;

endmodule
```

*(Nota: Agregue aquí los bloques ` ```verilog ` de `pwm_servo.v`, `teclado.v`, `registro_pass.v` y `fsm_cerradura.v` correspondientes a sus archivos en `src/`).*

---

## 📊 Imágenes de Simulación (`sim/`)

### 1. Módulo PWM (`pwm_servo.v`)
![Simulación PWM](sim/sim_pwm.png)

### 2. Escáner de Teclado (`teclado.v`)
![Simulación Teclado](sim/sim_teclado.png)

### 3. Registro de Contraseña (`registro_pass.v`)
![Simulación Registro](sim/sim_registro.png)

### 4. Máquina de Estados (`fsm_cerradura.v`)
![Simulación FSM](sim/sim_fsm.png)

### 5. Sistema Completo (`cerradura_top.v`)
![Simulación Sistema Completo](sim/sim_top.png)

---

## ⚙️ Asignación de Pines en Hardware (Cyclone IV EP4CE10E22C8)

| Señal | Pin FPGA | Estándar I/O / Configuración | Función |
| :--- | :--- | :--- | :--- |
| `clk` | `PIN_23` | `3.3-V LVTTL` | Reloj principal de $50\text{ MHz}$ |
| `rst` | `PIN_58` | `3.3-V LVTTL` | Reset global (Abajo = activo en tarjeta) |
| `modo` | `PIN_68` | `3.3-V LVTTL` | Arriba (`0`): Manual / Abajo (`1`): Cerradura |
| `sw[3:0]` | `PIN_64..67` | `3.3-V LVTTL` | Control manual de posición ($0^\circ$ a $180^\circ$) |
| `rows[3:0]` | `PIN_83, 106, 77, 76` | `3.3-V LVTTL` | Filas 1–4 del teclado matricial |
| `cols[3:0]` | `PIN_51, 50, 46, 43` | `3.3-V LVTTL` + **Weak Pull-Up On** | Columnas 1–4 del teclado matricial |
| `servo_pwm` | `PIN_39` | `2.5 V` (Banco 3) | Señal PWM hacia servomotor SG90 (+5 V ext.) |
| `led_abierta` | `PIN_75` | `3.3-V LVTTL` | Indicador de apertura ($5\text{ s}$) |
| `led_error` | `PIN_77` | `3.3-V LVTTL` | Indicador de clave errónea ($2\text{ s}$) |
| `leds[3:0]` | `PIN_70, 71, 69, 65` | `3.3-V LVTTL` | Última tecla decodificada en BCD |
