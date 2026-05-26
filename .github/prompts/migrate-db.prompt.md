---
description: "Generate Alembic database migration for Flask-SQLAlchemy model changes"
---

# Database Migration Generator

Generate an Alembic migration for the current model changes.

## Steps

1. **Analyze model changes** - Compare current SQLAlchemy models against existing migrations
2. **Generate migration** - Create a new Alembic revision with upgrade/downgrade functions
3. **Review SQL** - Show the raw SQL that will be executed
4. **Provide commands** - Output the PowerShell commands to apply the migration

## Migration Template

```python
"""${message}

Revision ID: ${revision}
Revises: ${down_revision}
Create Date: ${create_date}
"""
from alembic import op
import sqlalchemy as sa

revision = '${revision}'
down_revision = ${down_revision}
branch_labels = None
depends_on = None

def upgrade():
    ${upgrade_operations}

def downgrade():
    ${downgrade_operations}
```

## Commands to Run

```powershell
# Generate migration (after reviewing)
flask db migrate -m "${message}"

# Apply migration
flask db upgrade

# Rollback if needed
flask db downgrade
```

## User Input

What model changes should I create a migration for? Describe the new tables, columns, or modifications needed.
