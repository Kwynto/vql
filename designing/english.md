# VQL
This is a description of a vulgar query language (VQL) for database management systems (DBMS).  

***Information:*** 
VQL is a simple imperative language for working with databases, free from frills and helping to create simple, understandable and effective database queries.  

***Revision:*** 
VQL-24:1  

**_This repository is under development._**  


## List of terms: 
|Term|Description|
|:-|:-|
|DB|database|
|Table|two-dimensional array, is an organizational unit and a real repository of information inside databases|
|Record|row of a two-dimensional array (table) in which data is stored|
|Cell|Entry element|
|Column|the totality of all cells in the same position in all records|
|DBMS|database management system|


## VQL allows you to:

- create, delete and modify databases and tables 
- describe the data structure 
- identify and manage data in the database 
- access data in the DBMS 
- set permissions for data access and data manipulation


## Main features.

- A database is an organized collection of data designed to store information. 
- Data collections can be represented as two-dimensional arrays or tables. 
- The row of the table can be called "record". 
- The record consists of cells that form columns in the context of the table. 
- Columns must have names and define names for cells in records. 
- The names of databases, tables and columns (cells) must match the format `^[a-zA-Z][a-zA-Z0-9_-]*`
- Records may have additional service cells, the names of which must begin with the underscore "_", that is, correspond to the format `^_[a-zA-Z0-9_-]+`
- Each record within the table must have a unique identifier "_id". 
- The record ID is unique and when a record is deleted, this ID cannot be reused for another record. 
- The record ID is NOT an ordinal index in the table. 
- The record identifier can be generated cryptographically, or it can be a simple incremental identifier. 
- Cells whose names have the postfix "_id" in their composition are cells for linking tables, where the left part of the name (before "_id") is the name of the table. 
- Cells have no type, since the data in the cells is just a set of bytes, and the data in the cells can be interpreted as strings that can be converted to any type on the client software side, or in a user query script. 
- It is desirable to present the results of data samples in JSON format (for maximum compatibility with other languages).


## Query language

### Basic concepts

- The request is a script for accessing the DBMS (program) 
- The request consists of logical blocks called operations 
- An operation can consist of one or more operands 
- Each operation is written on a separate line without end characters, or with a separator "\n" at the end of each operation 
- The correct design of the request is only the design in accordance with the principle of 1TBS 
- Query execution
- The query does not have an entry point and the start of query execution is the first line of the query 
    - Each operation in the request is performed sequentially 
    - Declarations and descriptions of user-defined functions occur directly in the request body 
    - The function must be declared and described before it is called 
    - The function after the declaration can be saved to the database for reuse using the store funcName directive 
    - The function can be removed from the database by the forget funcName directive 
    - Each database has its own set of stored functions 
    - If, when saving a new function, the function name is already occupied, then the old function is deleted from the database, and the new one is written 
    - Function names must match the pattern `[a-zA-Z][a-zA-Z0-9_-]+` 
- The block of operations is indicated by the symbols "{" and "}", that is, the beginning and end of the block of operations are indicated in this way 
    - The block of operations is the body of the function description 
    - The block of operations is the body of the loop 
    - The block of operations can be declared arbitrarily anywhere in the query script, forming an anonymous function 
        - An anonymous function can be used as a right operand and return a value with the return directive 
        - An anonymous function can return a value and write the result of execution to a variable of the left operand, but its internal code cannot be assigned to a variable for further invocation 
- VQL assumes the use of variables that must start with the $ symbol and match the pattern `\$[a-zA-Z0-9_-]+` 
    - Variables are intended for temporary storage of data during query execution 
    - Variables do not require prior declaration or initialization 
    - A variable is considered declared and initialized after any mention of this variable in the left operand of any operation 
- The request can accept variables from the user 
    - The names of the incoming variables must match the pattern `\$[a-zA-Z0-9_-]+`, that is, like any other variables 
    - Incoming variables are passed to the request in JSON format as objects, where the key is the name of the variable and the value is the contents of the variable 
- Operations can use pipelines, that is, data from the execution of one request is transmitted as input to the next request 
    - The pipeline is indicated by the symbol "|" between requests and corresponds to the pattern `\s*|\s*` 
    - Requests in the pipeline can be on different lines, or on the same
line - The response from the left part of the pipeline operation is passed to the right part and fills in the fields of the arguments of the input function from left to right 
    - If the number of returned values from the left part of the pipeline exceeds the number of arguments from the right part, then only those values that are provided by the right part of the pipeline operation are filled in 
    - If the returned values of the pipeline operation are not enough to fill in the arguments, an error is generated and the execution of the entire request is terminated 
    - The lack of filling in the arguments of the right side of the pipeline operation can be compensated by directly filling in the arguments 
    - A complete pipeline can contain many pipeline operations in which data is transferred from one operation to another 
    - The result of the pipeline execution can be passed to the variable of the left operand on the first line of the pipeline 
- VQL supports assignment, comparison, logical operations, arithmetic operations and string concatenation 
    - Assignment is indicated by the symbol "="
- To the left of the assignment sign is the receiving variable and nothing else 
            - If there is no variable to the left of the assignment sign, then the data is passed to the standard reserved variable $result 
            - If the $result variable was used earlier, the data of this variable will be overwritten 
        - To the right of the assignment sign is the data source 
            - The data source can be a variable 
            - The data source can be a custom function 
            - The data source can be a directive or a standard function 
            - The data source can be a pipeline after all its pipeline operations have been performed 
    - Comparison operations 
        - Greater than: ">"
- Less than: "<"
- Equal to: "=="
- Greater than or equal to: ">="
- Less than or equal to: "<="
- Logical operations 
        - Logical AND: "&&"
- Logical OR: "||"
- Logical IS NOT: "!" 
    - Arithmetic operations only support integer calculations 
        - Addition: "+"
- Subtraction: "-"
- Multiplication: "*"
- Division without remainder: "/"
- Remainder of division: "%"
- Exponentiation: "^"
- Increase by one: "++"
- Decrease by one: "--"
- String concatenation 
        - Addition of strings: "."
- The operation can start with any number of space or tab characters, these characters do not carry any lexical meaning and are only needed for a beautiful design of the request body 
- There can be any number of space or tab characters between operands in an operation 
- Any undeclared set of characters is an atom, that is, its name in the source text of the query and its value are equal and are a constant 
    - Any number is an atom in itself, for example, the entry 123456789 is both the name of the number and its value 
    - Any set of characters without spaces and without tabs is a string constant and an atom 
    - To use a string with spaces and other special characters inside, you need to use enclosing quotes (double or single) 
- User-defined and standard functions, as well as directives, return data as the result of an operation.
- Procedures (functions that do not return data) do not make sense in VQL and are not used 
- If a function or directive is called without assigning a value to a variable, the result is placed in the service variable $result 
- Function signature:
- func function_name(argument1, argument2, other arguments) return_value {} 
    - func function_name(argument1, argument2, other arguments) (value1, value2, ...) {} 
- Non-operational data, called comments, may be recorded in the request. The beginning of the comment is indicated by a combination of characters "//" and continues to the end of the line.


### Data types

VQL does not require variable typing or data typing, but the data types themselves exist in the most minimalistic form.

|Type|Designation|Description|
|:-|:-|:-|
|any||Any type|
|atom|[a-zA-Z][a-zA-Z0-9_-]+|Any designation that is both a proper name and a proper meaning|
|boolean|true false|The elements of logic are a kind of atoms|
|number|[0-9]+|A number, a kind of atom that is both a proper name and a proper value|
|error|[0-9]+|A special type for storing the error code as a number, can be interpreted as a number and used in arithmetic operations|
||string|""|A set of any characters that can have a marketable appearance, or can be enclosed in quotation marks (single or double) if you want to use special characters in the meaning|
|array|[]|Array of values|
|object|{}|An object, an associative array, a structure, that is, any form of data that assumes a name-value (key-value) correspondence|

*Important:* Variables and their internal contents must be compatible with the JSON format (for maximum compatibility with other languages). 


### Directives and reserved words

___

***Directive:*** func  
***Arguments:*** signature  
***Description:*** Function Declaration  
***Example:***  
```vql
    func ifNotExists($db string, $table, $columns []string) error {
        $err = create-db($db)
        if $err != 0 {
            return $err
        }

        use($db, "")

        $err = create-table($table,  $columns)
        if $err != 0 {
            return $err
        }

        use($db, $table)

        return 0
    }
    
    $err = ifNotExists($db, $table, ["name", "age", "sex", "prof", "comment"])
```
___
