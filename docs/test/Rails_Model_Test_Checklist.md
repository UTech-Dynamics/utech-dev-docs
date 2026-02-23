✅ Rails Model Test Checklist (API + PostgreSQL)
1️⃣ Database-Level Integrity (Most Important)

These must always exist in DB and be tested when critical.

☐ NOT NULL constraints tested

Column has null: false

Spec verifies invalid without attribute

Example:

expect(build(:brand, name: nil)).not_to be_valid
☐ UNIQUE constraint tested

Model validation present

Unique index exists

Case sensitivity matches DB type (citext vs string)

Optional senior-level:

expect {
  Model.insert(...)
}.to raise_error(ActiveRecord::RecordNotUnique)
☐ Default values tested

If DB has:

t.boolean :active, default: true, null: false

You must test:

expect(Model.new.active).to eq(true)
☐ Foreign key constraints exist (if applicable)

Check:

add_foreign_key

null: false if required

Test behavior if association is mandatory.

2️⃣ Model Validations

For every validated attribute:

☐ Presence
☐ Uniqueness
☐ Length
☐ Numericality
☐ Format
☐ Inclusion / Exclusion
☐ Custom validation logic

Each rule must have:

Valid case

Invalid case

3️⃣ Associations

For each association:

☐ Association type tested

belongs_to

has_many

has_one

has_many :through

☐ dependent: behavior tested

Critical cases:

Option	Must Test
:destroy	Children removed
:nullify	FK becomes null
:restrict_with_exception	Raises error
:restrict_with_error	Adds error

Example (your case):

expect { brand.destroy }
  .to raise_error(ActiveRecord::DeleteRestrictionError)
4️⃣ Business Rules

Anything domain-specific must be tested.

Examples:

Cannot delete brand if products exist

Product price must be positive

Admin cannot deactivate themselves

Order total must match sum of items

If it is business logic, it must have a spec.

5️⃣ Scopes

For every scope:

☐ Returns correct records
☐ Does not return unintended records
☐ Chainable

Example:

expect(Brand.active).to include(active_brand)
expect(Brand.active).not_to include(inactive_brand)
6️⃣ Callbacks

If model has:

before_validation

before_save

after_commit

etc.

You must test:

☐ It runs
☐ It produces expected side effect
☐ It does not break data integrity
7️⃣ Methods (Instance + Class)

Every public method must have:

☐ Unit test
☐ Edge case test
☐ Failure case test

Example:

describe '#full_name' do
  it 'combines first and last name'
end
8️⃣ Schema Namespace Safety (Important in Your Setup)

Because you use:

create_table "salehobe.brands"

Ensure:

☐ schema_search_path correct in:

development

test

production

☐ Specs fail if schema misconfigured (optional defensive test)

You already experienced this pitfall earlier — so this belongs in your checklist permanently.

9️⃣ Factory Safety

For every factory:

☐ Produces valid object
expect(build(:brand)).to be_valid
☐ Does not rely on hardcoded duplicates

Use sequences for unique fields.

🔟 Security-Sensitive Models (Admin, Auth)

For models like Admin:

☐ Password digest exists
☐ Cannot save without password
☐ Token purpose validated
☐ Expiry logic tested
☐ Authorization edge cases tested

You’re already doing this properly in your auth specs.

🔎 Advanced (Senior-Level Discipline)

Optional but strong practice:

☐ DB constraint tested via raw insert
☐ Test that validation error messages are correct (if API depends on them)
☐ Race-condition safety for uniqueness
☐ Transaction rollback behavior tested
📌 Minimal Rule to Never Miss Anything

For every new model, ask:

What can break at DB level?

What can break at business logic level?

What should never be allowed?

What side effects happen?

What happens on delete?

If each has a spec → you are safe.