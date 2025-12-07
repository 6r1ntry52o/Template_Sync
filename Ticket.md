```dataview
TABLE 
FROM "2_Tickets"
WHERE completed = false
SORT due ASC
```
```dataview
TABLE 
FROM "2_Tickets"
WHERE Sub = true
SORT due ASC
```
```dataview
TABLE 
FROM "2_Tickets"
WHERE contains(file.tags, "Ticket")
SORT due ASC
```
