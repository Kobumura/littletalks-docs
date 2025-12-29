# PHP Data Access Layer Pattern

> **Used by**: littletalks-admin, WXING, football
>
> Standardized repository pattern with dynamic SQL builders for PHP projects.

## Why This Matters: Audit Integrity

**Critical for audit logging and change tracking systems.**

When using hardcoded column lists in INSERT/UPDATE statements, adding a new field requires updating:
1. The model class
2. The database migration
3. **Every repository method that touches that table** ← Easy to miss!

If you forget to update the repository, changes to that field are **silently ignored**:
- User edits the field in the UI
- Change request system records the change
- Audit log shows the intended change
- **But the database UPDATE doesn't include the column!**
- The change is lost, audit trail is broken

**The dynamic SQL builder pattern solves this** - new fields in the model's `toArray()` are automatically included in all INSERT/UPDATE operations.

## Overview

This pattern provides:
- Base `DataAccessObject` class with dynamic SQL builders
- Repository pattern for database operations
- Automatic handling of new model fields (no hardcoded column lists)
- Consistent create/update/save workflow
- **Audit-safe persistence** - changes tracked in audit logs are guaranteed to persist

## Architecture

```
DataAccessObject (base class)
    ↓ extends
Repository Classes (SoldierRepository, UserRepository, etc.)
    ↓ uses
Model Classes (Soldier, User, etc. with toArray() method)
```

## DataAccessObject Base Class

All repositories extend this class which provides:

### Query Methods
```php
// Execute query with parameters (returns self for chaining)
$this->query($sql, $params)->fetch();
$this->query($sql, $params)->fetchAll();

// Execute INSERT/UPDATE/DELETE (returns row count)
$this->execute($sql, $params);

// Get last insert ID
$this->getLastInsertId();
```

### Dynamic SQL Builders

**These are the key methods for avoiding hardcoded column lists:**

```php
/**
 * Insert from data array
 * @param string $table Table name
 * @param array $data Associative array of column => value
 * @param array $excludeColumns Columns to exclude (default: ['id'])
 * @param bool $addTimestamps Add created_at/updated_at = NOW()
 * @return int Last insert ID
 */
protected function insertFromArray(
    string $table,
    array $data,
    array $excludeColumns = ['id'],
    bool $addTimestamps = true
): int

/**
 * Update from data array
 * @param string $table Table name
 * @param array $data Associative array of column => value
 * @param int $id Record ID to update
 * @param array $excludeColumns Columns to exclude from update
 * @param bool $addTimestamp Add updated_at = NOW()
 * @return bool True if row was updated
 */
protected function updateFromArray(
    string $table,
    array $data,
    int $id,
    array $excludeColumns = ['id'],
    bool $addTimestamp = true
): bool

/**
 * Save (insert or update) from data array
 * @param string $table Table name
 * @param array $data Must include 'id' key if updating
 * @param array $excludeColumns Columns to exclude
 * @return int The record ID (new or existing)
 */
protected function saveFromArray(
    string $table,
    array $data,
    array $excludeColumns = []
): int
```

## Repository Implementation

### The save() Pattern

**GOOD - Dynamic approach (new fields automatically included):**

```php
class SoldierRepository extends DataAccessObject
{
    public function save(Soldier $soldier): ?Soldier
    {
        // Validate first
        $errors = $soldier->validate();
        if (!empty($errors)) {
            throw new \InvalidArgumentException('Validation failed: ' . implode(', ', $errors));
        }

        $data = $soldier->toArray();

        // Add system fields for new records
        if (!$soldier->getId()) {
            $data['status'] = $data['status'] ?? 'draft';
            $data['created_by'] = $data['created_by'] ?? 1;
        }

        $id = $this->saveFromArray('soldiers', $data);
        return $this->findById($id);
    }
}
```

**BAD - Hardcoded columns (breaks when new fields added):**

```php
// DON'T DO THIS - requires manual updates when model changes
public function update(Soldier $soldier): bool
{
    $sql = "UPDATE soldiers SET
            first_name = ?,
            last_name = ?,
            rank = ?,
            // ... every column listed manually ...
            WHERE id = ?";

    $params = [
        $soldier->getFirstName(),
        $soldier->getLastName(),
        $soldier->getRank(),
        // ... every getter manually ...
        $soldier->getId()
    ];

    return $this->execute($sql, $params) > 0;
}
```

### Backward Compatibility

Keep deprecated wrappers for existing code:

```php
/**
 * @deprecated Use save() instead
 */
public function create(Soldier $soldier): ?Soldier
{
    if ($soldier->getId()) {
        throw new \InvalidArgumentException('Cannot create with existing ID');
    }
    return $this->save($soldier);
}

/**
 * @deprecated Use save() instead
 */
public function update(Soldier $soldier): bool
{
    if (!$soldier->getId()) {
        throw new \InvalidArgumentException('Cannot update without ID');
    }
    return $this->save($soldier) !== null;
}
```

## Model Requirements

Models must implement `toArray()` that returns all persistable fields:

```php
class Soldier
{
    // ... properties and getters/setters ...

    /**
     * Convert to array for database persistence
     * This is the source of truth for what fields get saved
     */
    public function toArray(): array
    {
        return [
            'id' => $this->id,
            'first_name' => $this->firstName,
            'last_name' => $this->lastName,
            'military_rank' => $this->rank,
            'state' => $this->state,
            'dar_id' => $this->darId,
            'findagrave_id' => $this->findagraveId,  // New field - automatically included!
            'biographical_notes' => $this->biographicalNotes
        ];
    }
}
```

## Adding New Fields Checklist

When adding a new field to a model:

1. **Add database column** (migration)
   ```sql
   ALTER TABLE soldiers ADD COLUMN findagrave_id VARCHAR(20) NULL;
   ```

2. **Update Model class**
   - Add private property
   - Add getter method
   - Add setter method
   - Add to `toArray()` return array
   - Add to `hydrate()` method

3. **That's it!** - Repository automatically includes new field

No need to update repository INSERT/UPDATE statements if using the dynamic pattern.

## Fetch Methods

DataAccessObject provides null-safe fetch methods:

```php
// Single row (returns null, not false)
$row = $this->query($sql, $params)->fetch();

// Single column value
$count = $this->query("SELECT COUNT(*) FROM users")->fetchColumn();

// All rows
$rows = $this->query($sql, $params)->fetchAll();
```

## Transaction Support

```php
try {
    $this->beginTransaction();

    // Multiple operations...
    $this->saveFromArray('soldiers', $soldierData);
    $this->saveFromArray('soldier_sources', $sourceData);

    $this->commit();
} catch (\Exception $e) {
    $this->rollback();
    throw $e;
}
```

## Implementation in Projects

### WXING
- `includes/Classes/DataAccessObject.php` - Base class with helpers
- `includes/Repositories/*Repository.php` - All repositories

### littletalks-admin
- Similar structure, adapt namespace as needed

### football
- Similar structure, adapt namespace as needed

## Migration from Hardcoded to Dynamic

If you have existing repositories with hardcoded columns:

1. Add the dynamic SQL builder methods to your DataAccessObject
2. Create a `save()` method using `saveFromArray()`
3. Mark `create()` and `update()` as `@deprecated`
4. Update calls over time to use `save()`
