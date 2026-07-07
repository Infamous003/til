# Partial Unique Indexes & Soft Delete

Soft deleting keeps rows in the table by setting their `deleted_at` field instead of remobing them from the table.

Suppose a class exists:
Academic_Year: 2025-26
Standard: PRIMARY
Level: 1
Section: A
deleted_at = NULL

if this row is soft deleted:

deleted_at = 2026-07-07 ...

the row still exists in the database.

A normal unique index will still this and prevent adding another identical row with the same values.

To ignore soft-deleted rows, create a partial unique index

```sql
CREATE UNIQUE INDEX classes_unique_idx
ON classes (
    academic_year,
    standard,
    level,
    section
)
WHERE deleted_at IS NULL;
```

Only activet rows (`deleted_at IS NULL`) will participate in uniqueness

This allows recreating a previously deleted row, while still keeping the unique check for non deleted ones.

## Rule of thumb
Whenever a table has soft delete, think about whether it should also be
included in unique constraint or not, ideally you don't