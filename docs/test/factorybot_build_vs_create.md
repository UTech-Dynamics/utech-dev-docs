FactoryBot: build vs create in Rails

When writing model specs in Rails using FactoryBot, it’s important to understand the difference between build and create. Choosing the wrong one can lead to false positives or unexpected behavior in validations and database tests.

1. build

```
category = build(:category)
```

Creates a new Ruby object in memory.

Does not save to the database.

category.id is nil (no DB assignment yet).

Validations run only if you explicitly call category.valid? or category.save.

Associations depending on persisted IDs (like parent_id) may not work as expected.

Fast and lightweight, ideal for pure model unit tests.

Use cases:

Testing in-memory model validations.

Avoiding database overhead in fast unit tests.

Cases where DB constraints or foreign keys are not required.

2. create

```
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

3. Why it matters

Example: Preventing a category from referencing itself as parent.

```
# Using build (fails silently because id is nil)
category = build(:category)
category.parent_id = category.id   # category.id is nil
category.valid?                    # validation may not trigger

# Using create (works correctly)
category = create(:category)
category.parent_id = category.id   # now category.id is real
category.valid?                    # triggers "cannot reference itself"
```

If you rely on build for validations that depend on id or database constraints, the test can pass incorrectly.

4. Quick Reference Table

```
| Method   | ID assigned? | DB constraints enforced? | Speed  | Use case                                           |
| -------- | ------------ | ------------------------ | ------ | -------------------------------------------------- |
| `build`  | ❌           | ❌                       | Fast   | In-memory validation tests                         |
| `create` | ✅           | ✅                       | Slower | DB-level validation, associations, realistic tests |

```

5. Visual Flow Diagram

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

6. Recommendation for Specs

Use build for fast unit tests that only validate attributes.

Use create for tests depending on IDs, database constraints, or associations.

Always verify self-referencing associations or DB-level uniqueness with create.


7. Practical Example: Self-Parent Validation

# Correct way to test that a category cannot reference itself as parent

```
RSpec.describe Category, type: :model do
  it 'is invalid if parent references itself' do
    # Persisted category ensures id is assigned
    category = create(:category, name: 'SelfParent')

    # Assign itself as parent
    category.parent = category

    # Validation should fail
    expect(category).not_to be_valid
    expect(category.errors[:parent]).to include('cannot reference itself')
  end
end
```
