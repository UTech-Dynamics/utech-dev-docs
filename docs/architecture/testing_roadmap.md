Testing Roadmap – Rails Project (General)

This document provides a recommended testing strategy for backend development, covering unit, database, integration, and end-to-end tests.

```
                   ┌───────────────────┐
                   │   Unit Tests       │
                   │  (Model-level)     │
                   │  Validations,      │
                   │  Methods, Callbacks│
                   └─────────┬─────────┘
                             │
                             ▼
                   ┌───────────────────┐
                   │  DB-level Checks   │
                   │  (Optional/Minimal)│
                   │  Unique indexes,   │
                   │  Foreign keys,     │
                   │  Defaults          │
                   └─────────┬─────────┘
                             │
                             ▼
                   ┌───────────────────┐
                   │ Module Integration │
                   │  (Controller +     │
                   │   Routes + Models)│
                   │  Success & Failure │
                   │  Scenarios         │
                   └─────────┬─────────┘
                             │
                             ▼
                   ┌───────────────────┐
                   │Cross-module        │
                   │Integration Tests   │
                   │ (Module Interactions│
                   │  e.g., Catalog → Order) │
                   └─────────┬─────────┘
                             │
                             ▼
                   ┌───────────────────┐
                   │ End-to-End /       │
                   │ System Tests       │
                   │  Full Workflow     │
                   │  (Optional)        │
                   └───────────────────┘
```

1. Unit Tests (Model-level)
Purpose

Validate individual model behavior and business rules.

Catch issues early before controller or integration logic.

Scope

Model validations (presence, numericality, associations).

Custom methods (e.g., Variant#effective_price).

Callbacks if they implement important logic.

Guidelines

Test all validations including conditional validations.

Use factory data (FactoryBot) to generate test objects.

Keep tests fast and isolated from the database where possible (though Rails unit tests typically use a test DB).

Example
describe Variant do
  it 'validates unit_price presence' do
    variant = build(:variant, unit_price: nil)
    expect(variant).to be_invalid
  end

  it 'calculates effective_price correctly for percentage discount' do
    variant = build(:variant, unit_price: 100, discount_type: :percentage, discount_percentage: 10)
    expect(variant.effective_price).to eq(90)
  end
end
2. Database-level Tests (Optional / Minimal)
Purpose

Ensure database constraints and indexes enforce invariants.

Catch issues that unit tests cannot detect, e.g., duplicate unique constraints, foreign key integrity.

Scope

Check unique indexes (e.g., SKU or variant codes).

Check foreign keys (variant → product, order_item → variant).

Test default values and DB-level numeric constraints.

Guidelines

Only test critical DB constraints not enforced by Rails validations.

Keep them minimal; most logic should be tested at the model layer.

Can use RSpec with ActiveRecord::Base.connection to insert invalid data to confirm DB raises errors.

Note: Do not write DB tests for every model validation — that is redundant with unit tests.

3. Module Integration Tests (Controller + Routes)
Purpose

Verify end-to-end behavior of a single module (e.g., catalog, orders).

Ensure routes, controller actions, and models work together.

Scope

Controller actions (index, show, create, update, destroy).

Response codes and JSON payloads.

Error handling with invalid data.

Guidelines

Use request specs or Rails integration tests.

Include both success and failure scenarios.

Use factories for test data.

Example
describe 'Variants API', type: :request do
  it 'creates a variant successfully' do
    post '/api/v1/variants', params: { variant: valid_attributes }
    expect(response).to have_http_status(:created)
    expect(json['unit_price']).to eq(valid_attributes[:unit_price])
  end
end
4. Cross-module Integration Tests
Purpose

Test interactions between modules (e.g., catalog → order).

Ensure workflow correctness when multiple modules depend on each other.

Scope

Placing an order using existing variants.

Stock updates after order creation.

Discounts applied correctly from catalog.

Guidelines

Run these tests after each module has its own integration tests passing.

Keep tests focused on business-critical flows rather than every edge case.

5. End-to-End / System Tests (Optional)
Purpose

Verify full business workflows, often including frontend (if applicable).

Used for QA, staging, or CI/CD validation.

Scope

Complete flows:

User creates order → order items → total calculation → stock update.

Admin creates product → variant → order workflow.

Guidelines

Use tools like RSpec system specs, Capybara, or Cypress.

Keep these tests minimal and high-level; do not replace unit or module integration tests.

6. Recommended Testing Flow

Unit tests → Run whenever model or validation changes.

DB-level checks → Run after migrations that alter schema.

Module integration tests → Run after implementing controller/routes for a module.

Cross-module integration tests → Run after multiple modules are stable.

End-to-end tests → Run in staging or pre-release environments.

7. Notes on DB-level tests

Include only critical constraints that cannot be enforced in Rails validations.

Examples: unique index, foreign key integrity, default values.

Avoid duplicating every model validation in DB tests.

Helps prevent silent corruption if Rails validations are bypassed (e.g., direct SQL inserts).

8. Documentation & Maintenance

Keep tests alongside code (spec/models, spec/requests, etc.).

Use contexts in RSpec to organize different scenarios (success vs failure).

Update integration tests whenever module behavior or API changes.

Document assumptions about data and business rules in a separate docs/architecture/ file.