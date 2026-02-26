Rails Testing Reference – Salehobe API

*This concise reference ensures that any Utech developer can quickly understand:*

- How build/create affect tests

- Behavioral validation rules

- Nested attributes handling

- Testing database constraints

- Proper sequencing of model → controller tests

1. build vs create (FactoryBot)
Feature	build	create
Persistence	In-memory only	Saves to DB
Triggers DB validations	No	Yes
Associations (has_many / belongs_to)	May be nil or lazy	Fully loaded in DB
Use case	Unit-level validations, in-memory tests	DB-level tests, uniqueness, foreign keys, cascade checks

Tip: Use build for fast validations; use create when testing persistence, uniqueness, or association constraints.

2. Behavioral Validations

Write conditional validations in models with helper methods, e.g.:

validates :discount_percentage, presence: true, if: :percentage?
validates :discount_price, presence: true, if: :fixed_amount?

Use be_invalid in RSpec for better DSL clarity:

expect(product).to be_invalid

Print errors when failing to understand why:

puts product.errors.full_messages
puts product.variants.map(&:errors).inspect
3. Nested Attributes

Accept nested attributes in parent model:

accepts_nested_attributes_for :variants, allow_destroy: true
accepts_nested_attributes_for :product_images, allow_destroy: true

Test nested updates:

expect do
  product.update!(variants_attributes: [variant_attributes])
end.to change { product.reload.variants.count }.by(1)

Always .reload parent to reflect DB state.

4. Association Testing

Use reflect_on_association to check macro and options:

association = Product.reflect_on_association(:variants)
expect(association.macro).to eq(:has_many)
expect(association.options[:dependent]).to eq(:destroy)

Prefer testing behavior (allows parent-child relationship) instead of raw foreign key:

parent = create(:category)
child = create(:category, parent: parent)
expect(parent.children).to include(child)
expect(child.parent).to eq(parent)
5. Test Sequencing Recommendations

Test model validations and associations first.

Test nested attributes and dependent behaviors.

Test database constraints (uniqueness, foreign keys) separately.

Test controller endpoints only after domain models are stable.

Use failures as learning tools: try writing tests that fail first, then implement logic.
