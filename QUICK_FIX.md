# Quick Fix Guide

## **Problem**: "Could not find the table 'public.room_stats'"

### **Immediate Actions** (do these in order):

#### 1. Open Debug Tool 🔍
```
Open: debug.html in your browser
```
This will tell you exactly what's wrong.

---

#### 2. Most Likely Cause: Column Name Mismatch

**Backend uses**: `person_count`  
**Frontend expects**: `people_count` OR `person_count` (now auto-detects)

**The latest code update should handle this automatically**, but if it doesn't:

**Check your Supabase table structure:**
1. Go to Supabase Dashboard → Table Editor
2. Click on your table
3. Look at column names

**If column is named `person_count`:**
✅ The code now handles this automatically (just updated)

**If column is named `people_count`:**
✅ Also handled automatically

---

#### 3. If Table Doesn't Exist

**Run this SQL** in Supabase SQL Editor:

```sql
CREATE TABLE room_stats (
  id BIGSERIAL PRIMARY KEY,
  room_id TEXT NOT NULL,
  timestamp TIMESTAMPTZ NOT NULL,
  person_count INT NOT NULL,  -- Note: backend uses person_count
  
  CONSTRAINT person_count_non_negative CHECK (person_count >= 0)
);

-- Add indexes
CREATE INDEX idx_room_stats_room_id ON room_stats(room_id);
CREATE INDEX idx_room_stats_timestamp ON room_stats(timestamp DESC);

-- Enable RLS and add policy
ALTER TABLE room_stats ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Allow public read access" 
ON room_stats FOR SELECT 
TO anon 
USING (true);
```

---

#### 4. Add Test Data

```sql
INSERT INTO room_stats (room_id, timestamp, person_count)
VALUES 
  ('lobby', NOW(), 12),
  ('room_101', NOW(), 5),
  ('room_102', NOW(), 0);
```

---

#### 5. Verify RLS Policy

**Go to**: Supabase Dashboard → Authentication → Policies

**Ensure**: There's a SELECT policy for the `anon` role on `room_stats`

**If missing**, run:
```sql
CREATE POLICY "Public read access" 
ON room_stats FOR SELECT 
TO anon 
USING (true);
```

---

### **Changes Made to Fix Your Issue:**

1. ✅ **Added favicon** - Fixes Safari .ico warning
2. ✅ **Auto-detects column name** - Works with both `person_count` and `people_count`
3. ✅ **Better error messages** - Tells you what to do
4. ✅ **Debug tool** - `debug.html` diagnoses issues automatically
5. ✅ **Documentation** - `TROUBLESHOOTING.md` has full details

---

### **Test Your Fix:**

1. Hard refresh: `Cmd+Shift+R` (Mac) or `Ctrl+F5` (Windows)
2. Check console: Should see no errors
3. Check status: Should say "Connected • X room(s)"
4. If still broken: Open `debug.html`

---

### **Files to Check:**

| File | What to Check |
|------|---------------|
| `script.js` line 16 | `SUPABASE_URL` is correct |
| `script.js` line 17 | `SUPABASE_ANON_KEY` is correct |
| `script.js` line 19 | `TABLE_NAME` matches Supabase |
| Supabase Dashboard | Table exists with correct name |
| Supabase Dashboard | RLS policy allows SELECT |

---

### **Expected Result:**

After fixing, you should see:
- ✅ Green "Connected • 3 room(s)" (or however many rooms you have)
- ✅ Room cards displaying with counts
- ✅ No console errors (except maybe CSS warnings)
- ✅ Timestamps showing "Xs ago" or "Xm ago"

---

### **Still Not Working?**

1. Open `debug.html` - it will show exactly what's wrong
2. Take screenshot of the debug output
3. Check `TROUBLESHOOTING.md` for detailed solutions
4. Verify backend is actually writing data to Supabase

---

*Updated: November 26, 2025 - With auto-detection for column names*
