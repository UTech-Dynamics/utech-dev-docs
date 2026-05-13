## Rails Testing Reference

This reference summarizes the testing patterns used across the repository.

Use it together with [Rails Model Spec Standards](../model-spec-standards.md) and the more detailed [FactoryBot guide](factorybot_build_vs_create.md).

### 1. build vs create (FactoryBot)

| Feature | `build` | `create` |
| --- | --- | --- |
| Persistence | In-memory only | Saves to DB |
| Triggers DB validations | No | Yes |
| Associations | May be lazy or incomplete | Fully persisted |
| Use case | Validation checks, in-memory tests | DB constraints, uniqueness, foreign keys, cascade checks |

Use `build` for fast model validation tests and `create` when the test depends on persistence, IDs, or database constraints.

### 2. Behavioral Validations

Write conditional validations in models with helper methods, for example:

```ruby
validates :discount_percentage, presence: true, if: :percentage?
validates :discount_price, presence: true, if: :fixed_amount?
```

Use `be_invalid` in RSpec for clarity:

```ruby
expect(product).to be_invalid
```

Print errors when a failure needs more detail:

```ruby
puts product.errors.full_messages
puts product.variants.map(&:errors).inspect
```

### 3. Nested Attributes

Accept nested attributes in parent model:

```ruby
accepts_nested_attributes_for :variants, allow_destroy: true
accepts_nested_attributes_for :product_images, allow_destroy: true
```

Test nested updates:

```ruby
expect do
  product.update!(variants_attributes: [variant_attributes])
end.to change { product.reload.variants.count }.by(1)
```

Always .reload parent to reflect DB state.

### 4. Association Testing

Prefer testing behavior instead of reflection metadata:

```ruby
parent = create(:category)
child = create(:category, parent: parent)
expect(parent.children).to include(child)
expect(child.parent).to eq(parent)
```

If you need the association declaration itself, keep that coverage minimal and avoid coupling tests to implementation details like `options[:dependent]`.

### 5. Test Sequencing Recommendations

- Test model validations and associations first.
- Test nested attributes and dependent behaviors.
- Test database constraints (uniqueness, foreign keys) separately.
- Test controller endpoints only after domain models are stable.
- Use failures as learning tools: write tests that fail first, then implement logic.
