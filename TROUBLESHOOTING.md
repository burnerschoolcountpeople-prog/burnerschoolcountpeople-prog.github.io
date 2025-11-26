# Troubleshooting Guide

## Current Issue: "Could not find the table 'public.room_stats'"

This error indicates that the Supabase table either doesn't exist or has a different name.

### Quick Diagnosis Steps

1. **Open the Debug Tool**
   - Navigate to `debug.html` in your browser
   - It will automatically test your connection and discover available tables
   - Review the output to identify the issue

2. **Check Your Supabase Dashboard**
   - Log in to https://supabase.com
   - Go to your project
   - Click "Table Editor" in the left sidebar
   - **Look for the table name** - it might be:
     - `room_stats` (expected)
     - `visitor_counts` (old version)
     - `detections` (alternative name)
     - Something else entirely

### Common Causes & Solutions

#### Cause 1: Table Doesn't Exist
**Symptom**: Error message says "Could not find the table"

**Solution**: Create the table in Supabase SQL Editor:

```sql
CREATE TABLE room_stats (
  id BIGSERIAL PRIMARY KEY,
  room_id TEXT NOT NULL,
  timestamp TIMESTAMPTZ NOT NULL,
  people_count INT NOT NULL,
  
  CONSTRAINT people_count_non_negative CHECK (people_count >= 0)
);

-- Add indexes for performance
CREATE INDEX idx_room_stats_room_id ON room_stats(room_id);
CREATE INDEX idx_room_stats_timestamp ON room_stats(timestamp DESC);
CREATE INDEX idx_room_stats_room_timestamp ON room_stats(room_id, timestamp DESC);
```

#### Cause 2: Wrong Table Name
**Symptom**: Backend uses a different table name

**Solution**: Update `script.js` line 14:

```javascript
// Change this:
TABLE_NAME: 'room_stats',

// To whatever your actual table name is (from debug tool):
TABLE_NAME: 'your_actual_table_name',
```

#### Cause 3: Column Name Mismatch
**Symptom**: Table exists but queries fail

**Solution**: According to the backend docs, the column might be named `person_count` instead of `people_count`. Update the query in `script.js`:

Find line ~102:
```javascript
.select('room_id, timestamp, people_count')
```

Change to:
```javascript
.select('room_id, timestamp, person_count')
```

And update the rendering code to use `person_count` instead of `people_count`.

#### Cause 4: Row Level Security (RLS) Blocking Access
**Symptom**: Table exists but anon key can't read it

**Solution**: Add a policy in Supabase Dashboard:

1. Go to **Authentication > Policies**
2. Find your `room_stats` table
3. Click **New Policy**
4. Choose **"Enable read access for all users"**
5. Or create a custom policy:

```sql
CREATE POLICY "Allow public read access" ON room_stats
FOR SELECT
TO anon
USING (true);
```

#### Cause 5: Wrong Supabase Credentials
**Symptom**: Connection succeeds but queries fail

**Solution**: Verify credentials in `script.js`:

1. Go to Supabase Dashboard > **Settings > API**
2. Copy your **Project URL** (starts with `https://`)
3. Copy your **anon/public key** (NOT the service_role key)
4. Update lines 16-17 in `script.js`

---

## Step-by-Step Fix Process

### Step 1: Use the Debug Tool
```bash
# Open debug.html in your browser
open debug.html
```

This will tell you:
- ✓ If Supabase connection works
- ✓ If the table exists
- ✓ What tables ARE available
- ✓ Sample data structure

### Step 2: Verify Table Exists
Go to **Supabase Dashboard > Table Editor**

If table **DOES NOT exist**:
```sql
-- Run this in Supabase SQL Editor
CREATE TABLE room_stats (
  id BIGSERIAL PRIMARY KEY,
  room_id TEXT NOT NULL,
  timestamp TIMESTAMPTZ NOT NULL,
  people_count INT NOT NULL
);
```

### Step 3: Check RLS Policies
Go to **Supabase Dashboard > Authentication > Policies**

Add this policy if missing:
```sql
CREATE POLICY "Public read access" 
ON room_stats FOR SELECT 
TO anon 
USING (true);
```

### Step 4: Verify Column Names Match
If backend inserts use `person_count`, update frontend to match:

**Backend column name**: Check your Supabase table structure
**Frontend query**: Update `script.js` line 103

### Step 5: Test with Sample Data
Insert a test record in Supabase SQL Editor:

```sql
INSERT INTO room_stats (room_id, timestamp, people_count)
VALUES ('test_room', NOW(), 5);
```

Refresh your website - you should see the test room appear.

---

## Backend/Frontend Column Name Mismatch Issue

Based on the backend repository, there's a known inconsistency:

**Backend code** (`supabase_utils.py` line 246):
```python
data = {
    "room_id": room_id,
    "timestamp": timestamp.isoformat(),
    "person_count": people_count  # Note: Column is person_count
}
```

**Frontend code** (`script.js` line 103):
```javascript
.select('room_id, timestamp, people_count')  // Wrong!
```

### Fix: Align Frontend with Backend

Option 1: **Change Frontend to Match Backend**
```javascript
// In script.js, change all instances:
people_count → person_count
```

Option 2: **Change Backend to Match Frontend**
```python
# In backend/utils/supabase_utils.py, change:
"person_count" → "people_count"
```

**Recommended**: Option 1 (change frontend) since backend is already deployed.

---

## Safari-Specific Issues

### Favicon 404 Error
**Not a critical issue**, but to fix:

1. Create a simple favicon or download one
2. Add to your repository root
3. Or add this to `<head>` in `index.html`:

```html
<link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>👥</text></svg>">
```

---

## Quick Validation Checklist

- [ ] Table exists in Supabase (check Table Editor)
- [ ] Table name matches `CONFIG.TABLE_NAME` in script.js
- [ ] Column names match between backend and frontend
- [ ] RLS policy allows SELECT for anon role
- [ ] Supabase URL and anon key are correct
- [ ] Sample data exists in the table
- [ ] No browser console errors (other than favicon)

---

## Testing Your Fix

After making changes:

1. **Hard refresh** your browser: `Cmd+Shift+R` (Mac) or `Ctrl+Shift+R` (Windows)
2. **Check Console**: Open Developer Tools > Console
3. **Expected**: Green "Connected • X room(s)" status
4. **If still failing**: Run debug.html and share the output

---

## Getting Help

If none of these steps work:

1. Open `debug.html` and take a screenshot of the output
2. Share your Supabase table structure (Table Editor screenshot)
3. Check if backend is actually writing data to Supabase
4. Verify RLS policies in Supabase Dashboard

---

*Last Updated: November 26, 2025*
