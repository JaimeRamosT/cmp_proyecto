# Proyecto final - Compiladores

## Integrantes

- Romina Romani
- Jaime Ramos

## Comentarios

## Generación de código I

## Sentencia do-while

### Cambios en la gramática

El cambio realizado consistió en una adición en la gramática Stm, que se observa a continuación:

    Stm ::= id “=” Exp |
            “print” “(“ Exp “)” |
            “if” Exp “then” Body [“else” Body] “endif” |
            “while” Exp “do” Body “endwhile” |
            “for” id “:” Exp “,” Exp “do” Body “endfor” |
            “do” Body “while” Exp

### Cambios en el código

#### Parser

Adición de la definición en la gramática:
<p>
    <img src="images\3\2_parser.PNG" alt="Parser1" width="300" height="">
</p>  

#### Imp
Definición de dowhile como nuevo statement:
<p align="center">
    <img src="images\3\2_imp.PNG" alt="Parser1" width="300" height="">
</p>  

#### Printer

<p align="center">
    <img src="images\3\2_printer.PNG" alt="Parser1" width="300" height="">
</p>  

#### Interpreter

<p align="center">
    <img src="images\3\2_interpreter.PNG" alt="Parser1" width="300" height="">
</p>  

### Definición de typeCheck

### Definición de codegen

## Sentencias break y continue

### Cambios en la gramática

### Cambios en el código

### Definición de typeCheck

### Definición de codegen
