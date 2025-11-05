# Trigger Diagnostics Guide

## Issue Fixed
The database trigger has been updated with comprehensive logging to diagnose why `trial_feedback_count` wasn't incrementing.

## What Was Changed

1. **Enhanced Trigger Function**: Added detailed logging at every step
2. **Logging Table**: Created `trigger_execution_log` to track all trigger executions
3. **Security**: Added `SECURITY DEFINER` to ensure trigger has proper permissions
4. **Validation**: Added checks for NULL business_id and missing businesses

## How to Diagnose

### Step 1: Submit Customer Feedback
Submit feedback through your app as normal.

### Step 2: Check Trigger Execution Logs
Run this query in your Supabase SQL Editor:

```sql
-- View all trigger execution attempts
SELECT
    created_at,
    trigger_name,
    business_id,
    subscription_plan,
    old_count,
    new_count,
    success,
    error_message
FROM trigger_execution_log
ORDER BY created_at DESC
LIMIT 20;
```

### Step 3: Verify Business and Feedback Data
```sql
-- Check businesses and their trial counts
SELECT
    b.id,
    b.business_name,
    b.subscription_plan,
    b.trial_feedback_count,
    COUNT(cf.id) as actual_feedback_count
FROM businesses b
LEFT JOIN customer_feedback cf ON b.id = cf.business_id
GROUP BY b.id, b.business_name, b.subscription_plan, b.trial_feedback_count;
```

### Step 4: Check Recent Customer Feedback
```sql
-- View recent feedback submissions
SELECT
    cf.id,
    cf.business_id,
    cf.session_id,
    cf.created_at,
    b.business_name,
    b.subscription_plan,
    b.trial_feedback_count
FROM customer_feedback cf
LEFT JOIN businesses b ON cf.business_id = b.id
ORDER BY cf.created_at DESC
LIMIT 10;
```

## Common Issues and Solutions

### Issue 1: business_id is NULL
**Symptom**: Trigger log shows `error_message: 'business_id is NULL'`

**Solution**: The application isn't passing business_id when creating feedback. Check:
- The URL contains the correct business ID parameter
- `createFeedbackSession` is called with `businessId`
- `saveCustomerFeedback` is called with `businessId`

### Issue 2: Business does not exist
**Symptom**: Trigger log shows `error_message: 'Business does not exist'`

**Solution**: The business hasn't been created yet. Register a business first through the Business Registration page.

### Issue 3: Not on free_trial plan
**Symptom**: Trigger log shows `error_message: 'Not on free_trial plan'`

**Solution**: The business subscription_plan is not 'free_trial'. The trigger only increments for free trial businesses.

### Issue 4: Trial limit reached
**Symptom**: Trigger log shows `error_message: 'Trial limit reached'`

**Solution**: The business has already received 10 feedback submissions (the trial limit). This is expected behavior.

### Issue 5: RLS Policy Blocking
**Symptom**: No entries in trigger_execution_log at all

**Solution**: RLS might be blocking the trigger. The new trigger uses `SECURITY DEFINER` which should resolve this, but if issues persist, check RLS policies.

## Expected Behavior

When everything works correctly:
1. Customer submits feedback
2. Trigger fires immediately after INSERT
3. Trigger logs execution to `trigger_execution_log` with `success: true`
4. `trial_feedback_count` increments by 1 in the `businesses` table
5. When count reaches 10, no further increments occur

## Next Steps

1. Register a business through your app (if not already done)
2. Submit customer feedback for that business
3. Run the diagnostic queries above
4. Share the results if the issue persists

The trigger will now provide detailed information about why it's not incrementing, making it easy to identify and fix the root cause.
