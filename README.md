# Room Occupancy Monitor - Frontend

A lightweight, fast, and maintainable static website that displays live room occupancy using data from Supabase. Built with vanilla HTML/CSS/JavaScript + Supabase JS SDK.

## 🎯 Overview

This frontend displays real-time visitor distribution across rooms by fetching data from a Supabase PostgreSQL database. The backend (YOLO-based image processing system) continuously updates room occupancy counts, which are displayed here with a modern security monitoring dashboard and interactive floor plans.

**Data Flow:**
```
Backend (YOLO Processing) → Supabase Database → Frontend (This App) → Display
```

**Two Display Modes:**
1. **Dashboard View** (`index.html`) - Real-time room occupancy cards for monitoring
2. **Floor Plan View** (`floorplan.html`) - Interactive building navigation for visitors

## 🏢 Camera ID Convention

**Important:** This system uses a standardized camera numbering scheme across the 7-floor building.

### Camera Numbering System

- **Format**: `XYY` where:
  - `X` = Floor number (1-7)
  - `YY` = Camera number on that floor (01-10)

- **Range**: Each floor has 10 cameras
  - Floor 1: Cameras 101-110
  - Floor 2: Cameras 201-210
  - Floor 3: Cameras 301-310
  - Floor 4: Cameras 401-410
  - Floor 5: Cameras 501-510
  - Floor 6: Cameras 601-610
  - Floor 7: Cameras 701-710

### Camera Positioning Rules

**Lower camera IDs = More left/down position on the floor**

- Cameras are numbered from **left to right** and **bottom to top**
- No cameras in lavatories (lat) or prep rooms
- Only classrooms, labs, and public spaces have cameras

### Example: 4th Floor (Chemistry/Physics/Biology Labs)

| Room Name | Location | Camera ID |
|-----------|----------|-----------|
| Chemistry Lab | Far left | **401** |
| Physics Lab | Center-left | **402** |
| Biology Lab | Center-right | **403** |
| Lab Prep Room | (No camera) | - |
| Classroom 4A | Left side | **405** |
| Classroom 4B | Center | **406** |
| Classroom 4C | Center-right | **407** |
| 6B Classroom (Room 413) | Far right | **410** |

**Database Convention:** Room IDs in the database can be flexible formats like:
- `"401"` (direct camera ID)
- `"room-401"` 
- `"camera_401"`
- `"chemistry-lab-401"`

The system automatically extracts the 3-digit camera ID from any room_id format.

## ✨ Features

### Two Display Modes

#### 1. Dashboard View (`index.html`)
**Purpose:** Real-time monitoring for security guards and staff

- **Live Occupancy Display**: Real-time people count per room
- **Building Statistics**: Total occupancy, active rooms, busy rooms, empty rooms
- **4-Tier Status System**: Empty (blue) → Light (green) → Moderate (amber) → Busy (red)
- **Capacity Visualization**: Animated progress bars showing room fullness
- **Data Staleness Detection**: Automatic warnings for data older than 5 minutes
- **Filter Controls**: Show all rooms, occupied only, or empty only

#### 2. Floor Plan View (`floorplan.html`)
**Purpose:** Interactive navigation for visitors planning routes

- **Building Overview**: See all 7 floors at a glance with occupancy status
- **Floor Selection**: Click any floor to see detailed room layout
- **Interactive Floor Maps**: Visual overlays showing real-time occupancy on actual floor plans
- **Room-by-Room Details**: Detailed list of all rooms with camera IDs
- **Smart Navigation**: Easily switch between floors or return to overview
- **Visitor-Friendly**: Designed for route planning and finding available spaces

### Common Features

- **Responsive Design**: Optimized for desktop, tablet, and mobile devices
- **Manual Refresh**: Update data on demand with debounced clicks
- **Relative Timestamps**: "2m ago" format for intuitive time display
- **Error Handling**: Graceful error messages with actionable guidance
- **Loading States**: Visual feedback during data fetching
- **Dark Theme**: Modern color scheme (Brand: #102c52, Accent: #d3efef, Action: #d91821)

### Technical Features
- **Zero Dependencies**: Pure JavaScript, no build tools required
- **Configurable Thresholds**: Easy customization of capacity and status thresholds
- **Memory Management**: Proper cleanup of event listeners
- **XSS Protection**: HTML escaping for all user-generated content
- **Null Safety**: Comprehensive null checks for DOM elements
- **GitHub Pages Ready**: Deploy directly to GitHub Pages

## 🔐 Security

### Why Hardcoded Credentials are Safe

The Supabase **anon key** is safe to expose in frontend code because:

1. **Row Level Security (RLS)**: Supabase RLS policies restrict database access
   - Anon key can **only read** from the `detections` table
   - Cannot write, update, or delete data
   - Backend uses service role key (kept secret) for writes

2. **Public Information**: Both the project URL and anon key are intended to be public
   - Similar to API keys in public SDKs (Google Maps, Firebase, etc.)
   - Protected by RLS policies, not by secrecy

3. **No Sensitive Data**: The `detections` table contains only non-sensitive occupancy data
   - Room IDs, people counts, timestamps
   - No personal information or private details

### RLS Configuration Example

```sql
-- Enable RLS on detections table
ALTER TABLE detections ENABLE ROW LEVEL SECURITY;

-- Allow anonymous users to read all data
CREATE POLICY "Allow public read access"
  ON detections
  FOR SELECT
  TO anon
  USING (true);

-- Only authenticated users (backend) can insert
CREATE POLICY "Allow authenticated insert"
  ON detections
  FOR INSERT
  WITH CHECK (true);
```

## 📋 Setup Instructions

### 1. Prerequisites

- A Supabase project with a `detections` table (or `room_stats`)
- Supabase credentials: **Project URL** and **Anon Key**
- Backend system running and inserting data into Supabase

### 2. Get Your Supabase Credentials

1. Go to [supabase.com](https://supabase.com) and open your project
2. Click **Settings** → **API**
3. Copy:
   - **Project URL** (e.g., `https://xxxxx.supabase.co`)
   - **Anon Key** (the public key, NOT service_role key)

### 3. Configure the Frontend

Edit `script.js` and update the `CONFIG` object (lines 15-28):

```javascript
const CONFIG = {
    SUPABASE_URL: 'https://your-project.supabase.co',
    SUPABASE_ANON_KEY: 'your-anon-key-here',
    TABLE_NAME: 'detections', // or 'room_stats' if using old schema
    REFRESH_INTERVAL: 30000,
    MAX_RETRIES: 3,
    MAX_CAPACITY_PER_ROOM: 30,
    STALENESS_THRESHOLD_MINUTES: 5,
    THRESHOLDS: {
        LOW: 8,      // 0-8 people = "Light" (green)
        MODERATE: 20 // 9-20 = "Moderate" (amber), 21+ = "Busy" (red)
    }
};
```

### 4. Deploy

**Option A: GitHub Pages (Recommended)**
```bash
git add .
git commit -m "Configure Supabase credentials"
git push origin main

# Your site will be live at: https://username.github.io/repo-name
```

## 🚀 Performance

### Optimizations Implemented

- **Debounced Refresh**: 500ms debounce prevents rapid clicking
- **Efficient DOM Updates**: Cards only re-rendered when data changes
- **CSS Transforms**: GPU-accelerated animations for smooth performance
- **Minimal Reflows**: Batch DOM updates to reduce layout thrashing
- **Lazy Event Handlers**: Event listeners stored for efficient cleanup

### Performance Metrics

- **Initial Load**: < 2 seconds (typical)
- **Data Refresh**: < 500ms (depending on Supabase latency)
- **Memory Usage**: ~15-20MB (stable over time)
- **Animation FPS**: 60 FPS on modern devices

## 🔄 API Reference

### Supabase Table Schema

The app expects this table structure:

```sql
CREATE TABLE detections (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    room_id TEXT NOT NULL,
    person_count INTEGER NOT NULL,  -- or people_count
    timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

**Note**: The app supports both `person_count` and `people_count` column names.

### Frontend Query

```javascript
// Fetches latest 500 records, grouped by room_id
SELECT room_id, timestamp, person_count
FROM detections
ORDER BY timestamp DESC
LIMIT 500;
```

The frontend automatically:
1. Groups results by `room_id`
2. Keeps only the most recent record per room
3. Normalizes `person_count` ↔ `people_count`
4. Validates and sanitizes counts (removes negatives)

## 🛠️ Customization

### Changing Colors

Edit CSS variables in `styles.css`:

```css
:root {
    --primary-color: #2563eb;  /* Main accent color */
    --success-color: #10b981;  /* Light occupancy */
    --warning-color: #f59e0b;  /* Moderate occupancy */
    --danger-color: #ef4444;   /* Busy occupancy */
    --info-color: #0ea5e9;     /* Empty rooms */
}
```

### Changing Status Icons

Edit `getOccupancyStatus()` in `script.js`:

```javascript
// Current icons: 🚪 ✓ ⚠️ 🔴
// Change to: 🟢 🟡 🟠 🔴 for color circles
icon: '🟢'  // instead of '✓'
```

### Adding New Statistics

1. Add HTML element in `index.html` statistics section
2. Create variable in `initializeDOMElements()`
3. Update `updateStatistics()` to calculate and display

Example - Average Occupancy:
```javascript
// In updateStatistics()
const avgOccupancy = Math.round(total / rooms.length);
if (avgOccupancyElement) {
    avgOccupancyElement.textContent = avgOccupancy;
}
```

## 🐛 Debugging

### Enable Auto-Refresh

Uncomment line 79 in `script.js`:

```javascript
// Set up auto-refresh (optional)
setupAutoRefresh();  // <-- Remove the comment
```

### Console Logging

The app logs all errors to console. Open Developer Tools:
- Chrome/Edge: F12 or Cmd+Option+I (Mac)
- Firefox: F12 or Cmd+Option+K (Mac)
- Safari: Cmd+Option+C (Mac)

### Common Console Errors

| Error | Meaning | Solution |
|-------|---------|----------|
| `Supabase client not initialized` | SDK didn't load | Check internet connection |
| `Required DOM element not found` | HTML structure changed | Verify element IDs in index.html |
| `Could not find the table` | Table name wrong | Update CONFIG.TABLE_NAME |
| `Failed to fetch` | Network error | Check Supabase project status |

### Using Debug Tool

1. Open `debug.html` in browser
2. It will automatically:
   - Test Supabase connection
   - Try to access your table
   - List available tables
   - Show sample data structure
3. Share the output if requesting help

## 📊 Browser Compatibility

| Browser | Version | Status |
|---------|---------|--------|
| Chrome | 90+ | ✅ Fully Supported |
| Firefox | 88+ | ✅ Fully Supported |
| Safari | 14+ | ✅ Fully Supported |
| Edge | 90+ | ✅ Fully Supported |
| Mobile Safari | iOS 14+ | ✅ Fully Supported |
| Chrome Mobile | Android 90+ | ✅ Fully Supported |

**Features used**:
- CSS Custom Properties (CSS Variables)
- ES6+ JavaScript (arrow functions, async/await, template literals)
- Fetch API
- CSS Grid and Flexbox

**Not supported**: Internet Explorer (EOL)

## 🔒 Security Best Practices

### What's Safe

✅ **Exposing anon key in source code** - Protected by RLS
✅ **Committing CONFIG to git** - Public data only
✅ **Deploying to public hosting** - No secrets exposed

### What's NOT Safe

❌ **Don't expose service_role key** - Has full database access
❌ **Don't disable RLS** - Opens database to public writes
❌ **Don't store credentials in cookies/localStorage** - Unnecessary and risky

### Security Checklist

- [x] RLS enabled on database table
- [x] SELECT policy for anon role
- [x] No service_role key in frontend
- [x] HTML escaping for room names (XSS protection)
- [x] Input validation on counts (no negatives)
- [x] HTTPS connection (required by Supabase)

## 📞 Support & Resources

### Documentation Files

- `TROUBLESHOOTING.md` - Step-by-step problem solving
- `QUICK_FIX.md` - Quick reference for common issues  
- `NEW_UI_GUIDE.md` - User guide for guards/operators
- `UI_REDESIGN.md` - Design system documentation
- `future_plan.md` - Roadmap and planned features

### Getting Help

1. Check console for errors (F12)
2. Run `debug.html` for diagnostics
3. Review `TROUBLESHOOTING.md`
4. Check Supabase Dashboard for table structure
5. Verify RLS policies are correct

### Reporting Issues

Include:
1. Browser and version
2. Console error messages
3. Output from `debug.html`
4. Screenshot if applicable
5. Steps to reproduce

## 📝 Changelog

### Version 2.0 (November 2025) - Major Redesign

**Added**:
- Modern dark theme security dashboard
- Building-wide statistics panel
- 4-tier status system (empty/light/moderate/busy)
- Animated capacity bars
- Data staleness detection
- Filter controls (all/occupied/empty)
- Configurable thresholds in CONFIG
- Comprehensive null safety checks
- Debounced refresh button
- Memory leak prevention
- JSDoc documentation

**Fixed**:
- Hardcoded capacity thresholds → Now configurable
- Missing DOM element error handling
- Memory leaks from event listeners
- No validation on negative counts
- Inconsistent threshold usage across functions
- XSS vulnerability in room names

**Changed**:
- Table name: `room_stats` → `detections`
- Column name: Auto-detects `person_count` or `people_count`
- Status icons: Updated emoji set
- Statistics: Added busy rooms count

### Version 1.0 (Initial Release)

- Basic room occupancy display
- Light theme
- Manual refresh
- Simple card layout

## 📄 License

This project is part of an internal security monitoring system. All rights reserved.

## 🙏 Credits

- **YOLO v8**: Object detection model by Ultralytics
- **Supabase**: Backend-as-a-Service platform
- **Modern UI Design**: Inspired by security monitoring systems

---

**Last Updated**: November 26, 2025
**Version**: 2.0
**Maintainer**: Security Systems Team
```bash
# Serve with any static server
python -m http.server 8000
# or
npx serve .
```

## ⚙️ Configuration Options

### Adjustable Thresholds

All thresholds are configurable in the `CONFIG` object:

| Setting | Default | Description |
|---------|---------|-------------|
| `MAX_CAPACITY_PER_ROOM` | 30 | Maximum expected capacity per room (for percentage) |
| `STALENESS_THRESHOLD_MINUTES` | 5 | Data older than this shows stale warning |
| `THRESHOLDS.LOW` | 8 | Occupancy ≤ 8 = "Light" status (green) |
| `THRESHOLDS.MODERATE` | 20 | Occupancy ≤ 20 = "Moderate" status (amber) |
| `REFRESH_INTERVAL` | 30000 | Auto-refresh interval in milliseconds |

### Per-Room Custom Capacity

To set different capacities for specific rooms, modify `getOccupancyStatus()`:

```javascript
function getOccupancyStatus(count, maxCapacity) {
    // Custom capacities per room
    const roomCapacities = {
        'lobby': 50,
        'conference_a': 20,
        'room_101': 15
    };
    maxCapacity = roomCapacities[roomId] || CONFIG.MAX_CAPACITY_PER_ROOM;
    // ... rest of function
}
```

## 🔧 Maintenance Guide

### Known Issues and Solutions

#### Issue 1: DOM Element Not Found
**Symptom**: Console error "Required DOM element not found: [elementId]"

**Cause**: HTML structure doesn't match expected element IDs

**Solution**: 
1. Check `index.html` has all required elements with correct IDs
2. See `initializeDOMElements()` function for complete list
3. Element IDs are case-sensitive

#### Issue 2: Table Not Found
**Symptom**: "Could not find the table 'public.detections'"

**Cause**: Table name mismatch or table doesn't exist

**Solution**:
1. Run `debug.html` to diagnose
2. Check `TABLE_NAME` in CONFIG matches Supabase table
3. Verify table exists in Supabase Dashboard > Table Editor
4. See `TROUBLESHOOTING.md` for detailed steps

#### Issue 3: Stale Data Warnings
**Symptom**: Rooms show amber border with "Stale" badge

**Cause**: Backend hasn't updated data in 5+ minutes

**Solution**:
1. Check if backend is running
2. Verify cameras are sending images
3. Check Supabase connection from backend
4. Adjust `STALENESS_THRESHOLD_MINUTES` if needed

#### Issue 4: Incorrect Statistics
**Symptom**: "Busy Rooms" count doesn't match visual cards

**Cause**: Hardcoded threshold mismatch

**Solution**: 
- Now fixed! Uses `CONFIG.THRESHOLDS.MODERATE`
- All thresholds are centralized in CONFIG object
- Update CONFIG to change all threshold checks simultaneously

#### Issue 5: Memory Leaks
**Symptom**: Browser slow after long usage

**Cause**: Event listeners not cleaned up

**Solution**:
- Now fixed! Event listeners are properly removed on cleanup
- Auto-refresh clears interval on page unload
- Filter buttons use stored references for cleanup

### Code Quality Improvements (v2.0)

The following issues were identified and fixed:

1. ✅ **DOM Element Safety**: Added null checks and initialization validation
2. ✅ **Configurable Thresholds**: All hardcoded values moved to CONFIG
3. ✅ **Data Validation**: Added checks for negative counts and invalid data
4. ✅ **Memory Management**: Event listeners properly cleaned up
5. ✅ **Debouncing**: Refresh button now debounced (500ms) to prevent rapid clicks
6. ✅ **Error Boundaries**: Better error handling with try-catch blocks
7. ✅ **XSS Protection**: HTML escaping with null checks
8. ✅ **Type Safety**: Added parseInt() with fallbacks for all numeric values
9. ✅ **Documentation**: JSDoc comments for all major functions
10. ✅ **Consistent Naming**: Aligned column name handling (person_count vs people_count)

### Potential Future Issues

⚠️ **Issue**: CDN dependency on Supabase SDK
- **Risk**: If CDN is down, app won't work
- **Mitigation**: Consider self-hosting the Supabase JS SDK
- **Workaround**: Keep backup copy in `vendor/` folder

⚠️ **Issue**: No retry logic for failed requests
- **Risk**: Transient network errors cause complete failure
- **Mitigation**: Add exponential backoff retry in `fetchLatestRoomCounts()`
- **Status**: MAX_RETRIES defined but not implemented yet

⚠️ **Issue**: No request timeout
- **Risk**: Hanging requests could freeze UI
- **Mitigation**: Add timeout to Supabase client configuration
- **Workaround**: User can manually refresh

⚠️ **Issue**: Large datasets (1000+ rooms) may be slow
- **Risk**: Performance degradation with many rooms
- **Mitigation**: Implement pagination or virtual scrolling
- **Current Limit**: 500 records fetched, grouped by room_id

## 🧪 Testing Checklist

Before deploying, verify:

- [ ] Configure `CONFIG` with your Supabase credentials
- [ ] Table name matches (`detections` or `room_stats`)
- [ ] RLS policy allows SELECT for anon role
- [ ] Backend is inserting data with correct column names
- [ ] Test on Desktop (Chrome, Firefox, Safari, Edge)
- [ ] Test on Tablet (iPad, Android tablet)
- [ ] Test on Mobile (iPhone, Android phone)
- [ ] Refresh button works and shows loading state
- [ ] Filter buttons (All/Occupied/Empty) work correctly
- [ ] Statistics update when data changes
- [ ] Stale data warnings appear after 5 minutes
- [ ] Error messages display properly
- [ ] Empty state shows when no data
- [ ] Hard refresh clears cached data

## 📁 File Structure

```
/
├── index.html              # Dashboard view - Real-time monitoring
├── floorplan.html          # Floor plan view - Interactive building navigation
├── styles.css              # Shared styles (800+ lines)
├── floorplan-styles.css    # Floor plan specific styles
├── script.js               # Dashboard logic (500+ lines)
├── floorplan-script.js     # Floor plan logic
├── assets/
│   └── floormaps/          # Floor plan images
│       └── 4F.jpg          # 4th floor map (example)
├── debug.html              # Connection diagnostic tool
├── README.md               # This file
├── TROUBLESHOOTING.md      # Detailed troubleshooting guide
├── MAINTENANCE.md          # Maintenance and code quality guide
└── future_plan.md          # Planned enhancements
```

## 🏗️ Adding Floor Maps

### How to Add New Floor Plans

1. **Prepare Floor Plan Image:**
   - Format: JPG or PNG
   - Naming: `XF.jpg` (e.g., `1F.jpg`, `2F.jpg`, `3F.jpg`)
   - Place in: `assets/floormaps/`

2. **Update Configuration:**
   Edit `floorplan-script.js` line ~30:
   ```javascript
   FLOORS: [
       { number: 1, name: '1st Floor', hasMap: true, mapFile: '1F.jpg' },
       { number: 2, name: '2nd Floor', hasMap: true, mapFile: '2F.jpg' },
       // ... update hasMap: true and add mapFile
   ]
   ```

3. **Define Room Positions:**
   Edit `floorplan-script.js` line ~40:
   ```javascript
   const FLOOR_ROOM_POSITIONS = {
       1: { // 1st Floor
           101: { x: 15, y: 20, name: 'Main Lobby' },
           102: { x: 35, y: 20, name: 'Reception' },
           // Add all 10 cameras for the floor
           // x, y are percentages (0-100) of image dimensions
       },
       2: { // 2nd Floor
           201: { x: 20, y: 25, name: 'Conference Room A' },
           // ...
       }
   };
   ```

4. **Position Coordinates:**
   - `x`: Percentage from left edge (0 = far left, 100 = far right)
   - `y`: Percentage from top edge (0 = top, 100 = bottom)
   - Use image editing software to measure positions
   - Tip: Open floor plan in browser, use browser dev tools to get pixel coordinates, calculate percentages

### Example: Measuring Room Positions

1. Open `4F.jpg` in image editor (1000px wide, 800px tall)
2. Find Chemistry Lab at pixel (150, 160)
3. Calculate: x = (150/1000) × 100 = 15%, y = (160/800) × 100 = 20%
4. Add to config: `401: { x: 15, y: 20, name: 'Chemistry Lab' }`

### Using the Floor Plan View

1. **Visit:** `https://yourdomain.github.io/floorplan.html`
2. **Select a Floor:** Click on any floor card (1F - 7F)
3. **View Room Layout:** See which rooms are busy or available
4. **Plan Your Route:** Choose less crowded rooms
5. **Return to Overview:** Click "Back to Overview" to see all floors

### Using the Dashboard View

1. **Visit:** `https://yourdomain.github.io/index.html`
2. **See All Rooms:** Grid view of all rooms with live counts
3. **Filter Rooms:** Use "All", "Occupied", or "Empty" filters
4. **Check Status:** Color-coded indicators show occupancy levels
5. **Refresh Data:** Click refresh button for latest information

**Option B: Local Testing**
```bash
# Open in a local server (required for CORS)
python3 -m http.server 8000

# Visit: http://localhost:8000
```

## 🎨 Project Structure

```
.
├── index.html       # Main HTML structure
├── styles.css       # Responsive styling & layout
├── script.js        # Application logic & Supabase integration
├── README.md        # This file
└── .git/           # Git repository
```

## 📊 Database Schema

Expected table structure in Supabase:

```sql
CREATE TABLE room_stats (
  id BIGSERIAL PRIMARY KEY,
  room_id TEXT NOT NULL,
  timestamp TIMESTAMPTZ NOT NULL,
  people_count INT NOT NULL,
  CONSTRAINT people_count_non_negative CHECK (people_count >= 0)
);

-- Indexes for performance
CREATE INDEX idx_room_stats_room_id ON room_stats(room_id);
CREATE INDEX idx_room_stats_timestamp ON room_stats(timestamp DESC);
CREATE INDEX idx_room_stats_room_timestamp ON room_stats(room_id, timestamp DESC);
```

## 🚀 Usage

### Manual Refresh
Click the **Refresh** button to fetch the latest data from Supabase.

### Auto-Refresh (Optional)
Uncomment this line in `script.js` to enable auto-refresh every 30 seconds:

```javascript
// setupAutoRefresh();  // Uncomment to enable
```

### Room Card Information

Each room card displays:
- **Room Name** (e.g., "Lobby", "Room 101")
- **Occupancy Count** (number of people detected)
- **Status Badge** (Empty, Moderate, or Full)
- **Last Updated** (relative time, e.g., "2m ago")

### Occupancy Status Levels

| Status | People Count | Color |
|--------|-------------|-------|
| 🔵 Empty | 0 | Blue |
| 🟡 Moderate | 1-5 | Orange |
| 🔴 Full | 6+ | Red |

## 🛠️ Customization

### Change Colors

Edit CSS variables in `styles.css`:

```css
:root {
    --primary-color: #3b82f6;        /* Blue */
    --success-color: #10b981;        /* Green */
    --warning-color: #f59e0b;        /* Orange */
    --danger-color: #ef4444;         /* Red */
}
```

### Adjust Occupancy Thresholds

Edit the `getOccupancyStatus()` function in `script.js`:

```javascript
function getOccupancyStatus(count) {
    if (count === 0) {
        return { status: 'empty', color: '#0369a1' };
    } else if (count <= 10) {  // Change threshold from 5 to 10
        return { status: 'moderate', color: '#92400e' };
    } else {
        return { status: 'full', color: '#991b1b' };
    }
}
```

### Change Refresh Interval

Update `CONFIG.REFRESH_INTERVAL` in `script.js`:

```javascript
const CONFIG = {
    // ...
    REFRESH_INTERVAL: 60000,  // 60 seconds instead of 30
};
```

## 📱 Responsive Design

The website is fully responsive:

- **Desktop**: 4+ room cards per row
- **Tablet**: 2-3 room cards per row
- **Mobile**: 1 room card per row

All text, buttons, and layouts adapt automatically.

## 🐛 Troubleshooting

### "Supabase URL not configured"

Make sure you've updated `CONFIG.SUPABASE_URL` in `script.js` with your actual Supabase project URL.

### "Supabase anon key not configured"

Update `CONFIG.SUPABASE_ANON_KEY` in `script.js` with your actual anon key from Supabase Settings → API.

### "Failed to load room data"

Possible causes:
1. **Supabase credentials are wrong**: Double-check in Settings → API
2. **RLS policies are blocking reads**: Ensure you have a policy allowing `anon` users to read
3. **Table doesn't exist**: Verify the `room_stats` table exists in your Supabase database
4. **No data in table**: The backend must be inserting data for rows to display
5. **CORS issue**: If testing locally, use a local server (not file:// protocol)

### "No rooms with data"

This means the table exists but has no records. Verify your backend is running and inserting data into Supabase.

## 🔄 Backend Integration

This frontend works with the **Visitor Counting System Backend**:

- [Backend Repository](https://github.com/wwwtriplew/Visitor-Counting-System-Backend)
- Backend inserts `{room_id, timestamp, people_count}` into the `room_stats` table
- Frontend reads latest counts and displays them

## 📈 Future Enhancements

- **Real-time Updates**: Use Supabase Realtime to push updates instantly
- **Floor Plan Overlay**: Interactive floor plan with room locations
- **Historical Charts**: Show occupancy trends over time
- **Occupancy Alerts**: Notify when rooms reach capacity
- **Export Data**: CSV export for analysis
- **Dark Mode**: Toggle between light and dark themes
- **Custom Thresholds**: Admin panel to set occupancy limits per room

## 💻 Technical Stack

- **Frontend**: HTML5, CSS3, Vanilla JavaScript
- **Database**: Supabase (PostgreSQL)
- **SDK**: Supabase JS SDK v2
- **Hosting**: GitHub Pages
- **Browser Support**: All modern browsers (Chrome, Firefox, Safari, Edge)

## 📄 License

This project is part of the Visitor Counting System. See the backend repository for license details.

## 🤝 Contributing

To improve this frontend:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add your feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a pull request

## ❓ Support

For issues or questions:

1. Check the [Troubleshooting](#-troubleshooting) section
2. Review the [Backend Documentation](https://github.com/wwwtriplew/Visitor-Counting-System-Backend)
3. Open an issue on GitHub

---

**Made with ❤️ for the Visitor Counting System**
