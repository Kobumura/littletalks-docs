# PHP Migration System

> **Used by**: littletalks-admin, WXING, football
>
> A simple, file-based SQL migration system for PHP projects deployed via GitHub Actions to Plesk.

## Overview

This system:
- Tracks executed migrations in a `_migrations` table
- Runs SQL files in numerical order
- Skips already-executed migrations
- Integrates with GitHub Actions for auto-deploy

## Directory Structure

```
your-project/
├── scripts/
│   └── migrations/
│       ├── run_migrations.php      # Migration runner
│       ├── 001_create_users.sql    # First migration
│       ├── 002_add_email_column.sql
│       └── 003_create_orders.sql
└── .github/
    └── workflows/
        └── deploy.yml              # Triggers migrations on deploy
```

## File Naming Convention

```
NNN_description.sql
```

- **NNN**: Three-digit number (001, 002, 003...)
- **description**: Snake_case description of what it does
- Files are sorted alphabetically, so numbering ensures order

**Examples:**
```
001_create_messaging_prices_table.sql
002_create_users_table.sql
003_add_dashboard_preferences.sql
```

## Migration File Format

```sql
-- Migration: 001_create_users_table
-- Description: Creates the users table
-- Database: your_database_name

CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

**Guidelines:**
- Always use `IF NOT EXISTS` for CREATE TABLE
- Include header comments for documentation
- Multiple statements OK (separated by `;`)
- Use `utf8mb4` charset for emoji support

## The `_migrations` Table

Created automatically on first run:

```sql
CREATE TABLE IF NOT EXISTS _migrations (
    id INT AUTO_INCREMENT PRIMARY KEY,
    filename VARCHAR(255) NOT NULL UNIQUE,
    executed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

This tracks which migrations have run. The runner checks this before executing each file.

## Migration Runner Script

**Location**: `scripts/migrations/run_migrations.php`

### Basic Version (WXING pattern)

```php
<?php
/**
 * Database Migration Runner
 * Usage: php scripts/migrations/run_migrations.php
 */

require_once __DIR__ . '/../../includes/config.php';

use YourNamespace\Classes\Database;

echo "===========================================\n";
echo "Database Migrations\n";
echo "===========================================\n\n";

try {
    $db = Database::getInstance()->getPDO();
    echo "✓ Database connection successful.\n\n";
} catch (Exception $e) {
    die("Database connection failed: " . $e->getMessage() . "\n");
}

// Get all migration files
$migrationDir = __DIR__;
$migrations = glob($migrationDir . '/*.sql');
sort($migrations);

if (empty($migrations)) {
    echo "No migration files found.\n";
    exit(0);
}

echo "Found " . count($migrations) . " migration file(s)\n\n";

// Create tracking table
$db->exec("
    CREATE TABLE IF NOT EXISTS _migrations (
        id INT AUTO_INCREMENT PRIMARY KEY,
        filename VARCHAR(255) NOT NULL UNIQUE,
        executed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
");

// Get executed migrations
$stmt = $db->query("SELECT filename FROM _migrations");
$executedFiles = $stmt->fetchAll(PDO::FETCH_COLUMN);

// Run pending migrations
$ran = 0;
foreach ($migrations as $file) {
    $filename = basename($file);

    if (in_array($filename, $executedFiles)) {
        echo "[SKIP] {$filename} - already executed\n";
        continue;
    }

    echo "[RUN]  {$filename}...\n";

    $sql = file_get_contents($file);
    $statements = array_filter(
        array_map('trim', explode(';', $sql)),
        fn($s) => !empty($s) && strpos($s, '--') !== 0
    );

    try {
        foreach ($statements as $statement) {
            if (!empty($statement)) {
                $db->exec($statement);
            }
        }

        $stmt = $db->prepare("INSERT INTO _migrations (filename) VALUES (?)");
        $stmt->execute([$filename]);

        echo "       [OK]\n";
        $ran++;
    } catch (PDOException $e) {
        echo "       [ERROR] " . $e->getMessage() . "\n";
        die("Migration failed. Stopping.\n");
    }
}

echo "\nMigration complete. {$ran} migration(s) executed.\n";
```

### With Safety Check (littletalks-admin pattern)

For projects with multiple databases, add a safety check to prevent running migrations against the wrong database:

```php
// SAFETY CHECK: Only allow migrations on specific database
$ALLOWED_DATABASE = 'your_admin_database';

if ($config['database'] !== $ALLOWED_DATABASE) {
    echo "⛔ SAFETY BLOCK: Wrong database!\n";
    echo "This migration runner is ONLY for: {$ALLOWED_DATABASE}\n";
    echo "You tried to run against: {$config['database']}\n";
    exit(1);
}

echo "✓ Safety check passed\n\n";
```

## GitHub Actions Integration

**File**: `.github/workflows/deploy.yml`

```yaml
name: Deploy to Production

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    name: 🚀 Deploy to Production

    steps:
      - name: 🛒 Checkout code
        uses: actions/checkout@v4

      - name: 🚀 Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.LIGHTSAIL_HOST }}
          username: ${{ secrets.LIGHTSAIL_USER }}
          key: ${{ secrets.LIGHTSAIL_SSH_KEY }}
          port: 22
          script: |
            # Pull latest code
            cd /path/to/your/project
            git fetch origin main
            git reset --hard FETCH_HEAD

            # Run migrations
            echo "Running database migrations..."
            PHP_PATH=$(which php || echo "/opt/plesk/php/8.2/bin/php")
            $PHP_PATH scripts/migrations/run_migrations.php

            echo "Deployed at $(date)"

      - name: ✅ Deployment complete
        run: echo "✅ Deployment successful!"
```

### Required GitHub Secrets

| Secret | Description |
|--------|-------------|
| `LIGHTSAIL_HOST` | Server IP or hostname |
| `LIGHTSAIL_USER` | SSH username |
| `LIGHTSAIL_SSH_KEY` | Private SSH key |
| `GH_PAT` | GitHub Personal Access Token (if private repo) |

## Plesk-Specific Notes

### File Permissions

Plesk manages file ownership. After git pull, you may need:

```bash
sudo chown -R username:psacln /path/to/project/.git
```

### PHP Path

Plesk installs PHP in non-standard locations:

```bash
# Find PHP
PHP_PATH=$(which php || echo "/opt/plesk/php/8.2/bin/php")
```

### Safe Directory

Git may complain about safe directories:

```bash
git config --global --add safe.directory /path/to/project
```

## Manual Execution

To run migrations manually (e.g., after creating a new migration):

```bash
# SSH to server
ssh user@your-server

# Navigate to project
cd /path/to/your/project

# Run migrations
php scripts/migrations/run_migrations.php
```

Or trigger a deploy by pushing to main.

## Creating a New Migration

1. Create new SQL file with next number:
   ```bash
   touch scripts/migrations/004_add_status_column.sql
   ```

2. Write your SQL:
   ```sql
   -- Migration: 004_add_status_column
   -- Description: Adds status column to orders table

   ALTER TABLE orders ADD COLUMN status VARCHAR(50) DEFAULT 'pending';
   ```

3. Commit and push:
   ```bash
   git add scripts/migrations/004_add_status_column.sql
   git commit -m "[PROJ-123] Add status column to orders"
   git push
   ```

4. GitHub Actions runs → migrations execute automatically

## Troubleshooting

### Migration shows as "already executed" but table doesn't exist

The migration was recorded in `_migrations` but failed partway through. Fix:

```sql
DELETE FROM _migrations WHERE filename = '004_add_status_column.sql';
```

Then re-run migrations.

### "Duplicate column name" errors

Migration ran partially. Either:
1. Use `IF NOT EXISTS` in your SQL
2. Delete from `_migrations` and fix the migration file
3. Use the WXING pattern that handles "already exists" errors gracefully

### Permission denied on git pull

Plesk ownership issue:
```bash
sudo chown -R username:psacln /path/to/project/.git
```

---

*This pattern is intentionally simple. For complex projects, consider a proper migration framework like Doctrine Migrations or Phinx.*
