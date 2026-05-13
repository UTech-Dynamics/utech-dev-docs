Rails Model Spec Standards

UTech Dynamics – Engineering Guidelines

1. Purpose

This document defines the standard structure and philosophy for writing model specs in Rails applications at Utech Dynamics.

Goals:

Ensure consistent test structure across repositories

Separate validation, behavior, and database integrity concerns

Avoid brittle tests tied to Rails internals

Enforce domain correctness and data integrity

Improve long-term maintainability

2. Core Testing Philosophy

Model specs must test:

Domain behavior

Business rules (validations)

Data integrity guarantees (DB constraints)

Model specs must NOT test:

Rails DSL internals (e.g., reflection-only tests)

Framework implementation details

Redundant logic already guaranteed by Rails itself

3. Required Structure for Model Specs

All model specs must follow this structure:

RSpec.describe ModelName, type: :model do
  describe 'associations' do
  end

  describe 'validations' do
  end

  describe 'database constraints' do
  end
end

Order is intentional:

Associations → structural behavior

Validations → business rules

Database constraints → integrity guarantees

4. Associations — Behavioral Testing Only
❌ Do NOT Test Reflection Metadata

Avoid:

Category.reflect_on_association(:parent)

This tests Rails configuration, not behavior.

✅ Test Real Behavior

Example:

it 'allows parent-child relationship' do
  parent = create(:category)
  child = create(:category, parent: parent)

  expect(child.parent).to eq(parent)
  expect(parent.children).to include(child)
end

This verifies:

Association works

Foreign key is correct

Domain behavior is correct

Dependent Behavior

Test deletion behavior explicitly:

expect { parent.destroy }
  .to raise_error(ActiveRecord::DeleteRestrictionError)

Avoid inspecting:

association.options[:dependent]

Behavior > DSL.

5. Validations — Model-Level Business Rules
5.1 Use build for Validation Testing

Preferred pattern:

create(:category, name: 'Existing')

duplicate = build(:category, name: 'existing')

expect(duplicate).to be_invalid
expect(duplicate.errors[:name]).to include('has already been taken')
Why build?

Fast

Tests model validation layer

No exceptions

Clean intent

5.2 When to Use create!

Use create! only when testing that an exception is raised:

expect {
  create!(:category, name: nil)
}.to raise_error(ActiveRecord::RecordInvalid)

Do not mix exception testing with validation matcher style.

5.3 Prefer Positive Matchers

Prefer:

expect(model).to be_invalid

Over:

expect(model).not_to be_valid

They are functionally identical, but positive matchers reduce cognitive load.

6. Database Constraints — Data Integrity Layer

Model validations are not enough.

Every critical data-integrity rule should also be covered at the database level, especially uniqueness and foreign-key constraints.

6.1 Correct Pattern for DB Constraint Testing

Always interpolate the table name:

INSERT INTO #{Model.table_name}

Never hardcode schema names.

Example
it 'enforces uniqueness at DB level' do
  create(:category, name: 'Unique')

  expect do
    ActiveRecord::Base.connection.execute <<-SQL.squish
      INSERT INTO #{Category.table_name} (name, created_at, updated_at)
      VALUES ('Unique', NOW(), NOW())
    SQL
  end.to raise_error(ActiveRecord::RecordNotUnique)
end

This ensures:

Protection against race conditions

Protection against bypassing Rails validations

True data integrity

Keep these tests focused. Do not duplicate every model validation in SQL; reserve DB-level checks for rules that must hold even if Rails is bypassed.

7. Using Associations Instead of Foreign Keys

Prefer:

category.parent = category

Over:

category.parent_id = category.id

Reason:

Expresses domain intent

Avoids coupling tests to implementation details

Uses ActiveRecord API properly

Direct foreign key manipulation should be avoided in specs unless explicitly testing low-level behavior.

8. Test Data Strategy
8.1 Use FactoryBot Sequences

Factories must avoid hardcoded duplicates:

sequence(:name) { |n| "Category #{n}" }

This prevents accidental uniqueness collisions.

8.2 Avoid Overusing create

Guideline:

Scenario	Use
Validation test	build
Association behavior	create
Exception testing	create!
DB constraint test	Raw SQL
9. Ignoring Rails Internals

Avoid tests that check:

options[:class_name]

options[:foreign_key]

inverse_of

optional: true

These are implementation details.

If behavior is correct, the DSL is usually correct.

For broader test placement guidance, see [Testing Roadmap](architecture/testing_roadmap.md).

10. Self-Referential Model Testing Pattern

For self-referencing models:

it 'is invalid if parent references itself' do
  category = create(:category)
  category.parent = category

  expect(category).to be_invalid
  expect(category.errors[:parent]).to include('cannot reference itself')
end

Do not rely on manipulating IDs manually.

11. What Belongs in Model Specs vs Other Specs
Concern	Spec Type
Validation rules	Model spec
Association behavior	Model spec
Domain logic	Model spec
Query scopes	Model spec
API behavior	Request spec
Service orchestration	Service spec
Authorization	Policy spec

Keep boundaries clean.

12. Commit Message Standard for Test Changes

Use:

test(model): concise description

Examples:

test(category): refactor association behavior test
test(category): add DB-level uniqueness constraint test
test(category): standardize spec structure

Do NOT use:

perf (unless performance change)

opt

chore (unless maintenance only)

refactor (unless changing implementation)

Test changes → always test

13. Golden Rules

Test behavior, not Rails internals.

Separate validation layer from DB constraint layer.

Use build for validation tests.

Use create! only for exception testing.

Always test critical uniqueness at DB level.

Use associations instead of foreign key manipulation.

Keep spec structure consistent across repositories.

14. Quality Standard

A production-grade model spec must:

Cover valid case

Cover invalid cases

Cover edge cases

Cover DB-level protection

Avoid reflection-only tests

Follow structural order

Be deterministic and readable