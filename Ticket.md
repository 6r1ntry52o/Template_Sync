
```dataview
TABLE due as Due
FROM "1_Tickets"
WHERE (completed = false) AND (contains(file.tags, "Ticket"))
SORT due DESC
```
```dataview
TABLE 
FROM "1_Tickets"
WHERE  (completed = true) AND (Sub = true) AND (contains(file.tags, "Ticket"))
SORT due DESC
```
```dataview
TABLE created as Created, due as Due
FROM "1_Tickets"
WHERE (contains(file.tags, "Ticket"))
SORT file ASC
```

```
```
