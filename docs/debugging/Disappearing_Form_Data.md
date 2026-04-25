# 📝 The "Disappearing Data" Trap

## 🚩 The Problem
You fill out a form, hit submit, and get a validation error saying a field "can't be blank"—even though you definitely filled it in.

## 🕵️ The Investigation
When this happens, check your Rails server logs. You will likely see:

  1. The Data exists: Parameters: { "business" => { "address" => "Dhaka", ... } }

  2. The Warning: Unpermitted parameter: :address

  3.  The Result: Completed 422 Unprocessable Content

## 💡 The Root Cause: Strong Parameters
Rails uses Strong Parameters as a security wall. If you don't explicitly "invite" a field into the controller, Rails silently deletes it from the data hash before it ever reaches the Model.

## ✅ The Fix
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

## 🚀 Key Takeaway for the Team

**If the logs show the data but the model says it's blank, check your Strong Params in the Controller immediately!**
