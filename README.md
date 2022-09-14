# Ensamble de instrucciones RISC-V

El proceso de ensamble consiste en convertir una instrucción en ensamblador a su correspondiente código máquina. Este proceso es sencillo porque cada instrucción en ensamblador tiene un único código máquina. Este código máquina se calcula dependiendo del tipo de instrucción RISC-V; no obstante, el procedimiento general es el siguiente.

Para ensamblar una instrucción RISC-V, deben emplearse los códigos `opcode`, `funct3`, `funct7`, así como las direcciones de los registros y el valor de las constantes inmediatas, para llenar los campos que integran los códigos máquina de las instrucciones. Estos datos se encuentran en la ISA RV32-I y en la hoja de referencia de RISC-V.

El desensamble de una instrucción se realiza separando el código máquina en campos de la instrucción. Después, la instrucción se construye a partir del valor de los campos. El tipo de instrucción se deduce a partir del valor del campo `opcode`.

## Ejemplo de ensamble de instrucciones tipo R

Estas instrucciones se caracterizan por emplear tres registros: un registro destino, denotado como `rd`, y dos operandos llamados registros fuente `rs1` y `rs2`.

Un ejemplo de instrucción tipo R es `rem s1, a7, t6`. Esta instrucción tiene por registro fuente uno a `a7` y a `t6` por registro fuente dos; su registro destino es `s1`. Los campos de la instrucción se obtienen de la hoja de referencia RISC-V y se presentan en la siguiente tabla.

| funct7    | rs2     | rs1     | funct3  | rd      | opcode    |
| --------- | ------- | ------- | ------- | ------- | --------- |
| `0000001` | `11111` | `10001` | `110`   | `01001` | `0110011` |

En hexadecimal, el código máquina de la instrucción es `0x03F8E4B3`. 

## Ejemplo de desensamble de instrucciones tipo R

Debido a que la relación entre en el código máquina de una instrucción y la instrucción RISC-V es unívoca, es posible obtener una instrucción a partir de su código máquina.

Para obtener la instrucción que corresponde al siguiente código máquina `0x00450233` es necesario calcular el campo `opcode`, el cual se encuentra en los siete bits menos significativos de la instrucción, es decir, los bits [6:0]. Los seis bits menos significativos de la instrucción `0x00450233` valen `0110011`, por lo que esta instrucción es tipo R. Una vez que el tipo de instrucción se conoce, es posible obtener el valor de sus campos, tal como se muestra en la siguiente tabla.

| funct7    | rs2     | rs1     | funct3  | rd      | opcode    |
| --------- | ------- | ------- | ------- | ------- | --------- |
| `0000000` | `00100` | `01010` | `000`   | `00100` | `0110011` |

De acuerdo con la hoja de referencia de RISC-V, la instrucción `add` tiene los valores `0000000` y `000`, por lo tanto, el código máquina corresponde a la suma. El registro destino de la suma es el `00100`, el cual corresponde al registro `tp`. Los registros fuente son `a0` y `tp` porque las direcciones de dichos registros son ocho y cuatro, respectivamente. Con base en los resultados anteriores, el código máquina `0x00450233` corresponde a la instrucción `add tp, a0, tp`.

## Instrucciones tipo I

Estas instrucciones emplean una constante inmediata como operando, un registro fuente `rs` también como operando y un registro destino `rd`. La constante inmediata se encuentra en el rango $[-2048, 2047]$. 

Un ejemplo de instrucción tipo I es `addi a0, zero, -10`, donde `a0` es el registro destino, `zero` es el registro fuente y `-10` es la constante inmediata. 

Un caso especial de las instrucciones tipo es corresponde a las instrucciones de carga `lw`, `lh` y `lb`. En estas instrucciones, el registro destino se carga con el valor apuntado por el registro fuente más una compensación, la cual corresponde a la constante inmediata.

## Instrucciones tipo S

Son tres instrucciones que permiten guardar en la memoria principal el valor de algún registro. Estas instrucciones requieren de un registro `rs` que contiene el valor a ser almacenado en la memoria. También requieren de un apuntador `rp` que contiene una dirección. Finalmente, ocupan una constante inmediata de doce bits que compensa el valor de `rp`. 

Un ejemplo de instrucción tipo S es `sb t0, -40(s0)`, en donde t0 es `rs`, `s0` es `rp` y `-40` es la compensación (*offset*).

## Instrucciones tipo B

Estas instrucciones implementan los saltos condicionales. Sus operandos son dos registros fuente y una constante inmediata de 13 bits cuyo bit menos significativo no se almacena porque su valor debe ser igual a cero. Esto implica que la constante inmediata debe ser par. 

En otras arquitecturas, como la MIPS, las constantes son múltiplo de cuatro dado que las instrucciones miden cuatro bytes; sin embargo, en la arquitectura RISC-V existen instrucciones comprimidas de 2 bytes, de ahí que por generalidad se maneje que las direcciones de las instrucciones sean múltiplo de dos.

Un ejemplo de instrucción tipo B es `bne t0, t1, L1`, donde `t0` y `t1` son los registros fuente y L1 es un constante que se calcula contando el número de instrucción que hay entre la instrucción de salto y la etiqueta `L1`. Dicho número se multiplica por cuatro. El resultado de la multiplicación se multiplica por `-1` cuando la etiqueta está arriba de la instrucción de salto.

## Instrucciones tipo U

Son dos instrucciones que se utilizan para cargar una constante en los 20 bits más significativos de un registro o del PC (*program counter*). La instrucción requiere un registro destino `rd` y la constante inmediata de 20 bits `imm20`.

Un ejemplo de instrucción tipo U es `lui t0, 0x8FFFAA`, en donde `t0` es el registro en donde se cargará la constante `0x8FFFAA000`. Nota que los doce bits menos significativos quedan cero.

## Instrucción tipo J

Solamente existe una instrucción tipo J y ésta es `jal`. La función requiere de un registro destino `rd` en donde se guardará la dirección de retorno de la función y una constante inmediata de 21 bits cuyo bit menos significativo vale cero, por lo que este bit no se almacena en el código máquina de la instrucción.

La instrucción `jal ra function` es un ejemplo de la instrucción J. El registro destino `ra` almacena la dirección de retorno de la función `function`, la cual equivale al valor que actualmente el contador de programa PC tiene más cuatro bytes. El valor de la constante inmediata se calcula restando la dirección actual del PC menos la dirección de la primera instrucción de la función a llamar, `function` bajo este ejemplo.
