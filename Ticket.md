```dataview
TABLE 
FROM "2_Tickets"
WHERE (completed = false) AND (contains(file.tags, "Ticket"))
SORT due ASC
```
```dataview
TABLE 
FROM "2_Tickets"
WHERE (Sub = true) AND (contains(file.tags, "Ticket"))
SORT due ASC
```
```dataview
TABLE 
FROM "2_Tickets"
WHERE (contains(file.tags, "Ticket"))
SORT due ASC
```