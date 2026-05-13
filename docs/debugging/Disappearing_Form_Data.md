# The Disappearing Data Trap

## The Problem
You fill out a form, hit submit, and get a validation error saying a field "can't be blank"—even though you definitely filled it in.

## The Investigation
When this happens, check your Rails server logs. You will likely see:

1. The data exists: `Parameters: { "business" => { "address" => "Dhaka", ... } }`
2. The warning: `Unpermitted parameter: :address`
3. The result: `Completed 422 Unprocessable Content`

## The Root Cause: Strong Parameters

Rails uses Strong Parameters as a security wall. If you do not explicitly permit a field in the controller, Rails silently removes it before it reaches the model.

## The Fix
Update your controller to permit the new field:

```ruby
# app/controllers/businesses_controller.rb

private

def business_params
  # ⛔ WRONG: Missing :address
  # params.require(:business).permit(:name, :contact_email)

  # ✅ RIGHT: Explicitly permit :address
  params.require(:business).permit(:name, :contact_email, :contact_phone, :address)
end
```

## Key Takeaway for the Team

If logs show the data but the model says it is blank, check Strong Parameters in the controller first.
