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

***Directive:*** return  
***Arguments:*** data  
***Description:*** Interrupts the execution of the function and returns the data. By default, with empty parameters, the data is equal to an empty string, an empty array, an empty object, false or 0.  
***Example:***  
```vql
    func someFunc() boolean {
        return true
    }

    $ok = someFunc()
```
___

***Directive:*** store  
***Arguments:*** name of the previously declared function  
***Description:*** Writes a function to the current database, but does not run it.  
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

    store ifNotExists
```
___

***Directive:*** forget  
***Arguments:*** name of the function already saved in the database  
***Description:*** Removes a function from the current database.  
***Example:***  
```vql
    forget ifNotExists
```
___

***Directive:*** functionality  
***Return value:*** a string with the text of all saved functions  
***Description:*** Returns the full text of the functions stored in the database.  
***Example:***  
```vql
    $sourceCode = functionality
```
```vql
    functionality
    $sourceCode = $result
```
___

***Directive:*** for _conditions_ {}  
***Arguments:*** initialization, condition, operators  
***Description:*** Cyclic design, for repeated repetition of any operations.  
***Example:***  
```vql
    for $i = 0, $i < 10, $i++ {
        // some kind of operation 
        log("INFO", "i = ".$i)
    }
```
___

***Directive:*** continue  
***Description:*** Ending the current iteration of the loop and moving on to the next iteration.  
***Example:***  
```vql
    for $i = 0, $i < 10, $i++ {
        // some kind of operation
        if $i < 6 {
            continue
        }

        log("INFO", "i = ".$i)
    }
```
___

***Directive:*** break  
***Description:*** Completion of the cycle.  
***Example:***  
```vql
    for $i = 0, $i < 10, $i++ {
        // some kind of operation
        if $i > 6 {
            break
        }
        
        log("INFO", "i = ".$i)
    }
```
___

***Directive:*** if _condition_ {} else if _condition_ {} else {}  
***Arguments:*** logical operations  
***Description:*** Conditional code execution.  
***Example:***  
```vql
    if $1 == 0 {
        log("INFO", "null")
    } elseif $1 > 0 {
        log("INFO", "more")
    } else {
        log("INFO", "less")
    }
```
___

***Directive:*** where _condition_ < orderby _colName_ asc|desc > < limit _start_, _offset_ >  
***Arguments:*** logical operations  
***Return value:*** array of _id identifiers  
***Description:*** Conditional selection returns an array of _id corresponding to the specified conditions. The selection is performed from the current database and tables set by the use() command. The left operand in the condition is always the name of the cell in the table entry, and the right operand is always the value of the cell in the record for fulfilling the condition.  
***Example:***  
```vql
    use("shop", "customers")
    $idsСustomers = where "age" >= 18 AND "city" == "Moscow" 
```
```vql
    use(shop, customers)
    $idsСustomers = where age >= $1 AND city == $2 
```
___

***Directive:*** < ... > limit _start_, _offset_  
***Arguments:*** numerical constraints  
***Description:*** This is an additional subdirectory for the where directive. The limiters are the initial index and offset in the array after sampling according to the condition to form the final resulting array of identifiers.  
***Example:***  
```vql
    use("shop", "customers")
    $idsСustomers = where "age" >= 18 AND "city" == "Moscow" limit 0, 100
```
```vql
    use(shop, customers)
    $idsСustomers = where age >= $1 AND city == $2 limit 100, 100
```
___

***Directive:*** < ... > orderby _colName_ asc|desc  
***Arguments:*** name of the column to sort  
***Description:*** This is an additional subdirectory for the where directive. Specifying the sort order and column name as the basis for sorting the resulting dataset. Sorting is carried out in the forward and reverse order. By default, sorting is done in the forward order (asc).  
***Example:***  
```vql
    use("shop", "customers")
    $idsСustomers = where "age" >= 18 AND "city" == "Moscow" orderby "name" asc
```
```vql
    use("shop", "customers")
    $idsСustomers = where "age" >= 18 AND "city" == "Moscow" orderby "_id" desc
```
```vql
    use(shop, customers)
    $idsСustomers = where age >= $1 AND city == $2 orderby age asc limit 0, 100
```
___

***Directive:*** transaction { _operations_ rollback _operations_ commit }  
***Arguments:*** operations  
***Description:*** Atomic execution of several related operations. Starts a transaction, cancels operations, and confirms the execution of operations. The rollback and commit subdirectories complete the transaction.  
***Example:***  
```vql
    use("shop", "customers")
    $idsСustomers = where "age" >= 18 AND "city" == "Moscow"

    transaction {
        use("shop", "sales")
        $idsSales = where "customers_id" == $idsСustomers
        $err = update( {"geosale": $1}, $idsSales )
        if $err != 0 {
            rollback
        }

        use("shop", "customers")
        $data, $err = select(["sumsale"], $idsСustomers)
        if $err != 0 {
            rollback
        }

        $l1 = len($data)
        for $i = 0, $i < $l1, $i++ {
            $record = $data[$i]
            $id = $record["_id"]
            $value = $record["sumsale"] + $1
            $err = update( {"sumsale": $value}, [$id] )
            if $err != 0 {
                rollback
            }
        }

        log("INFO", "Transaction is complete.")
        commit
    }
```
___

***Directive:*** response  
***Arguments:*** variable with data  
***Description:*** Interrupts the execution of the request and sends the client a return object corresponding to a specific DBMS.  
***Example:***  
```vql
    use("shop", "customers")
    $idsСustomers = where "age" >= 18 AND "city" == "Moscow" limit 0, 100

    $newTicket, $_ = verification()

    $res = ""
    if len($newTicket) == 0 {
        $res = {"status": "ok", "data": $idsСustomers}
    } else {
        $res = {"status": "ok", "ticket": $newTicket, "data": $idsСustomers}
    }

    response $res
```
___


### Standard functions 

#### GPF - General Purpose Functions

___

##### func len( \$v any ) number
***Name:*** len  
***Arguments:*** one variable of any type  
***Return value:*** number (length of the entity)  
***Description:*** Defines the type of the variable and returns the length of the entity (for numbers, returns the number of digits in the number)  
***Example:***  
```vql
    $lengthStr = len("any string")
```

```vql
    $lengthStr = len($1)
```
___

##### func verification() string, error 
***Name:*** verification  
***Arguments:***  
***Return value:*** ticket and error code  
***Description:*** Checks the validity of the ticket and returns a new ticket, if necessary, or an old ticket. It should be noted that a request with an incorrect ticket should not start execution at all, and the return of a new ticket is only needed to format the correct response.  
***Example:***  
```vql
    $newTicket, $_ = verification()
```
___

##### func log($level string, $v any) boolean
***Name:*** log  
***Arguments:*** logging level and any value  
***Return value:*** indicates that the operation is being performed  
***Description:*** Writes to the database log.  
***Example:***  
```vql
    $ok = log("INFO", "This is information.")
```
```vql
    log("INFO", "This is information.")
```
___

##### func version() string, string 
***Name:*** version  
***Return value:*** language version and DBMS version  
***Description:*** Returns two rows with the VQL version and the DBMS version.  
***Example:***  
```vql
    $vql, $dbms = version()
    log("INFO", "VQL  version: ".$vql)
    log("INFO", "DBMS version: ".$dbms)
```
___


#### DDF - Data Definition Functions 

___

##### func create-db( \$db string ) error 
***Name:*** create-db  
***Arguments:*** database name  
***Return value:*** Error code  
***Description:*** Creating a database with a given name.  
***Example:***  
```vql
    $err = create-db("dbName")
```
```vql
    $err = create-db($1)
```
___

##### func create-table( \$table string, \$columns []string) error 
***Name:*** create-table  
***Arguments:*** table name and array of rows with column names  
***Return value:*** Error code  
***Description:*** Creating a table with a given name and a set of specified columns. Before executing, it is necessary to execute use(dbName, "").  
***Example:***  
```vql
    use("dbName", "")
    $err = create-table("tableName", ["name", "age", "sex", "prof", "comment"])
```
```vql
    use($1, "")
    $err = create-table($2, $3)
```
___

##### func alter-db-rename( \$oldName string, \$newName string ) error 
***Name:*** alter-db-rename  
***Arguments:*** old and new database name  
***Return value:*** Error code  
******Description:*** Changing the database name.  
***Example:***  
```vql
    $err = alter-db-rename("dbName", "newDbName")
```
```vql
    $err = alter-db-rename($1, $2)
```
___

##### func alter-table-rename( \$oldName string, \$newName string ) error 
***Name:*** alter-table-rename  
***Arguments:*** old and new table name  
***Return value:*** Error code  
***Description:*** Changing the name of the table in the melon database.  
***Example:***  
```vql
    use("dbName", "")
    $err = alter-table-rename("tableName", "newTableName")
```
```vql
    use("dbName", "")
    $err = alter-table-rename($1, $2)
```
___

##### func alter-table-add( \$colName string, \$sett object ) error 
***Name:*** alter-table-add  
***Arguments:*** the name of the new column in the table and its characteristics  
***Return value:*** Error code  
***Description:*** Adding a new column to the current worksheet. The column characteristics can be an empty object, or filled in in various ways depending on the capabilities of the DBMS.  
***Example:***  
```vql
    use("shop", "customers")
    $err = alter-table-add("city", {})
```
```vql
    use("shop", "customers")
    $err = alter-table-add("city", {"Default": "Moscow", "NotNull": true, "Unique": false} )
```
```vql
    use($1, $2)
    $err = alter-table-add($3, $4)
```
___

##### func alter-table-drop( \$colName string ) error 
***Name:*** alter-table-drop  
***Arguments:*** name of the column to delete in the table  
***Return value:*** Error code  
***Description:*** Deleting a column from the current worksheet.  
***Example:***  
```vql
    use("shop", "customers")
    $err = alter-table-drop("city")
```
```vql
    use($1, $2)
    $err = alter-table-drop($3)
```
___

##### func alter-table-modify( \$colName string, \$sett object ) error 
***Name:*** alter-table-modify  
***Arguments:*** the name of the column being modified in the table and a set of its characteristics  
***Return value:*** Error code  
***Description:*** Changing the set of column characteristics from the current worksheet.  
***Example:***  
```vql
    use("shop", "customers")
    $err = alter-table-modify("city", {"Default": "Moscow", "NotNull": true, "Unique": false} )
```
```vql
    use($1, $2)
    $err = alter-table-modify($3, $4)
```
___

##### func alter-table-modify-rename( \$colOldName string, \$colNewName string ) error 
***Name:*** alter-table-modify-rename  
***Arguments:*** old and new column name  
***Return value:*** Error code  
***Description:*** Renaming a column from the current worksheet.  
***Example:***  
```vql
    use("manage", "users")
    $err = alter-table-modify-rename("login", "username" )
```
```vql
    use($1, $2)
    $err = alter-table-modify-rename($3, $4)
```
___

##### func drop-db( \$db string ) error 
***Name:*** drop-db  
***Arguments:*** database name  
***Return value:*** Error code  
***Description:*** Deleting a database with the specified name.  
***Example:***  
```vql
    $err = drop-db("dbName")
```
```vql
    $err = drop-db($1)
```
___

##### func drop-table( \$table string ) error 
***Name:*** drop-table  
***Arguments:*** table name  
***Return value:*** Error code  
***Description:*** Deleting a table with the specified name from the current working database.  
***Example:***  
```vql
    use("dbName", "")
    $err = drop-table("tableName")
```
```vql
    use($1, "")
    $err = drop-table($2)
```
___


#### DMF - Data Manipulation Functions 

___

##### func select( \$fields []string, $where []number) []{}, error 
***Name:*** select  
***Arguments:*** array of field names for selection and array of identifiers _id  
***Return value:*** array of objects and error code  
***Description:*** Retrieves records from the current worksheet. Receives at the input an array of strings with the names of the extracted cells and an array of unique record identifiers. The response is an array of records. Each record is an object in which the name of the field is the name of the cell of the record from the worksheet, and the value of the field is the extracted value. If the field array is empty, then all fields are selected. If the array of identifiers is empty, then the selection covers the entire table.  
***Example:***  
```vql
    use("shop", "customers")
    $idsСustomers = where "age" >= 18 AND "city" == "Moscow" limit 0, 100

    $data, $err = select(["login", "name", "surname", "age", "city"], $idsСustomers)
```
```vql
    use($1, $2)
    $idsСustomers = where "age" >= $3 AND "city" == $4 limit 0, 100

    $data, $err = select(["login", "name", "surname", "age", "city"], $idsСustomers)
```
```vql
    use($1, $2)
    $idsСustomers = where "age" >= $3 AND "city" == $4 limit 0, 100

    $data, $err = select([], $idsСustomers)
```
```vql
    func selectAll( $nameDb string, $nameTab string) []{}, error {
        use($nameDb, $nameTab)
        $inData, $err = select([], [])
        return $inData, $err
    }

    store selectAll
```
___

##### func insert( \$records []{}) []number, error 
***Name:*** insert  
***Arguments:*** array of objects for creating new records from field names and their values  
***Return value:*** array of unique identifiers of created records and error code  
***Description:*** Creates one or more records in the current worksheet. Retrieves an array of objects from field names and their values to be inserted into the table as a new entry. The response is an array of new record IDs.  
***Example:***  
```vql
    use("shop", "customers")
    
    $insId, $err = insert( [{"login": "kwynto", "name": "Constantine", "surname": "Zavezeon", "age": 42, "city": "Kursk"}] )
```
```vql
    use($1, $2)
    $record = [{"login": $3, "name": $4, "surname": $5, "age": $6, "city": $7}]

    $insId, $err = insert($record)
```
```vql
    use($1, $2)
    
    $insId, $err = insert($3)
```
___

##### func update( \$fields {}, \$where []number) error 
***Name:*** update  
***Arguments:*** an object with field names and new values, and an array of _id identifiers of the records being modified  
***Return value:*** Error code  
***Description:*** Modifies the entries in the current worksheet.  
***Example:***  
```vql
    use("shop", "customers")
    $idsСustomers = where "city" == "Nur-Sultan"
    
    $err = update( {"city": "Astana"}, $idsСustomers )
```
```vql
    use($1, $2)
    $idsСustomers = where "city" == $3
    
    $err = update( {"city": $4}, $idsСustomers )
```
```vql
    func renameTheCity( $nameDb string, $nameTab string, $oldNameCity string, $newNameCity string ) error {
        use($nameDb, $nameTab)
        $idsСustomers = where "city" == $oldNameCity
        $err = update( {"city": $newNameCity}, $idsСustomers )
        return $err
    }

    store renameTheCity
```
___

##### func delete( \$where []number) error 
***Name:*** delete  
***Arguments:*** array of identifiers _id of deleted records  
***Return value:*** Error code  
***Description:*** Deletes entries.  
***Example:***  
```vql
    use("shop", "customers")
    $idsСustomers = where "ban" == true
    
    $err = delete( $idsСustomers )
```
___

##### func truncate() error 
***Name:*** truncate  
***Return value:*** Error code  
***Description:*** Clearing the table (deleting all records). It does not affect incremental identifiers, only deletes records with data.  
***Example:***  
```vql
    use("shop", "customers")
        
    $err = truncate()
```
___


#### DCF - Data Control Functions 

___

##### func auth( \$login string, \$password string ) string
***Name:*** auth  
***Arguments:*** login and password  
***Return value:*** ticket  
***Description:*** Authorizes the user, accepts a username and an unhashed password, and returns a string with a ticket for further work with the DBMS.  
***Example:***  
```vql
    $ticket = auth("root", "toor")
```

```vql
    $ticket = auth($1, $2)
```
___

##### func auth-secure( \$login string, \$hash string ) string
***Name:*** auth-secure  
***Arguments:*** login and password hash  
***Return value:*** ticket  
***Description:*** Authorizes the user, accepts the username and password hash, and returns the ticket string for further work with the DBMS.  
***Example:***  
```vql
    $ticket = auth-secure("root", "1b4f0e9851971998e732078544c96b36c3d01cedf7caa332359d6f1d83567014")
```
```vql
    $ticket = auth-secure($1, $2)
```
___

##### func auth-new( \$login string, \$password string ) error
***Name:*** auth-new  
***Arguments:*** login and password  
***Return value:*** Error code  
***Description:*** Adds a new user, without privileges. Accepts a username and an unhashed password, and returns formal signs of completion.  
***Example:***  
```vql
    $err = auth-new("test1", "test1")
```
```vql
    $err = auth-new($1, $2)
```
___

##### func auth-remove( \$login string ) error
***Name:*** auth-remove  
***Arguments:*** login  
***Return value:*** Error code  
***Description:*** Deletes the user. Accepts the login, and returns the formal signs of the operation.  
***Example:***  
```vql
    $err = auth-remove("test1")
```
```vql
    $err = auth-remove($1)
```
___

##### func auth-change( \$login string, \$password string, \$roles []string ) error
***Name:*** auth-change  
***Arguments:*** login, password, list of new roles  
***Return value:*** Error code  
***Description:*** Changes the password for the user and his role within the DBMS. Accepts a username, a new unencrypted password and a list of roles, and returns formal signs of fulfillment.  
***Example:***  
```vql
    $err = auth-change("test1", "test1", ["ADMIN", "ENGINEER", "MANAGER", "USER"])
```
```vql
    $err = auth-change(test1, test1, [ADMIN, ENGINEER, MANAGER, USER])
```
```vql
    $err = auth-change($1, $2, $3)
```
___

##### func use( \$db string, \$table string ) error
***Name:*** use  
***Arguments:*** DB name and table name  
***Return value:*** Error code  
***Description:*** Sets the current working database and the working table that will be used by the rest of the commands. If the table name is an empty string, then only the working database is installed.  
***Example:***  
```vql
    $err = use("dbName", "tableName")
```
```vql
    $err = use($1, $2)
```
```vql
    $err = use("dbName", "")
```
___

##### func show-dbs() []string, error
***Name:*** show-dbs  
***Arguments:***  
***Return value:*** array of strings and error code  
***Description:*** Returns the names of all databases maintained by this DBMS instance.  
***Example:***  
```vql
    $dbs, $err = show-dbs()
```
```vql
    $dbs, $err = show-dbs()
```
___

##### func show-tables() []string, error
***Name:*** show-tables  
***Arguments:***  
***Return value:*** array of strings and error code  
***Description:*** Returns the names of all tables in the current database.  
***Example:***  
```vql
    $dbs, $err = show-tables()
```
```vql
    $dbs, $err = show-tables()
```
___

##### func describe( \$table string ) []string, error
***Name:*** describe  
***Arguments:*** table name  
***Return value:*** array of strings and error code  
***Description:*** Returns an array of rows, where each row is the name of the columns.  
***Example:***  
```vql
    $columns, $err = describe("users")
```
```vql
    $columns, $err = describe($1)
```
___

##### func explain( \$table string ) []{}, error
***Title:*** explain  
***Arguments:*** table name  
***Return value:*** array of objects and error code  
***Description:*** Returns an array of objects, where each object (array element) is a complete description of the column properties with the specifics of a particular DBMS.  
***Example:***  
```vql
    $columns, $err = explain("users")
```
```vql
    $columns, $err = explain($1)
```
___

##### func grant( \$dbs []string, \$users []string, \$privileges []string ) error
***Name:*** grant  
***Arguments:*** database list, user list, privilege list  
***Return value:*** Error code  
***Description:*** Accepts three arrays of strings with database names, user names, and privilege names. Then sets the specified privileges for users in each database. Returns the error code.  
***Example:***  
```vql
    $err = grant(["dbName1", "dbName2"], ["test1", "test2"], ["select", "insert", "update", "delete"])
```
```vql
    $err = grant($dbs, $users, $privs)
```
___

##### func revoke( \$dbs []string, \$users []string, \$privileges []string ) error
***Name:*** revoke  
***Arguments:*** database list, user list, privilege list  
***Return value:*** Error code  
***Description:*** Accepts three arrays of strings with database names, user names, and privilege names. Then deletes the specified privileges for users in each database. Returns the error code.  
***Example:***  
```vql
    $err = revoke(["dbName1", "dbName2"], ["test1", "test2"], ["select", "insert", "update", "delete"])
```
```vql
    $err = revoke($dbs, $users, $privs)
```
___

