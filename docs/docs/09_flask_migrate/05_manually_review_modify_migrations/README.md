---
title: Manually review and modify database migrations
description: Alembic can generate database migrations parting from model changes, but sometimes we need to modify them manually.
---

# Manually review and modify database migrations

## Default column values

When you add a column that uses a default value, any rows that existed previously will have `null` as the value.

You'll have to go into the database to add the default value to those rows.

You can also do this during the migration, since Alembic migrations can execute any arbitrary SQL queries.

Here's an example for a column being added with a default value:

```py title="migrations/versions/sample_migration.py"
from alembic import op
import sqlalchemy as sa


# revision identifiers, used by Alembic.
revision = "9c386e4052be"
down_revision = "713af8a4cb34"
branch_labels = None
depends_on = None


def upgrade():
    # ### commands auto generated by Alembic - please adjust! ###
    op.add_column(
        "invoices",
        sa.Column("enable_downloads", sa.Boolean(), nullable=True, default=False),
    )
    # ### end Alembic commands ###


def downgrade():
    # ### commands auto generated by Alembic - please adjust! ###
    op.drop_column("invoices", "enable_downloads")
    # ### end Alembic commands ###
```

You can see that we're adding a column called `enable_downloads` to the `invoices` table. The default value for new rows will be `False`, but what is the value for all the invoices we already have in the database?

The value will be undefined, or `null`.

What we must do is tell Alembic to insert `False` into each of the existing rows. We can do that with SQL:

```py title="migrations/versions/sample_migration.py"
from alembic import op
import sqlalchemy as sa


# revision identifiers, used by Alembic.
revision = "9c386e4052be"
down_revision = "713af8a4cb34"
branch_labels = None
depends_on = None


def upgrade():
    # ### commands auto generated by Alembic - please adjust! ###
    op.add_column(
        "invoices",
        sa.Column("enable_downloads", sa.Boolean(), nullable=True, default=False),
    )
    # highlight-start
    op.execute("UPDATE invoices SET enable_downloads = False")
    # highlight-end
    # ### end Alembic commands ###


def downgrade():
    # ### commands auto generated by Alembic - please adjust! ###
    op.drop_column("invoices", "enable_downloads")
    # ### end Alembic commands ###
```