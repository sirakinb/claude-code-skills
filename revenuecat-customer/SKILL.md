---
name: revenuecat-customer
description: "Create a new customer in RevenueCat using a Firebase UID. Use when granting promotional/free access to users who haven't gone through the paywall. Triggers on: create revenuecat customer, add customer to revenuecat, grant free access, promotional access, free subscription."
---

# RevenueCat Customer Creation

Create customers in RevenueCat using their Firebase UID so you can grant them promotional entitlements.

---

## The Job

1. Get the Firebase UID from the user
2. Run the API call to create the customer
3. Direct user to RevenueCat dashboard to grant the promotional entitlement

---

## Step 1: Get Firebase UID

Ask the user for the Firebase UID of the person they want to grant access to.

The UID can be found in:
- Firebase Console → Authentication → Users → find their email → copy UID

---

## Step 2: Create the Customer

Run this curl command, replacing `{FIREBASE_UID}` with the actual UID:

```bash
curl --request GET \
  --url https://api.revenuecat.com/v1/subscribers/{FIREBASE_UID} \
  --header 'Authorization: Bearer sk_GbznGlptzOjGEIfkbBENxmeCCHcPk' \
  --header 'Content-Type: application/json'
```

**Note:** The GET endpoint creates the customer if they don't exist.

A successful response looks like:
```json
{
  "subscriber": {
    "entitlements": {},
    "original_app_user_id": "the-firebase-uid",
    "subscriptions": {}
  }
}
```

---

## Step 3: Grant Entitlement in Dashboard

After creating the customer, instruct the user to:

1. Go to RevenueCat Dashboard → **Customers**
2. Search for the Firebase UID
3. Click on their profile
4. Click **"Grant Promotional Entitlement"**
5. Select the entitlement and duration

---

## Notes

- Customer lists in the dashboard refresh every ~2 hours, but search works immediately
- If user signs in with Apple vs Email, they get different Firebase UIDs - may need to create both
- The promotional entitlement will show as `rc_promo` prefix in the dashboard
