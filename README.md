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
<p>
    <img src="images\3\2_imp.PNG" alt="Parser1" width="300" height="">
</p>  

#### Printer

<p>
    <img src="images\3\2_printer.PNG" alt="Parser1" width="300" height="">
</p>  

#### Interpreter

<p>
    <img src="images\3\2_interpreter.PNG" alt="Parser1" width="400" height="">
</p>  

### Definición de typeCheck

Como el caso del while do, el tipo de expression debe ser booleano.

<p>
    <img src="images\3\3.PNG" alt="Parser1" width="500" height="">
</p>  

### Definición de codegen

Se diferencia del while do en el orden de la lectura del body y expression, además de la ubicación de los labels.

<p>
    <img src="images\3\4.PNG" alt="Parser1" width="300" height="">
</p>  

## Sentencias break y continue

### Cambios en la gramática

En primer lugar, estas instrucciones podrán ser utilizadas únicamente dentro de un iterador. Además, deben poder ser utilizadas como si se tratase de un stm normal. Se incluyó una definición distinta de body y statement list, que permitirán el uso de break y continue en su interior en adición al resto de statements:

    Stm             ::= id “=” Exp |
                        “print” “(“ Exp “)” |
                        “if” Exp “then” Body [“else” Body] “endif” |
                        “while” Exp “do” LoBody “endwhile” |
                        “for” id “:” Exp “,” Exp “do” LoBody “endfor” |
                        “do” LoBody “while” Exp 
    LoBody          ::= VarDecList LoStatementList
    LoStatementList ::= Stm | “break” | “continue” (“;” Stm | “break” | “continue”)*

### Cambios en el código

#### Parser

Creación de nuevos tokens:

<p>
    <img src="images\4\2_parser_tokens.PNG" alt="Parser1" width="600" height="">
</p>  

Declaración de nuevas palabras reservadas:

<p>
    <img src="images\4\2_parser_reserved.PNG" alt="Parser1" width="300" height="">
</p>  

Se definió un nuevo atributo de la clase LoBody en el parseo de statement. Reemplaza a las llamadas a body realizadas por los iteradores:

<p>
    <img src="images\4\2_parser_parseStatement.PNG" alt="Parser1" width="450" height="">
</p>  


Parseo de LoBody:

<p>
    <img src="images\4\2_parser_parseLoBody.PNG" alt="Parser1" width="350" height="">
</p>  

Parseo de LoStmL:

<p>
    <img src="images\4\2_parser_parseLoStm.PNG" alt="Parser1" width="550" height="">
</p>  

#### Imp

Declaración de statements break y continue:

<p>
    <img src="images\4\2_imp_D_C.PNG" alt="Parser1" width="300" height="">
</p>  

Cambios en definición de iteradores:

<p>
    <img src="images\4\2_imp_While.PNG" alt="Parser1" width="300" height="">
</p>  

<p>
    <img src="images\4\2_imp_DoW.PNG" alt="Parser1" width="300" height="">
</p>  

<p>
    <img src="images\4\2_imp_For.PNG" alt="Parser1" width="450" height="">
</p>  

Definición de LoBody:

<p>
    <img src="images\4\2_imp_LoBody.PNG" alt="Parser1" width="350" height="">
</p>  

Definición de LoStmL:

<p>
    <img src="images\4\2_imp_LoStm.PNG" alt="Parser1" width="250" height="">
</p>  

#### Printer

Escritura de break y continue en el printer:

<p>
    <img src="images\4\2_printer.PNG" alt="Parser1" width="400" height="">
</p>  

#### Interpreter

No se implementó la funcionalidad del break ni del continue, pero sí fueron definidos en el interpreter:

<p>
    <img src="images\4\2_interpreter.PNG" alt="Parser1" width="400" height="">
</p>  

Implementación de LoBody en interpreter:

<p>
    <img src="images\4\2_interpreter_LoBody.PNG" alt="Parser1" width="300" height="">
</p>  

Implementación de LoStmL en interpreter:

<p>
    <img src="images\4\2_interpreter_LoStmL.PNG" alt="Parser1" width="450" height="">
</p>  

### Definición de typeCheck

Al ser statements sin contenido, la declaración del typecheck para break y continue fue vacía:

<p>
    <img src="images\4\3_B_C.PNG" alt="Parser1" width="300" height="">
</p>  

Por otro lado, los typecheck de LoBody y LoStmL fueron similares a los originales:

<p>
    <img src="images\4\3_LoBody.PNG" alt="Parser1" width="300" height="">
</p>  

<p>
    <img src="images\4\3_LoStmL.PNG" alt="Parser1" width="450" height="">
</p>  

Por último, el cambio en los typecheck de los iteradores fue de nuevo la llamada a LoBody:

<p>
    <img src="images\4\3_While.PNG" alt="Parser1" width="500" height="">
</p>  

<p>
    <img src="images\4\3_DoW.PNG" alt="Parser1" width="500" height="">
</p>  

<p>
    <img src="images\4\3_For.PNG" alt="Parser1" width="500" height="">
</p>  

### Definición de codegen

Se observa que los labels utilizados por los iteradores guardan una relación entre posición y orden de declaración:

<p>
    <img src="images\4\4_While.PNG" alt="Parser1" width="500" height="">
</p>  

<p>
    <img src="images\4\4_DoW.PNG" alt="Parser1" width="500" height="">
</p>  

<p>
    <img src="images\4\4_For.PNG" alt="Parser1" width="450" height="">
</p>  

Entonces, al encontrarse dentro de los iteradores, los operadores break y continue pueden recuperar las posiciones de la evaluación de condición y salida a través de current_label.

<p>
    <img src="images\4\4_B_C.PNG" alt="Parser1" width="350" height="">
</p>  
