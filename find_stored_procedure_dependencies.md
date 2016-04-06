title: Find Stored Procedure Dependencies
date: 2016-04-05
---

Recently working in Microsoft SQL Server I needed to find all the dependencires for a specific stored procedure. I ended up with the following:

```sql
use MY_DB

SELECT ROUTINE_NAME
FROM INFORMATION_SCHEMA.ROUTINES
WHERE ROUTINE_DEFINITION LIKE '%myStoredProcedure%'
AND ROUTINE_TYPE='PROCEDURE'
```

This query looks through all the stored procedure definitions, in  the  **MY_DB** database, for the words ```myStoredProcedure```. The results will display all the stored procedures including the **myStoredProcedure**.