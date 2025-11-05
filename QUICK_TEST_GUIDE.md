# Quick Test Guide - Trial Feedback Counter

## ✅ What Has Been Fixed

1. **Database Schema**: All tables created and migrations applied
2. **Enhanced Trigger**: Added comprehensive logging and error handling
3. **Security**: Fixed RLS policies and permissions
4. **Diagnostics**: Created logging table to track every trigger execution

## 🧪 How to Test

### Step 1: Register a Business
1. Go to your app's Business Registration page
2. Create a new business
3. Note the business ID or short name

### Step 2: Submit Customer Feedback
1. Navigate to the feedback form for your business (e.g., `/feedback/your-business-name`)
2. Submit a customer feedback response with ratings
3. The feedback should save successfully

### Step 3: Check the Counter
Run this query in Supabase SQL Editor:

```sql
-- Check if counter incremented
SELECT
    business_name,
    subscription_plan,
    trial_feedback_count,
    trial_notification_sent
FROM businesses
ORDER BY created_at DESC;
```

### Step 4: View Trigger Logs (Debugging)
If the counter didn't increment, check the logs:

```sql
-- See what happened during trigger execution
SELECT
    created_at,
    business_id,
    subscription_plan,
    old_count,
    new_count,
    success,
    error_message
FROM trigger_execution_log
ORDER BY created_at DESC
LIMIT 10;
```

## 🔍 What to Look For

### ✅ Success Scenario
- **trigger_execution_log**: `success = true`, `new_count = old_count + 1`
- **businesses.trial_feedback_count**: Incremented by 1
- **No error_message** in the log

### ❌ Common Issues

| Error Message | Cause | Solution |
|---------------|-------|----------|
| `business_id is NULL` | App not passing business ID | Check URL parameters and code |
| `Business does not exist` | Business not registered | Register business first |
| `Not on free_trial plan` | Wrong subscription plan | Change to 'free_trial' in businesses table |
| `Trial limit reached` | Already at 10 feedback | Expected behavior - limit reached |

## 🎯 Expected Behavior

1. **First feedback**: `trial_feedback_count` = 0 → 1
2. **Second feedback**: `trial_feedback_count` = 1 → 2
3. **...continues until 10**
4. **After 10th feedback**: Counter stops at 10, no further increments

## 📊 Monitoring Query

Use this comprehensive query to see everything at once:

```sql
SELECT
    b.business_name,
    b.subscription_plan,
    b.trial_feedback_count,
    COUNT(cf.id) as actual_feedback_count,
    (SELECT COUNT(*) FROM trigger_execution_log WHERE business_id = b.id AND success = true) as successful_increments,
    (SELECT COUNT(*) FROM trigger_execution_log WHERE business_id = b.id AND success = false) as failed_increments
FROM businesses b
LEFT JOIN customer_feedback cf ON b.id = cf.business_id
GROUP BY b.id, b.business_name, b.subscription_plan, b.trial_feedback_count;
```

## 🚀 Next Steps

1. Test the flow end-to-end
2. If counter still doesn't increment, check the trigger logs
3. Share the results from the monitoring query above for further assistance

The trigger now has complete visibility into its execution, making it easy to identify any remaining issues!
