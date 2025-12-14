
実装機のチケット
```dataview
TABLE created as Created, due as Due
FROM "2_Tickets"
WHERE contains(file.name, "XPLT") OR contains(file.name, "NPM")
SORT file.name ASC
```
