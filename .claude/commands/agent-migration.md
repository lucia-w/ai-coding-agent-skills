# /agent-migration

Write and apply database, schema, or API migrations with a coding agent: plan for reversibility, validate data before and after, define a compatibility window, and never run destructive steps without explicit approval.

**When to use:** Adding or modifying database columns or tables, changing an API contract that existing consumers depend on, or migrating data from one shape to another.

---

## Steps

### 1. Classify the migration type and risk

Before writing anything, classify what you're doing:

| Type | Examples | Risk |
|------|----------|------|
| Additive (safe) | Add nullable column, add new endpoint, add enum value | Low — no existing data affected |
| Modifying (careful) | Rename column, change column type, add NOT NULL constraint | Medium — may require a backfill or compatibility window |
| Destructive (dangerous) | Drop column, drop table, remove API endpoint, truncate data | High — irreversible without a backup |

Checkpoint: Is this migration additive, modifying, or destructive? If destructive, do you have a backup?

### 2. Back up before any modifying or destructive migration

Never run a schema or data migration that modifies or deletes existing data without a verified backup:

```bash
# Postgres example
pg_dump $DATABASE_URL > backup_$(date +%Y%m%d_%H%M%S).sql
```

Verify the backup is readable before continuing. A backup you can't restore is not a backup.

The agent must not run migration commands until you confirm the backup exists and is valid.

### 3. Write the migration as a reversible pair

Every migration must have a `up` and a `down`:

```sql
-- up: add the new column
ALTER TABLE orders ADD COLUMN shipped_at TIMESTAMPTZ;

-- down: revert
ALTER TABLE orders DROP COLUMN shipped_at;
```

Ask the agent to write both before you review either. If the agent can't write a `down` migration, the `up` is probably destructive — treat it as such.

For data migrations, the pair is: transform forward + restore from backup. If the forward transform can't be reversed without the backup, document this explicitly.

### 4. Define the compatibility window for API migrations

If you're changing an API that external consumers depend on:

- **Never remove a field or endpoint without a deprecation period** — consumers break silently
- **Run old and new in parallel** during the window — both old field names and new ones accepted/returned
- **Set a sunset date** and communicate it before the migration

Ask the agent to implement the parallel phase first, before the removal:

```
Implement the new field `shipping_address` while keeping the old field `address` populated.
Both should work identically during the compatibility window.
The old field removal is a separate PR after the window closes.
```

### 5. Test the migration on a copy of production data

Do not test migrations only on synthetic or seed data. Real production data contains shapes your dev fixtures don't:

```
Before running the migration on production:
1. Restore a production backup to a staging database
2. Run the migration on staging
3. Verify row counts before and after match expectations
4. Spot-check a sample of migrated records
5. Run the application against staging and confirm it behaves correctly
```

The agent can help write the validation queries, but you run the migration on staging yourself.

### 6. Write the rollout plan

For any migration beyond a simple additive change, write a rollout sequence before touching production:

```markdown
## Rollout plan

1. Deploy application code that supports both old and new schema (backward-compatible)
2. Run `up` migration on production
3. Verify data with validation queries
4. Monitor application for errors for [time window]
5. Deploy application code that removes support for old schema
6. (If destructive) Remove old columns/tables after compatibility window closes

## Rollback procedure

If step 2 fails: run `down` migration — no data loss expected
If step 3 fails: run `down` migration, restore from backup if data was modified
If step 4 fails: roll back application code (schema stays as-is), investigate
```

### 7. Write data validation queries

After running the migration, verify the data:

```sql
-- row count unchanged
SELECT COUNT(*) FROM orders;

-- no NULLs in a column that should be populated
SELECT COUNT(*) FROM orders WHERE shipped_at IS NULL AND status = 'shipped';

-- spot check migrated values
SELECT id, old_field, new_field FROM orders LIMIT 20;
```

Ask the agent to write these queries before the migration runs. Run them after.

### 8. Get explicit approval before running on production

The agent must not execute migration commands on a production database without explicit, typed approval from you:

```
I am ready to run the migration. This will modify [N] rows in the [table] table.
The backup at [location] was verified at [time].
Do I have approval to proceed?
```

This is a hard stop. No autonomous execution of production migrations.

---

## Exit criteria

- [ ] Migration classified: additive, modifying, or destructive
- [ ] Backup verified (for modifying and destructive migrations)
- [ ] `up` and `down` migrations both written and reviewed
- [ ] Compatibility window defined for API-breaking changes
- [ ] Migration tested on a staging environment with production-like data
- [ ] Data validation queries written and run after migration
- [ ] Rollout plan and rollback procedure documented
- [ ] Explicit human approval given before any production migration runs

---

## Common shortcuts to avoid

**"It's just adding a column — no need for a backup."** Additive migrations are low risk. Modifying and destructive ones are not. Get into the habit of always knowing your backup situation before you start.

**"I'll write the rollback later."** Write it before the migration runs, when you're thinking clearly about the state you're moving from. After a failed migration is not the time to figure out how to reverse it.

**"Let the agent run the migration autonomously."** The agent can write and validate the migration. Execution on a production database requires a human in the loop — the consequences of a mistake are immediate and may be irreversible.

**"We'll remove the old API field in this same PR."** Deprecation and removal are two separate deployments with a window between them. Collapsing them breaks consumers without warning.
