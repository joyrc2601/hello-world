# Welcome to GitHub

Welcome to GitHub—where millions of developers work together on software. Ready to get started? Let’s learn how this all works by building and publishing your first GitHub Pages website!



Getting started is the hardest part. If there’s anything you’d like to know as you get started with GitHub, try searching [GitHub Help](https://help.github.com). Our documentation has tutorials on everything from changing your repository settings to configuring GitHub from your command line.

%{
    #include <stdio.h>
    #include <string.h>
    #include <stdbool.h>

    extern yylineno;

    struct variable {
        char name [100];
        bool is_struct;
    }var_list [100][100];

    int variable_size[100] = {0};

    struct structure {
        char name[100];
        struct variable v [5];
        int var_list_len;
    } struct_list [100][100];

    bool err = false;

    int struct_size[100] = {0};
    int scope = 0;
    int curr_struct = -1;

    char keywords[13][100] = {"if", "else", "for", "while", "switch", "break", "do", "include", "main", "return", "struct"};
    int keywords_size = 11;

    bool is_a_keyword (char id[]) {

        for (int i = 0; i < keywords_size; i++) {
            if (strcmp (id, keywords[i]) == 0)
                return true;
        }

        return false;
    }

    bool is_in_list (char id[], int flag) {

        for (int i = scope; i >=0; i--) {
            for (int j = 0; j < variable_size[i]; j++)
                if (strcmp (id, var_list[i][j].name) == 0)
                    return true;
            if (flag == 0)
                break;
        }

        for (int i = scope; i >=0; i--) {
            for (int j = 0; j < struct_size[i]; j++)
                if (strcmp (id, struct_list[i][j].name) == 0)
                    return true;
            if (flag == 0)
                break;
        }

        return false;
    }

    void add_to_variable_list (char id[], bool is_struct) {
        if (scope == 100){
            printf ("Too many scopes !!!\n");
            return;
        }

        if (variable_size[scope] == 100) {
             printf ("Too many declarations !!!\n");
            return;
        }

        strcpy (var_list[scope][variable_size[scope]].name, id);
        var_list[scope][variable_size[scope]++].is_struct = is_struct;
    }

    void check (char id[], int flag, bool is_struct) {

        // printf ("%d %s %d %d\n", yylineno, id, flag, is_struct);

        if (is_a_keyword (id)) {
            printf ("line %3d: '%s' is a keyword\n", yylineno, id);
            return;
        }

        bool ans = is_in_list (id, flag);

        if (!flag && ans) {
            printf ("line %3d: Variable '%s' is already declared\n", yylineno, id);
            return;
        }

        if (flag){
            if (!ans) 
                printf ("line %3d: Variable '%s' is not declared\n", yylineno, id);
            
            return;
        }

        add_to_variable_list (id, is_struct);

    }

    void add_to_struct_list (char id[]) {
        if (scope == 100){
            printf ("Too many scopes !!!\n");
            return;
        }

        if (struct_size[scope] == 100) {
             printf ("Too many declarations !!!\n");
            return;
        }

        strcpy (struct_list[scope][struct_size[scope]++].name, id);
        curr_struct++;
    }

    void struct_check (char id[], int flag) {

        if (is_a_keyword (id)) {
            printf ("line %3d: '%s' is a keyword\n", yylineno, id);
            return;
        }

        bool ans = is_in_list (id, flag);

        if (!flag && ans) {
            printf ("line %3d: struct '%s' is already declared\n", yylineno, id);
            return;
        }

        if (flag){
            if (!ans) 
                printf ("line %3d: struct '%s' is not declared\n", yylineno, id);
            return;
        }

        add_to_struct_list (id);
    }

    void check_struct_var (char id[]) {

        for (int i = scope; i >=0; i--) {
            for (int j = 0; j < variable_size[i]; j++)
                if (strcmp (id, var_list[i][j].name) == 0 && var_list[i][j].is_struct)
                    return;
        }

        printf ("line %3d: Variable '%s' is not declared\n", yylineno, id);
    }
%}

%union {
	char *f;
}

%token HEADER MACRO TYPE STRUCT IF ELIF ELSE FOR WHILE DO BREAK CONT EXIT MAIN RETURN OCB CCB ORB CRB OSB CSB COMMA SM DOT RELOP EQ UNOP OP NUM ID
%type <f> ID_LIST
//%type <f> ID
%type <f> USE_ID
%type <f> DEC_ID
%type <f> STRUCT_DEC_ID
%type <f> STRUCT_USE_ID

%start S

%%

S :     HEADERS MACROS STRUCT_DEC MAIN_FUNC                                                {if (!err) printf ("VALID SYNTAX\n\n");}
        ;

HEADERS :   HEADER HEADERS              
            |                           
            ;

MACROS :    MACRO MACROS                
            |                           
            ;

STRUCT_DEC :    STRUCT STRUCT_DEC_ID OCB DECLARATION2 CCB SM STRUCT_DEC
                | STRUCT STRUCT_DEC_ID OCB DECLARATION2 CCB NO_SM STRUCT_DEC      
                | STRUCT STRUCT_DEC_ID NO_OCB DECLARATION2 CCB SM STRUCT_DEC
                | STRUCT STRUCT_DEC_ID NO_OCB DECLARATION2 CCB NO_SM STRUCT_DEC
                |
                ;

NO_SM :                             { printf ("line %3d: Expected ';'\n", yylineno); err = true; }
                ;

NO_OCB :                            { printf ("line %3d: Expected '{'\n", yylineno); err = true; }
                ;

STRUCT_DEC_ID : ID                  { struct_check ($1, 0); }
                ;

STRUCT_USE_ID : ID                  { struct_check ($1, 1); }
                ;

DECLARATION2 :  TYPE IN_DEC_ID COMMA ID_LIST2 SM DECLARATION2
                | TYPE IN_DEC_ID SM DECLARATION2
                | STRUCT STRUCT_USE_ID IN_IS_DEC_ID COMMA ID_LIST2 SM DECLARATION2
                | STRUCT STRUCT_USE_ID IN_IS_DEC_ID SM DECLARATION2
                | TYPE IN_DEC_ID OSB NUM CSB COMMA ID_LIST2 SM DECLARATION2
                | TYPE IN_DEC_ID OSB NUM CSB SM DECLARATION2
                | STRUCT STRUCT_USE_ID IN_IS_DEC_ID OSB NUM CSB COMMA ID_LIST2 SM DECLARATION2
                | STRUCT STRUCT_USE_ID IN_IS_DEC_ID OSB NUM CSB SM DECLARATION2
                | TYPE IN_DEC_ID COMMA ID_LIST2 NO_SM DECLARATION2
                | TYPE IN_DEC_ID NO_SM DECLARATION2
                | STRUCT STRUCT_USE_ID IN_IS_DEC_ID COMMA ID_LIST2 NO_SM DECLARATION2
                | STRUCT STRUCT_USE_ID IN_IS_DEC_ID NO_SM DECLARATION2
                | TYPE IN_DEC_ID OSB NUM CSB COMMA ID_LIST2 NO_SM DECLARATION2
                | TYPE IN_DEC_ID OSB NUM CSB NO_SM DECLARATION2
                | STRUCT STRUCT_USE_ID IN_IS_DEC_ID OSB NUM CSB COMMA ID_LIST2 NO_SM DECLARATION2
                | STRUCT STRUCT_USE_ID IN_IS_DEC_ID OSB NUM CSB NO_SM DECLARATION2
                |
                ;

ID_LIST2 :      IN_DEC_ID COMMA ID_LIST2
                | IN_DEC_ID
                ;

IN_DEC_ID :     ID                      
                ;

IN_IS_DEC_ID :  ID                      
                ;

IS_DEC_ID :     ID                      { check ($1, 0, true); }
                ;

MAIN_FUNC :     TYPE MAIN ORB CRB OCB SCOPE_INC BODY CCB SCOPE_DEC                       
                | TYPE MAIN ORB CRB OCB SCOPE_INC BODY RETURN NUM SM CCB SCOPE_DEC       
                | TYPE MAIN ORB CRB OCB SCOPE_INC BODY RETURN ID SM CCB SCOPE_DEC         
                | TYPE MAIN ORB CRB OCB SCOPE_INC BODY RETURN SM CCB SCOPE_DEC           
                ;

FOR_LOOP :  FOR SCOPE_INC ORB INIT SM COND SM FOR_ASSIGNMENT CRB OCB BODY1 CCB SCOPE_DEC          
            | FOR SCOPE_INC ORB INIT SM COND SM FOR_ASSIGNMENT CRB SM SCOPE_DEC                  
            | FOR SCOPE_INC ORB INIT SM COND SM FOR_ASSIGNMENT CRB NO_OCB BODY1 CCB SCOPE_DEC 
            ;

SCOPE_INC :                                 { scope ++; }
            ;

SCOPE_DEC :                                 { variable_size[scope] = 0; scope--; }
            ;

INIT :  DECLARATION                             
        | ASSIGNMENT                           
        |                                     
        ;

COND :  USE_ID RELOP COND_LIST  
        | STRUCT_VAR DOT DOT_LIST RELOP COND_LIST               
        |
        ;

USE_ID :    ID                         {check ($1, 1, false); }
            ;

COND_LIST :     USE_ID     
                | STRUCT_VAR DOT DOT_LIST                     
                | NUM
                | USE_ID RELOP COND_LIST  
                | STRUCT_VAR DOT DOT_LIST RELOP COND_LIST       
                | NUM RELOP COND_LIST
                ;

DECLARATION :   TYPE DEC_ID EQ EXP COMMA ID_LIST    
                | TYPE DEC_ID COMMA ID_LIST         
                | TYPE DEC_ID EQ EXP                
                | TYPE DEC_ID      
                | STRUCT STRUCT_USE_ID IS_DEC_ID COMMA ID_LIST
                | STRUCT STRUCT_USE_ID IS_DEC_ID
                | TYPE DEC_ID OSB NUM CSB EQ OCB NUM CCB COMMA ID_LIST  
                | TYPE DEC_ID OSB NUM CSB COMMA ID_LIST         
                | TYPE DEC_ID OSB NUM CSB EQ OCB NUM CCB               
                | TYPE DEC_ID OSB NUM CSB  
                | STRUCT STRUCT_USE_ID IS_DEC_ID OSB NUM CSB COMMA ID_LIST
                | STRUCT STRUCT_USE_ID IS_DEC_ID OSB NUM CSB           
                ;

DEC_ID :    ID                             {check ($1, 0, false); }
            ;

ID_LIST :       DEC_ID COMMA ID_LIST                
                | DEC_ID EQ EXP COMMA ID_LIST       
                | DEC_ID EQ EXP                     
                | DEC_ID                            
                ;
        

FOR_ASSIGNMENT :    ASSIGNMENT
                    |
                    ;

ASSIGNMENT :    USE_ID EQ EXP COMMA ASSIGNMENT_LIST 
                | USE_ID EQ EXP         
                | STRUCT_VAR DOT DOT_LIST EQ EXP COMMA ASSIGNMENT_LIST 
                | STRUCT_VAR DOT DOT_LIST EQ EXP              
                | UNEXP COMMA ASSIGNMENT_LIST
                | UNEXP
                ;

ASSIGNMENT_LIST :   USE_ID EQ EXP COMMA ASSIGNMENT_LIST 
                    | USE_ID EQ EXP         
                    | STRUCT_VAR DOT DOT_LIST EQ EXP COMMA ASSIGNMENT_LIST 
                    | STRUCT_VAR DOT DOT_LIST EQ EXP                  
                    | UNEXP COMMA ASSIGNMENT_LIST
                    | UNEXP
                    ;

BODY :      OCB SCOPE_INC BODY1 CCB BODY SCOPE_DEC               
            | BODY1
            ;

BODY1 :     DECLARATION SM BODY
            | ASSIGNMENT SM BODY
            | CONDITIONAL BODY
            | FOR_LOOP BODY
            | WHILE_LOOP BODY
            | DO_LOOP BODY
            | BREAK SM BODY
            | CONT SM BODY
            | EXIT ORB NUM CRB SM BODY
            | EXIT ORB ID CRB SM BODY
            |
            ; 

STRUCT_VAR :    ID                  { check_struct_var ($1); }
                ;

EXP :       USE_ID OP EXP                           
            | USE_ID EQ EXP                       
            | NUM OP EXP
            | UNEXP
            | USE_ID                              
            | NUM
            | STRUCT_VAR DOT DOT_LIST OP EXP                           
            | STRUCT_VAR DOT DOT_LIST EQ EXP 
            | STRUCT_VAR DOT DOT_LIST
            ;

DOT_LIST :  ID DOT DOT_LIST
            | ID
            ; 

UNEXP :     USE_ID UNOP                             
            | UNOP USE_ID 
            | STRUCT_VAR DOT DOT_LIST UNOP                             
            | UNOP STRUCT_VAR DOT DOT_LIST                             
            ;

CONDITIONAL :       IF ORB EXP CRB SCOPE_INC OCB BODY CCB SCOPE_DEC MORE_CONDITIONS
                    ;

MORE_CONDITIONS :   ELIF ORB EXP CRB SCOPE_INC OCB BODY CCB SCOPE_DEC MORE_CONDITIONS
                    | ELSE SCOPE_INC OCB BODY CCB SCOPE_DEC
                    |
                    ;

WHILE_LOOP :    WHILE ORB EXP CRB SCOPE_INC OCB BODY CCB SCOPE_DEC
                ;

DO_LOOP :       DO SCOPE_INC OCB BODY CCB WHILE ORB EXP CRB SM SCOPE_DEC
                ;

%%

int main () {
    yyparse ();
    return 0;
}

int yyerror (char *s) {
    printf ("line %3d: Syntax error\n\n", yylineno);
}
