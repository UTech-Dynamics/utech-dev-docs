## FactoryBot: build vs create in Rails

Choosing the wrong FactoryBot method can lead to false positives or tests that accidentally depend on the database.

### 1. `build`

```ruby
category = build(:category)
```

Creates a new Ruby object in memory.

Does not save to the database.

category.id is nil (no DB assignment yet).

Validations run only if you explicitly call category.valid? or category.save.

Associations depending on persisted IDs may not work as expected.

Fast and lightweight, ideal for pure model unit tests.

Use cases:

Testing in-memory model validations.

Avoiding database overhead in fast unit tests.

Cases where DB constraints or foreign keys are not required.

### 2. `create`

```ruby
category = create(:category)
```

Creates a new object AND saves it to the database.

category.id is assigned by the DB.

Validations are automatically run before save (create! raises an exception if invalid).

Database constraints (uniqueness, foreign keys) are enforced.

Associations requiring a persisted record work correctly.

Use cases:

Testing DB-level constraints: uniqueness, foreign keys.

Testing associations like parent-child.

Simulating real-world scenarios where the record must exist in the database.

### 3. Why it matters

Example: preventing a category from referencing itself as parent.

```ruby
# Using build (may not exercise the real constraint)
category = build(:category)
category.parent = category
category.valid?

# Using create (exercises the real constraint)
category = create(:category)
category.parent = category
category.valid?
```

If you rely on build for validations that depend on id or database constraints, the test can pass incorrectly.

### 4. Quick Reference Table

| Method   | ID assigned? | DB constraints enforced? | Speed  | Use case                                           |
| -------- | ------------ | ------------------------ | ------ | -------------------------------------------------- |
| `build`  | ❌           | ❌                       | Fast   | In-memory validation tests                         |
| `create` | ✅           | ✅                       | Slower | DB-level validation, associations, realistic tests |

### 5. Visual Flow Diagram

```
        +-------------------+
        | build(:model)     |
        +-------------------+
        | In-memory object  |
        | id = nil          |
        | DB not touched    |
        +-------------------+
                  |
                  v
       +----------------------+
       | validations optional |
       | associations limited |
       +----------------------+

        +-------------------+
        | create(:model)    |
        +-------------------+
        | Saved to DB       |
        | id assigned       |
        | validations run   |
        | DB constraints enforced |
        +-------------------+
                  |
                  v
       +--------------------------+
       | Realistic test behavior  |
       | Associations work        |
       | Foreign keys enforced    |
       +--------------------------+
```

### 6. Recommendation for Specs

Use build for fast unit tests that only validate attributes.

Use create for tests depending on IDs, database constraints, or associations.

Always verify self-referencing associations or DB-level uniqueness with create.


### 7. Practical Example: Self-Parent Validation

Correct way to test that a category cannot reference itself as parent:

```ruby
RSpec.describe Category, type: :model do
  it 'is invalid if parent references itself' do
    # Persisted category ensures the association is exercised realistically
    category = create(:category, name: 'SelfParent')

    # Assign itself as parent
    category.parent = category

    # Validation should fail
    expect(category).to be_invalid
    expect(category.errors[:parent]).to include('cannot reference itself')
  end
end
```
