# Proyecto final - Compiladores

## Integrantes

- Mariajulia Romani Tafur
- Jaime Ramos Talla

## Comentarios

El cambio realizado para incluir comentarios de una sola línea en cualquier punto del programa se realizó en el scanner. En el método `nextToken()` de la clase `Scanner` se agregó el siguiente código:

```cpp
    while(c == '/'){
        c = nextChar();
        if (c == '/'){
            while (c!='\n') c=nextChar();
            while (c == ' ' || c == '\t'  || c == '\n') c = nextChar();
        }
        else{
            rollBack();
            break;
        }
    }
```

El código anterior permite que, al encontrar un '/', se verifique si el siguiente caracter es también un '/'. En caso de serlo, se lee el resto de la línea y se descarta. En caso contrario, se devuelve el caracter leído y se continúa con el análisis.

## Generación de código I

Para implementar la generación de código objeto para constantes booleanas, los operadores and/or y la versión de for loops considerada por el intérprete que solo considera for-loops con valores ascendentes se modificó el código de la siguiente manera:

Primero, para la generación de código objeto para constantes booleanas se agregó el siguiente código en el método `visit(BoolConst* e)` de la clase `ImpCodeGen`:

```cpp
    int ImpCodeGen::visit(BoolConstExp* e) {
        codegen(nolabel,"push ",e->b ? 1 : 0);
        return 0;
    }
```

Luego, para la generación de código objeto para los operadores and/or se agregó el siguiente código en el método `visit(BinaryExp* e)` de la clase `ImpCodeGen`:

```cpp
    int ImpCodeGen::visit(BinaryExp* e) {
        e->left->accept(this);
        e->right->accept(this);
        string op = "";
        switch(e->op) {
        case PLUS: op =  "add"; break;
        case MINUS: op = "sub"; break;
        case MULT:  op = "mul"; break;
        case DIV:  op = "div"; break;
        case LT:  op = "lt"; break;
        case LTEQ: op = "le"; break;
        case EQ:  op = "eq"; break;
        case AND:  op = "and"; break;
        case OR:  op = "or"; break;
        default: cout << Exp::binopToString(e->op) << " not implemented" << endl;
        }
        codegen(nolabel, op);
        return 0;
    }
```

Finalmente, para la generación de código objeto para la versión de for loops considerada por el intérprete que solo considera for-loops con valores ascendentes se agregó el siguiente código en el método `visit(ForStatement* s)` de la clase `ImpCodeGen`:

```cpp
    int ImpCodeGen::visit(ForStatement* s) {
    string l1 = next_label();         
    string l2 = next_label();         

    s->e1->accept(this);
    codegen(nolabel, "store", direcciones.lookup(s->id));
    codegen(l1, "skip");
    s->e2->accept(this);
    codegen(nolabel, "load", direcciones.lookup(s->id));
    codegen(nolabel, "lt");
    codegen(nolabel, "jmpz", l2);
    s->body->accept(this);               
    codegen(nolabel, "load", direcciones.lookup(s->id));
    codegen(nolabel, "push", 1);
    codegen(nolabel, "add");
    codegen(nolabel, "store", direcciones.lookup(s->id));
    codegen(nolabel, "goto", l1);
    codegen(l2, "skip");

    return 0;
    }
```

## Sentencia do-while


Para implementar la sentencia do-while se agregó la siguiente definición en la gramática:

```
    Stm ::= id “=” Exp |
            “print” “(“ Exp “)” |
            “if” Exp “then” Body [“else” Body] “endif” |
            “while” Exp “do” Body “endwhile” |
            “for” id “:” Exp “,” Exp “do” Body “endfor” |
            “do” Body “while” Exp
```

Luego, añadimos en el metodo `parseStatement()` de la clase `Parser` la definición de la sentencia do-while:

```cpp
    Statement* Parser::parseStatement() {
        ...
        else if (match(Token::DO)) {
            tb = parseBody();
            if (!match(Token::WHILE)) 
                parserError("Esperaba 'while'");
            e = parseExp();
            s = new DoWhileStatement(tb, e);
        } 
        ...
    }
```

Además, se agregó la definición de do-while como un nuevo tipo de statement en la clase `Imp`:

```cpp
    class DoWhileStatement : public Stm {     
        public:
        Exp* cond;
        Body* body;                         
        DoWhileStatement(Body* b, Exp* c);
        int accept(ImpVisitor* v);
        void accept(TypeVisitor* v);
        ~DoWhileStatement();
    };
```

Se modificó la clase `ImpPrinter` para que imprima la sentencia do-while:

```cpp
    int ImpPrinter::visit(DoWhileStatement* s) {
        cout << "do" << endl;
        s->body->accept(this);
        cout << "while (";
        s->cond->accept(this);
        cout << ")";
        return 0;
    }
```

También se modificó la clase `ImpInterpreter` para que interprete la sentencia do-while:

```cpp
    int ImpInterpreter::visit(DoWhileStatement* s) {
        do {
            s->body->accept(this);
        } while (s->cond->accept(this));
        return 0;
    }
```

Asimismo, se modificó la clase `ImpTypeChecker` para que verifique la sentencia do-while:

```cpp
    void ImpTypeChecker::visit(DoWhileStatement* s) {
        s->body->accept(this);
        if(!s->cond->accept(this).match(booltype)){
            cout << "Condicional en dowhile debe ser: " << booltype << endl;
            exit(0);
        }
        return;
    }
```
Puede observarse que como el caso del while, el tipo de expresión debe ser booleano.

Finalmente, se modificó la clase `ImpCodeGen` para que genere código objeto para la sentencia do-while:

```cpp
    int ImpCodeGen::visit(DoWhileStatement* s) {
        string l1 = next_label();
        string l2 = next_label();
        string l3 = next_label();

        codegen(l1, "skip");
        s->body->accept(this);
        s->cond->accept(this);
        codegen(l2, "skip");
        codegen(nolabel, "jmpn", l1);
        codegen(l3, "skip");

        return 0;
    }
```
Note que la implementación de la sentencia do-while es similar a la del while, con la diferencia en el orden de la lectura del body y la expresión, además de la ubicación de los labels.


## Sentencias break y continue

En primer lugar, estas instrucciones podrán ser utilizadas únicamente dentro de un iterador. Además, deben poder ser utilizadas como si se tratase de un statement normal. Se incluyó una definición distinta de body y statement list, que permitirán el uso de break y continue en su interior en adición al resto de statements.

Para implementar break y continue se agregó la siguiente definición en la gramática:

```
    Stm             ::= id “=” Exp |
                        “print” “(“ Exp “)” |
                        “if” Exp “then” Body [“else” Body] “endif” |
                        “while” Exp “do” LoBody “endwhile” |
                        “for” id “:” Exp “,” Exp “do” LoBody “endfor” |
                        “do” LoBody “while” Exp 
    LoBody          ::= VarDecList LoStatementList
    LoStatementList ::= Stm | “break” | “continue” (“;” Stm | “break” | “continue”)*
```

Empezamos añadiendo nuevos tokens en la clase `Token`:

```cpp
    const char* Token::token_names[37] = {
    "LPAREN" , "RPAREN", "PLUS", "MINUS", "MULT","DIV","EXP","LT","LTEQ","EQ",
    "NUM", "ID", "PRINT", "SEMICOLON", "COMMA", "ASSIGN", "CONDEXP", "IF", "THEN", "ELSE", "ENDIF", "WHILE", "DO",
    "ENDWHILE", "ERR", "END", "VAR" , "NOT", "TRUE", "FALSE", "AND", "OR"
    "FOR", "COLON" , "ENDFOR", "BREAK", "CONTINUE"
    };
```

Continuamos declarando las palabras reservadas en la clase `Scanner`:

```cpp
    Scanner::Scanner(string s):input(s),first(0),current(0) {
        ...
        reserved["break"] = Token::BREAK;
        reserved["continue"] = Token::CONTINUE;
        ...
    }
```
Se definió un nuevo atributo de la clase LoBody en el parseo de statement. Reemplaza a las llamadas a body realizadas por los iteradores:

```cpp
    Stm* Parser::parseStatement() {
        Stm* s = NULL;
        Exp* e;
        Body *tb, *fb;
        LoBody* lb;  
        ...   
    }                       
```
Luego, parseamos `LoBody`:

```cpp
    LoBody* Parser::parseLoBody() {
        VarDecList* vdl = parseVarDecList();
        LoStatementList* sl = parseLoStatementList();
        return new LoBody(vdl, sl);
    }
```

Parseamos `LoStmL`:

```cpp
    LoStatementList* Parser::parseLoStatementList() {
        LoStatementList* p = new LoStatementList();
        Stm* s;
        if (match(Token::BREAK)){
            s = new BreakStatement();
        } else if(match(Token::CONTINUE)){
            s = new ContinueStatement();
        } else{
            s = parseStatement();
        }
        p->add(s);
        while(match(Token::SEMICOLON)) {
            if (match(Token::BREAK)) {
                s = new BreakStatement();
                p->add(s);
            } else if(match(Token::CONTINUE)){
                s = new ContinueStatement();
                p->add(s);
            } else{
                p->add(parseStatement());
            }
        }
        return p;
    }
```

Seguimos con las modificaciones en la clase `Imp` añadiendo la definición de `LoBody` y `LoStatementList`:

```cpp
    class LoBody{     
        public:
        VarDecList* var_decs;
        LoStatementList* slist;
        LoBody(VarDecList* vdl, LoStatementList* sl);
        int accept(ImpVisitor* v);
        void accept(TypeVisitor* v);
        ~LoBody();
    };

    class LoStatementList {     
        public:
        list<Stm*> slist;
        LoStatementList();
        void add(Stm* s);
        int accept(ImpVisitor* v);
        void accept(TypeVisitor* v);
        ~LoStatementList();
    };
```

También, declaramos los statements de `Break` y `Continue`:

```cpp
    class BreakStatement : public Stm {     
        public:
        BreakStatement();
        int accept(ImpVisitor* v);
        void accept(TypeVisitor* v);
        ~BreakStatement();
    };

    class ContinueStatement : public Stm {     
        public:
        ContinueStatement();
        int accept(ImpVisitor* v);
        void accept(TypeVisitor* v);
        ~ContinueStatement();
    };
```

Realizamos cambios en la definición de iteradores para que utilicen `LoBody`:

```cpp
    class WhileStatement : public Stm {     
        public:
        Exp* cond;
        LoBody* lobody;                         
        WhileStatement(Exp* c, LoBody* lb);
        int accept(ImpVisitor* v);
        void accept(TypeVisitor* v);
        ~WhileStatement();
    };

    class DoWhileStatement : public Stm {     
        public:
        Exp* cond;
        LoBody* lobody;                         
        DoWhileStatement(LoBody* lb, Exp* c);
        int accept(ImpVisitor* v);
        void accept(TypeVisitor* v);
        ~DoWhileStatement();
    };

    class ForStatement : public Stm {     
        public:
        string id;
        Exp* e1;
        Exp* e2;
        LoBody* lobody;                         
        ForStatement(string i, Exp* e1, Exp* e2, LoBody* lb);
        int accept(ImpVisitor* v);
        void accept(TypeVisitor* v);
        ~ForStatement();
    };
```
Asimismo, se modificó la clase `ImpPrinter` para que imprima los statements de `Break` y `Continue`:

```cpp
    int ImpPrinter::visit(BreakStatement* s) {
        cout << "break";
        return 0;
    }

    int ImpPrinter::visit(ContinueStatement* s) {
        cout << "continue";
        return 0;
    }
```

En la clase `ImpInterpreter` no se implementó la funcionalidad de `Break` y `Continue`, pero sí fueron definidos:

```cpp
    int ImpInterpreter::visit(BreakStatement* s) {
        return 0;
    }

    int ImpInterpreter::visit(ContinueStatement* s) {
        return 0;
    }
```

En dicha clase se implementó la funcionalidad de `LoBody` y `LoStatementList`:

```cpp
    int ImpInterpreter::visit(LoBody* b) {
        env.add_level();
        b->var_decs->accept(this);
        b->slist->accept(this);
        env.remove_level();
        return 0;
    }

    int ImpInterpreter::visit(LoStatementList* s) {
        list<Stm*>::iterator it;
        for (it = s->slist.begin(); it != s->slist.end(); it++) {
            (*it)->accept(this);
        }
    }
```

En la clase `ImpTypeChecker`, al ser statements sin contenido, la declaración del typecheck para break y continue fue vacía:

```cpp
    void ImpTypeChecker::visit(BreakStatement* s) {
        return;
    }

    void ImpTypeChecker::visit(ContinueStatement* s) {
        return;
    }
```

Por otro lado, los typecheck de `LoBody` y `LoStmL` fueron similares a los originales:

```cpp
    void ImpTypeChecker::visit(LoBody* b) {
        env.add_level();
        b->var_decs->accept(this);
        b->slist->accept(this);
        env.remove_level();
        return;
    }

    void ImpTypeChecker::visit(LoStatementList* s) {
        list<Stm*>::iterator it;
        for (it = s->slist.begin(); it != s->slist.end(); it++) {
            (*it)->accept(this);
        }
    }
```

Para terminar con las modificaciones en la clase `ImpTypeChecker`, se modificaron los typecheck de los iteradores para que utilicen `LoBody`:

```cpp
    void ImpTypeChecker::visit(WhileStatement* s) {
        if(!s->cond->accept(this).match(booltype)){
            cout << "Condicional en while debe ser: " << booltype << endl;
            exit(0);
        }
        s->lobody->accept(this);
        return;
    }

    void ImpTypeChecker::visit(DoWhileStatement* s) {
        s->lobody->accept(this);
        if(!s->cond->accept(this).match(booltype)){
            cout << "Condicional en dowhile debe ser: " << booltype << endl;
            exit(0);
        }
        return;
    }

    void ImpTypeChecker::visit(ForStatement* s) {
        ImpType t1 = s->e1->accept(this);
        ImpType t2 = s->e2->accept(this);
        if(!t1.match(inttype) || !t2.match(inttype)){
            cout << "Expresiones en for deben ser: " << inttype << endl;
            exit(0);
        }
        env.add_level();
        env.add_var(s->id, inttype);
        s->lobody->accept(this);
        env.remove_level();
        return;
    }
```
Se modificó la clase `ImpCodeGen` para generamos código objeto para cada iterador:

```cpp
    int ImpCodeGen::visit(WhileStatement* s) {
        string l1 = next_label();
        string l2 = next_label();

        codegen(l1,"skip");
        s->cond->accept(this);
        codegen(nolabel,"jmpz",l2);
        s->lobody->accept(this);
        codegen(nolabel,"goto",l1);
        codegen(l2,"skip");

        return 0;
    }

    int ImpCodeGen::visit(DoWhileStatement* s) {
        string l1 = next_label();
        string l2 = next_label();
        string l3 = next_label();

        codegen(l1, "skip");
        s->lobody->accept(this);
        codegen(l2, "skip");
        s->cond->accept(this);
        codegen(nolabel, "jmpn", l1);
        codegen(l3, "skip");

        return 0;
    }

    int ImpCodeGen::visit(ForStatement* s) {
        string l1 = next_label();
        string l2 = next_label();

        s->e1->accept(this);
        codegen(nolabel, "store", direcciones.lookup(s->id));
        codegen(l1, "skip");
        s->e2->accept(this);
        codegen(nolabel, "load", direcciones.lookup(s->id));
        codegen(nolabel, "lt");
        codegen(nolabel, "jmpz", l2);
        s->lobody->accept(this);
        codegen(nolabel, "load", direcciones.lookup(s->id));
        codegen(nolabel, "push", 1);
        codegen(nolabel, "add");
        codegen(nolabel, "store", direcciones.lookup(s->id));
        codegen(nolabel, "goto", l1);
        codegen(l2, "skip");

        return 0;
    }
```
Observamos que los labels utilizados por los iteradores guardan una relación entre posición y orden de declaración. Entonces, al encontrarse dentro de los iteradores, los operadores break y continue pueden recuperar las posiciones de la evaluación de condición y salida a través de current_label.

```cpp
    int ImpCodeGen::visit(BreakStatement* s) {
        auto obj = "L" + to_string(current_label-1);
        codegen(nolabel, "goto", obj);
        return 0;
    }

    int ImpCodeGen::visit(ContinueStatement* s) {
        auto obj = "L" + to_string(current_label-2);
        codegen(nolabel, "goto", obj);
        return 0;
    }
```