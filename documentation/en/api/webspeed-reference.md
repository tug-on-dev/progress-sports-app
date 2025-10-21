# WebSpeed Reference

> **📖 Complete Version**: For full WebSpeed/4GL reference, see [French WebSpeed Reference](../../fr/api/webspeed-reference.md)

## Standard Pattern

All data API programs follow this standard pattern:

```progress
USING Progress.Json.ObjectModel.JsonObject.

CREATE WIDGET-POOL.

{src/web2/wrap-cgi.i}

DEFINE TEMP-TABLE tt{Entity} LIKE {Entity}.
DEFINE DATASET ds{Entity} FOR tt{Entity}.

RUN process-web-request.

PROCEDURE outputHeader :
    output-content-type ("application/json":U).
END PROCEDURE.

PROCEDURE process-web-request :
    DEFINE VARIABLE oJsonObject AS JsonObject NO-UNDO.
    DEFINE VARIABLE lChar AS LONGCHAR NO-UNDO.
    
    RUN outputHeader.
    
    EMPTY TEMP-TABLE tt{Entity}.
    FOR EACH {Entity} NO-LOCK:
        CREATE tt{Entity}.
        BUFFER-COPY {Entity} TO tt{Entity}.
    END.
    
    oJsonObject = NEW JsonObject().
    oJsonObject:READ(DATASET ds{Entity}:HANDLE).
    
    lChar = oJsonObject:GetJsonText().
    {&OUT-LONG} lChar.
END PROCEDURE.
```

## Data API Programs

### customer-data.p
Returns customer data in JSON format.

**Key Features**:
- Transforms names to uppercase
- Uses temp-table/dataset pattern
- NO-LOCK for performance

### state-data.p
Returns US state data in JSON format.

**Dataset Structure**:
```progress
DEFINE TEMP-TABLE ttState LIKE State
    FIELD State      AS CHARACTER
    FIELD StateName  AS CHARACTER
    FIELD Region     AS CHARACTER.

DEFINE DATASET dsState FOR ttState.
```

## View Programs (SpeedScript)

### customer-view.html
HTML page with embedded SpeedScript generating a Kendo UI grid.

```html
<?WS>
USING Progress.Json.ObjectModel.*.
</?WS>
<?WS>
{ table-config.i &TableName = "Customer" }
</?WS>
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Customer View</title>
    <?WS>{header.i}</?WS>
    <script>
        var config = `cConfig`;
    </script>
    <script src="/grid.js"></script>
</head>
<body>
    <div id="grid"></div>
</body>
</html>
```

## Include Files

### table-config.i
Generates dynamic JSON configuration for Kendo UI grids based on database metadata.

**Usage**:
```progress
{ table-config.i &TableName = "Customer" }
```

**Generates**:
- Field configurations from DB schema
- API URL references
- Grid column definitions

### header.i
Generates standard HTML headers with Kendo UI links.

**Includes**:
- Kendo UI CSS (Blue Opal theme)
- jQuery
- Kendo UI JavaScript
- Progress JSDO library

### dbconnect.p
Database connection utility.

```progress
IF NOT CONNECTED("sports2020") THEN
    CONNECT "-pf /psc/wrk/autoreconnect.pf".
```

## Compilation

### compile.p
Compiles 4GL programs and SpeedScript files.

**Usage**:
```bash
mpro -b -p src/compile.p \
     -param src=src/webspeed,target=/artifacts/webspeed
```

**Supported Formats**:
- `.p` files → `.r` (compiled 4GL)
- `.w` files → `.r` (WebSpeed programs)
- `.html` files → `.w` → `.r` (SpeedScript)

## Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| Data APIs | `{entity}-data.p` | `customer-data.p` |
| View Pages | `{entity}-view.html` | `customer-view.html` |
| Static Pages | `{name}.html` | `about.html` |
| Include Files | `{name}.i` | `table-config.i` |
| Utilities | `{function}.p` | `dbconnect.p` |

## Best Practices

### Performance
```progress
/* Use appropriate indexes */
FOR EACH Customer NO-LOCK
    USE-INDEX CustNum
    WHERE Customer.Country = "USA":
    /* Processing */
END.

/* Avoid queries in loops */
/* Create widget pool explicitly */
CREATE WIDGET-POOL.
/* Clean up at end */
DELETE WIDGET-POOL.
```

### Error Handling
```progress
PROCEDURE process-web-request :
    DEFINE VARIABLE lError AS LOGICAL NO-UNDO.
    DEFINE VARIABLE cErrorMsg AS CHARACTER NO-UNDO.
    
    DO ON ERROR UNDO, LEAVE:
        /* Business logic */
        IF ERROR-STATUS:ERROR THEN DO:
            lError = TRUE.
            cErrorMsg = ERROR-STATUS:GET-MESSAGE(1).
        END.
    END.
    
    IF lError THEN
        RUN output-error(cErrorMsg).
    ELSE
        RUN output-success.
END PROCEDURE.
```

### Security
```progress
/* Input validation */
DEFINE VARIABLE cParam AS CHARACTER NO-UNDO.
cParam = get-value("param1").

IF cParam = ? OR cParam = "" THEN
    cParam = "default-value".

/* Escape for SQL injection prevention */
cParam = REPLACE(cParam, "'", "''").
```

---

**📚 For complete reference including:**
- Detailed program structure
- Advanced patterns
- Full code examples
- Optimization techniques
- Debugging tips

**See the [complete French documentation](../../fr/api/webspeed-reference.md)**
