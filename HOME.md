```dataview
LIST
FROM "0_Daily"
WHERE file.day = date(today)
FLATTEN file.content
```
> [!danger] Do
> ```tasks
> not done
> due before today
> sort by due
> ```
> ---
> ```tasks
> not done
> path includes 0_Daily
> (description does not include [[) AND (no due date)
> sort by due
> ```

> [!todo] Task
> ```tasks
> not done
> path includes 0_Daily
> sort by due
> ```

> [!success] Done
> ```tasks
> done
> done after date - 3 days
> sort by done reverse
> ```

