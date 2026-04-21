[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# `options.storage` - Storage Type Configuration

## Quick Start (30 seconds)

```javascript
const userState = state({ name: 'Alice', email: 'alice@example.com' });

// Save to localStorage (persists forever)
autoSave(userState, 'userData', {
  storage: 'localStorage'
});

// Save to sessionStorage (cleared when tab closes)
const sessionState = state({ tempData: 'temporary' });
autoSave(sessionState, 'session', {
  storage: 'sessionStorage'
});

// Default is localStorage if not specified
const defaultState = state({ count: 0 });
autoSave(defaultState, 'counter'); // Uses localStorage ✨
```

**What just happened?** You chose WHERE to store your data - either localStorage (permanent) or sessionStorage (temporary)!

  

## What is `options.storage`?

`options.storage` is a configuration option that **specifies which browser storage mechanism to use for persisting your reactive state**.

Simply put: it's like choosing between a filing cabinet (localStorage) and a desk drawer (sessionStorage). One keeps things forever, the other only while you're working.

Think of it as **selecting your storage container** - permanent or temporary.

  

## Syntax

```javascript
autoSave(state, key, {
  storage: 'localStorage' | 'sessionStorage'
})
```

**Value:**
- `'localStorage'` (default) - Data persists forever until manually deleted
- `'sessionStorage'` - Data persists only for the current browser tab/window session

**Default:** `'localStorage'` if not specified

  

## Why Does This Exist?

### The Challenge: Different Data Needs Different Lifetimes

Not all data should live forever:

```javascript
// Without storage option - everything uses localStorage
autoSave(userPreferences, 'prefs');    // Should persist ✅
autoSave(authToken, 'token');          // Should persist ✅
autoSave(formDraft, 'draft');          // Should persist ✅
autoSave(searchQuery, 'search');       // Shouldn't persist! ❌
autoSave(filterState, 'filters');      // Shouldn't persist! ❌

// Problem: Search and filters stick around between sessions
// User closes tab and returns tomorrow
// Search box still shows yesterday's query ❌
// Filters still show yesterday's selection ❌
```

**What's the Real Issue?**

```
All data in localStorage
        |
        v
Some data should be temporary
        |
        v
But it persists forever
        |
        v
Stale data on next visit ❌
        |
        v
Confusing user experience
```

**Problems:**
❌ **Data pollution** - Temporary data clutters permanent storage  
❌ **Privacy concerns** - Sensitive data stays on disk  
❌ **Stale state** - UI shows outdated temporary data  
❌ **No automatic cleanup** - Must manually clear temporary data  

### The Solution with `options.storage`

```javascript
// User preferences - keep forever
const preferences = state({ theme: 'dark', lang: 'en' });
autoSave(preferences, 'prefs', {
  storage: 'localStorage'  // Persists between sessions ✅
});

// Search query - only for this session
const searchState = state({ query: '' });
autoSave(searchState, 'search', {
  storage: 'sessionStorage'  // Auto-clears when tab closes ✅
});

// User closes tab and returns tomorrow
// Preferences: Still there ✅
// Search query: Gone (fresh start) ✅
```

**What Just Happened?**

```
Choose storage type
        |
        v
localStorage → Permanent
sessionStorage → Temporary
        |
        v
Data stored appropriately
        |
        v
Automatic cleanup for temp data ✅
```

**Benefits:**
✅ **Right lifetime for each data type** - Permanent vs temporary  
✅ **Automatic cleanup** - sessionStorage clears itself  
✅ **Better privacy** - Sensitive data doesn't persist  
✅ **Cleaner experience** - No stale temporary data  

  

## Mental Model

Think of `localStorage` as a **filing cabinet**:

```
localStorage (Filing Cabinet)
┌─────────────────────┐
│  Permanent storage  │
│                     │
│  Files stay until   │
│  manually removed   │
│                     │
│  Survives:          │
│  - Tab close        │
│  - Browser restart  │
│  - Computer restart │
│                     │
│  Perfect for:       │
│  - User settings    │
│  - Saved data       │
│  - Preferences      │
└─────────────────────┘
```

Think of `sessionStorage` as a **desk notepad**:

```
sessionStorage (Desk Notepad)
┌─────────────────────┐
│  Temporary storage  │
│                     │
│  Notes cleared when │
│  you leave desk     │
│                     │
│  Cleared when:      │
│  - Tab closes       │
│  - Browser closes   │
│                     │
│  Survives:          │
│  - Page refresh     │
│  - Navigation       │
│                     │
│  Perfect for:       │
│  - Session data     │
│  - Temp filters     │
│  - Search queries   │
└─────────────────────┘
```

**Key Insight:** Choose localStorage for permanent data, sessionStorage for temporary data.

  

## How Does It Work?

When you specify `options.storage`, autoSave routes your data to the appropriate browser storage:

### Step-by-Step Process

**1️⃣ Configuration**
```javascript
autoSave(state, 'myData', {
  storage: 'sessionStorage'
});
```

**2️⃣ Storage Selection**
```
autoSave receives config
        |
        v
Check storage option
        |
    localStorage? ──> Use window.localStorage
    sessionStorage? ──> Use window.sessionStorage
        |
        v
Create storage wrapper
```

**3️⃣ Data Operations**
```
When state changes:
        |
        v
Serialize to JSON
        |
        v
Save to selected storage
        |
        v
sessionStorage.setItem('myData', json)
    or
localStorage.setItem('myData', json)
```

**4️⃣ Lifetime Management**
```
localStorage:
    Data persists forever ∞
        |
        v
    Manual deletion required

sessionStorage:
    Data cleared on tab close
        |
        v
    Automatic cleanup ✨
```

  

## Basic Usage

### Example 1: Default (localStorage)

```javascript
const userState = state({
  name: 'Alice',
  email: 'alice@example.com'
});

// Default storage is localStorage
autoSave(userState, 'user');

// Or explicitly specify
autoSave(userState, 'user', {
  storage: 'localStorage'
});

// Data persists forever
// Close tab, restart browser - data still there!
```

  

### Example 2: Temporary Data (sessionStorage)

```javascript
const searchState = state({
  query: '',
  filters: [],
  page: 1
});

// Use sessionStorage for temporary data
autoSave(searchState, 'search', {
  storage: 'sessionStorage'
});

// Data only lasts for this session
// Close tab - data disappears automatically ✨
```

  

### Example 3: Mixed Storage Strategy

```javascript
// Permanent user preferences
const preferences = state({
  theme: 'dark',
  language: 'en',
  fontSize: 16
});

autoSave(preferences, 'preferences', {
  storage: 'localStorage'  // Keep forever
});

// Temporary session data
const session = state({
  currentPage: 'home',
  scrollPosition: 0,
  expandedItems: []
});

autoSave(session, 'sessionData', {
  storage: 'sessionStorage'  // Clear on tab close
});
```

  

## localStorage vs sessionStorage

### Comparison Table

```
┌────────────────────┬──────────────────┬────────────────────┐
│ Feature            │ localStorage     │ sessionStorage     │
├────────────────────┼──────────────────┼────────────────────┤
│ Lifetime           │ Forever          │ Current session    │
│ Survives tab close │ Yes ✅           │ No ❌              │
│ Survives refresh   │ Yes ✅           │ Yes ✅             │
│ Shared across tabs │ Yes ✅           │ No ❌              │
│ Storage limit      │ ~5-10MB          │ ~5-10MB            │
│ Auto-cleanup       │ No ❌            │ Yes ✅             │
└────────────────────┴──────────────────┴────────────────────┘
```

### When to Use Each

**Use localStorage for:**
- User preferences and settings
- Authentication tokens (with expiration)
- Shopping cart contents
- User-created data
- App configuration
- Feature flags
- Favorites/bookmarks

**Use sessionStorage for:**
- Search queries and results
- Form data in progress
- Temporary filters
- Current tab state
- Wizard/multi-step form data
- Temporary authentication (for sensitive apps)
- Session-specific UI state

  

### Example: Side-by-Side Comparison

```javascript
// localStorage Example: User Preferences
const preferences = state({
  theme: 'dark',
  notifications: true
});

autoSave(preferences, 'prefs', {
  storage: 'localStorage'
});

// User sets theme to dark
preferences.theme = 'dark';

// Close browser, reopen tomorrow
// Theme is still 'dark' ✅

//   

// sessionStorage Example: Search State
const search = state({
  query: 'javascript',
  results: []
});

autoSave(search, 'search', {
  storage: 'sessionStorage'
});

// User searches for 'javascript'
search.query = 'javascript';

// Close tab, open new tab
// Search is empty (fresh start) ✅
```

  

## Real-World Examples

### Example 1: E-commerce Site

```javascript
// Permanent: Shopping cart (keep between sessions)
const cart = state({
  items: [],
  total: 0
});

autoSave(cart, 'shoppingCart', {
  storage: 'localStorage'
});

// Temporary: Current filters (only for this session)
const filters = state({
  category: '',
  priceRange: [0, 1000],
  sortBy: 'relevance'
});

autoSave(filters, 'productFilters', {
  storage: 'sessionStorage'
});

// User browses, adds items to cart, applies filters
// Closes tab and returns next day:
// - Cart items still there ✅
// - Filters reset to defaults ✅
```

  

### Example 2: Blog Editor

```javascript
// Permanent: User settings
const settings = state({
  autoSave: true,
  spellCheck: true,
  wordWrap: true
});

autoSave(settings, 'editorSettings', {
  storage: 'localStorage'
});

// Temporary: Current editing session
const editorSession = state({
  cursorPosition: 0,
  selection: null,
  undoStack: []
});

autoSave(editorSession, 'editSession', {
  storage: 'sessionStorage'
});

// Permanent: Draft content (important!)
const draft = state({
  title: '',
  content: '',
  lastSaved: null
});

autoSave(draft, 'articleDraft', {
  storage: 'localStorage'
});
```

  

### Example 3: Dashboard Application

```javascript
// Permanent: User dashboard layout
const layout = state({
  widgets: ['chart', 'table', 'summary'],
  positions: { chart: [0, 0], table: [1, 0] }
});

autoSave(layout, 'dashboardLayout', {
  storage: 'localStorage'
});

// Temporary: Current view state
const viewState = state({
  selectedWidget: 'chart',
  zoomLevel: 1,
  filters: []
});

autoSave(viewState, 'viewState', {
  storage: 'sessionStorage'
});

// User customizes dashboard, closes tab
// Returns later:
// - Layout preserved ✅
// - View state reset ✅
```

  

### Example 4: Multi-Step Form

```javascript
// Temporary: Form progress (only for this session)
const formState = state({
  currentStep: 1,
  completedSteps: [],
  formData: {
    step1: {},
    step2: {},
    step3: {}
  }
});

autoSave(formState, 'multiStepForm', {
  storage: 'sessionStorage'
});

// If user accidentally closes tab:
// - Can refresh and continue ✅
// - But closing tab clears it (privacy) ✅
// - Fresh start if they return later ✅
```

  

### Example 5: Authentication System

```javascript
// Permanent: Remember me (long-term token)
const persistentAuth = state({
  refreshToken: '',
  userId: '',
  rememberMe: false
});

autoSave(persistentAuth, 'auth', {
  storage: 'localStorage'  // Stays logged in
});

// Temporary: Current session (sensitive)
const sessionAuth = state({
  accessToken: '',
  sessionId: '',
  permissions: []
});

autoSave(sessionAuth, 'session', {
  storage: 'sessionStorage'  // Auto-logout on tab close
});

// Security benefit:
// - Long-term token in localStorage for convenience
// - Sensitive session token in sessionStorage
// - Session token cleared on tab close ✅
```

  

## Common Patterns

### Pattern 1: Check Storage Availability

```javascript
function createAutoSavedState(key, initialData, useSession = false) {
  const state = state(initialData);
  
  const storageType = useSession ? 'sessionStorage' : 'localStorage';
  
  // Check if storage is available
  try {
    window[storageType].setItem('test', 'test');
    window[storageType].removeItem('test');
    
    autoSave(state, key, { storage: storageType });
  } catch (e) {
    console.warn(`${storageType} not available, state won't persist`);
  }
  
  return state;
}
```

  

### Pattern 2: Dynamic Storage Selection

```javascript
function smartAutoSave(state, key, isPermanent) {
  const storage = isPermanent ? 'localStorage' : 'sessionStorage';
  
  return autoSave(state, key, {
    storage,
    debounce: 300
  });
}

// Usage
const userPrefs = state({ theme: 'dark' });
smartAutoSave(userPrefs, 'prefs', true);  // localStorage

const filters = state({ category: '' });
smartAutoSave(filters, 'filters', false); // sessionStorage
```

  

### Pattern 3: Migrate Between Storage Types

```javascript
function migrateToSessionStorage(key) {
  // Get from localStorage
  const data = localStorage.getItem(key);
  
  if (data) {
    // Move to sessionStorage
    sessionStorage.setItem(key, data);
    
    // Remove from localStorage
    localStorage.removeItem(key);
    
    console.log(`Migrated ${key} to sessionStorage`);
  }
}

migrateToSessionStorage('tempData');
```

  

### Pattern 4: Dual Storage Strategy

```javascript
// Save to both for redundancy
function dualSave(state, key) {
  autoSave(state, key, {
    storage: 'localStorage',
    onSave: (data) => {
      // Also save to sessionStorage
      sessionStorage.setItem(key, JSON.stringify(data));
      return data;
    }
  });
}
```

  

### Pattern 5: Privacy Mode Detection

```javascript
function autoSaveWithPrivacy(state, key, options = {}) {
  // Detect if private browsing
  const isPrivate = !window.indexedDB;
  
  // Use sessionStorage in private mode
  const storage = isPrivate ? 'sessionStorage' : 'localStorage';
  
  return autoSave(state, key, {
    ...options,
    storage
  });
}
```

  

### Pattern 6: Environment-Based Storage

```javascript
function autoSaveForEnv(state, key) {
  // Use sessionStorage in development
  const storage = process.env.NODE_ENV === 'development'
    ? 'sessionStorage'
    : 'localStorage';
  
  return autoSave(state, key, { storage });
}
```

  

## Summary

**What is `options.storage`?**  
A configuration option that specifies whether to use localStorage (permanent) or sessionStorage (temporary) for persisting reactive state.

**Why use it?**
- ✅ Choose data lifetime (permanent vs temporary)
- ✅ Automatic cleanup for temporary data
- ✅ Better privacy for sensitive data
- ✅ Cleaner user experience
- ✅ Appropriate storage for each use case

**Key Takeaway:**

```
Without storage option     With storage option
         |                         |
   All localStorage          Smart choices
         |                         |
   Everything persists      Permanent data → localStorage
         |                  Temporary data → sessionStorage
         |                         |
   Stale data ❌            Clean experience ✅
```

**One-Line Rule:** Use `localStorage` for data that should survive sessions, `sessionStorage` for data that should be temporary.

**Quick Decision Guide:**
- **Should data persist between visits?** → `localStorage`
- **Should data clear when tab closes?** → `sessionStorage`
- **Is data sensitive/temporary?** → `sessionStorage`
- **Is data user preferences/settings?** → `localStorage`

**Remember:** Choose the right container for the right data! 🎉