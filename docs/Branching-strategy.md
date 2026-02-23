## Naming Convention:

Use:
```
<type>/<scope>-<objective>
```
Types:

-  feature/

-  refactor/

-  fix/

-  test/

-  chore/

-  hardening/

Clean Examples

If focusing on data integrity:

-  hardening/product-data-integrity

-  hardening/order-state-validation

-  hardening/user-constraints

-  refactor/product-domain-rules

-  test/order-request-specs

-  Avoid vague names like:

-  update-product

-  new-changes

-  db-update

-  final-fix

They lack semantic clarity.

-----

1️⃣ Branch Name Conventions

Prefix options (optional but recommended):

feature/ → new feature or domain modeling

refactor/ → structural change or redesign

db/ → database-focused changes

enhancement/ → minor improvement or new attributes

2️⃣ Descriptive Branch Name Ideas

feature/product-catalog-normalization

Explains clearly that the products table is normalized with brands/categories.

feature/product-lifecycle-management

Focuses on the status column and lifecycle states.

feature/brands-categories-entities

Emphasizes turning brand/category into separate entities.

db/product-catalog-refactor

Highlights that the migration/schema is changing at the DB layer.

feature/catalog-structure-redesign

Broad but descriptive; indicates full structural redesign.