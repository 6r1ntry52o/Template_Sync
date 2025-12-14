
```dataview
LIST
FROM "0_Daily"
WHERE file.day = date(today)
FLATTEN file.content
```
> [!danger] Do
> ```tasks
> not done
> due before tomorrow
> sort by due
> ```
> ---
> ```tasks
> not done
> path includes 0_Daily
> (priority is not low) AND (priority is not lowest)
> (description does not include [[) OR (no due date)
> sort by due
> ```

> [!todo] Task
> ```tasks
> not done
> NOT (due before tomorrow)
> path includes 0_Daily
> sort by due
> ```
```dataview
TABLE due as Due
FROM "2_Tickets"
WHERE (completed = false) AND (contains(file.tags, "Ticket"))
SORT due DESC
```
> [!success] Done
> ```tasks
> done
> done after date - 3 days
> sort by done reverse
> ```

