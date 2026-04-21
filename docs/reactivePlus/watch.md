[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# `watch()` - Observe State Changes

## Quick Start (30 seconds)

```javascript
const user = state({
  name: 'Alice',
  status: 'online'
});

// Watch a specific property
watch(user, {
  status: (newValue, oldValue) => {
    console.log(`Status changed from ${oldValue} to ${newValue}`);
    showNotification(`User is now ${newValue}`);
  }
});

user.status = 'away';
// Console: "Status changed from online to away"
// Notification: "User is now away" ✨
```

**That's it.** The callback runs every time the watched property changes, giving you both the new and old values.

 

## What is `watch()`?

`watch()` lets you **react to specific property changes** with full access to both the previous and new values. Unlike `effect()` which runs whenever any accessed property changes, `watch()` explicitly targets specific properties.

Think of it as **setting up a motion sensor for specific doors**. Instead of monitoring everything, you watch exactly what you care about and get detailed information when changes occur.

**In practical terms:** Use watch when you need to know what changed, what the old value was, and perform specific actions in response.

 

## Syntax

```javascript
// Watch specific properties on a state object
const stopWatching = watch(state, {
  propertyName: (newValue, oldValue) => {
    // React to the change
  }
});

// Watch multiple properties
watch(state, {
  name: (newVal, oldVal) => { /* handle name change */ },
  email: (newVal, oldVal) => { /* handle email change */ },
  status: (newVal, oldVal) => { /* handle status change */ }
});

// Stop watching
stopWatching();
```

**Parameters:**
- `state` - The reactive state object to watch
- `definitions` - Object where keys are property names and values are callback functions

**Callback Parameters:**
- `newValue` - The new value after the change
- `oldValue` - The previous value before the change

**Returns:**
- A cleanup function that stops all watchers when called

 

## Why Does This Exist?

### Approach 1: effect() - General Reactivity

`effect()` is excellent for general reactive updates:

```javascript
const user = state({ name: 'Alice', age: 25 });

effect(() => {
  // Runs when ANY accessed property changes
  Elements.display.update({ textContent: `${user.name}, ${user.age}` });
});

user.name = 'Bob';  // Effect runs
user.age = 26;      // Effect runs
```

This approach is **great when you need**:
✅ Automatic dependency tracking
✅ Update UI when any related data changes
✅ Simple "just keep this in sync" behavior

### When Targeted Observation Is Your Goal

In scenarios where you need to react to **specific changes** with access to **before and after values**, `watch()` provides a more direct approach:

```javascript
const user = state({ name: 'Alice', age: 25 });

watch(user, {
  name: (newName, oldName) => {
    // Only runs when name changes
    // Has access to both values
    logAudit(`Name changed: ${oldName} → ${newName}`);
    sendNotification(`Welcome, ${newName}!`);
  },
  age: (newAge, oldAge) => {
    // Only runs when age changes
    if (newAge >= 18 && oldAge < 18) {
      unlockAdultFeatures();
    }
  }
});
```

**This approach is especially useful when:**
✅ You need the previous value for comparison
✅ You want to watch specific properties only
✅ You're implementing undo/redo functionality
✅ You need to log or audit changes
✅ You're syncing with external systems

**The Choice is Yours:**
- Use `effect()` when you want automatic, dependency-based reactions
- Use `watch()` when you need targeted observation with old/new values
- Both approaches are valid and can be combined

**Benefits of watch():**
✅ Explicit property targeting
✅ Access to previous values
✅ Clear intent in code
✅ Perfect for auditing and logging
✅ Ideal for conditional logic based on change

 

## Mental Model: The Security Camera

Think of `watch()` like **security cameras with motion detection**:

**effect() (General Sensor):**
```
┌─────────────────────────────────┐
│  Motion Sensor                  │
│                                 │
│  Detects ANY movement           │
│  in the whole room              │
│                                 │
│  Tells you: "Something moved!"  │
│                                 │
│  Simple, automatic, broad ✅    │
└─────────────────────────────────┘
```

**watch() (Targeted Camera):**
```
┌─────────────────────────────────┐
│  Security Camera                │
│                                 │
│  Watches SPECIFIC door          │
│  Records before AND after       │
│                                 │
│  Tells you:                     │
│  "Door was CLOSED,              │
│   now it's OPEN"                │
│                                 │
│  Detailed, targeted, precise ✅ │
└─────────────────────────────────┘
```

**Key Insight:** Watch gives you specific, detailed information about exactly what changed.

 

## How Does It Work?

Under the hood, watch uses effects with value comparison:

```
watch(state, { name: callback })
              ↓
┌─────────────────────────────────────────────────────┐
│ 1. Store initial value: oldValue = state.name       │
└─────────────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────────┐
│ 2. Create an effect that:                           │
│    - Reads state.name (tracks dependency)           │
│    - Compares with oldValue                         │
│    - If different: call callback(new, old)          │
│    - Updates oldValue for next comparison           │
└─────────────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────────┐
│ 3. When state.name changes:                         │
│    - Effect re-runs                                 │
│    - Detects newValue !== oldValue                  │
│    - Calls your callback with both values           │
└─────────────────────────────────────────────────────┘
```

**Step-by-step:**

1️⃣ **You set up a watch:**
```javascript
watch(user, {
  status: (newVal, oldVal) => {
    console.log(`${oldVal} → ${newVal}`);
  }
});
```

2️⃣ **System stores initial value:**
```javascript
// Internally: oldValue = 'online'
```

3️⃣ **When the property changes:**
```javascript
user.status = 'away';
// System compares: 'away' !== 'online'
// Calls callback('away', 'online')
```

4️⃣ **Old value updates for next change:**
```javascript
// Internally: oldValue = 'away'
user.status = 'offline';
// Calls callback('offline', 'away')
```

 

## Basic Usage

### Example 1: Simple Property Watch

```javascript
const counter = state({ count: 0 });

watch(counter, {
  count: (newCount, oldCount) => {
    console.log(`Count: ${oldCount} → ${newCount}`);
  }
});

counter.count = 5;  // "Count: 0 → 5"
counter.count = 10; // "Count: 5 → 10"
```

 

### Example 2: Multiple Property Watches

```javascript
const settings = state({
  theme: 'light',
  fontSize: 16,
  language: 'en'
});

watch(settings, {
  theme: (newTheme, oldTheme) => {
    console.log(`Theme changed to ${newTheme}`);
    applyTheme(newTheme);
  },
  fontSize: (newSize, oldSize) => {
    console.log(`Font size: ${oldSize}px → ${newSize}px`);
    document.body.style.fontSize = `${newSize}px`;
  },
  language: (newLang, oldLang) => {
    console.log(`Language changed to ${newLang}`);
    loadTranslations(newLang);
  }
});
```

 

### Example 3: Conditional Actions Based on Values

```javascript
const gameState = state({
  lives: 3,
  score: 0,
  level: 1
});

watch(gameState, {
  lives: (newLives, oldLives) => {
    if (newLives < oldLives) {
      // Lost a life
      playSound('damage.mp3');
      flashScreen('red');
    }
    if (newLives === 0) {
      // Game over
      showGameOverScreen();
    }
  },
  score: (newScore, oldScore) => {
    const gained = newScore - oldScore;
    if (gained > 0) {
      showPointsAnimation(`+${gained}`);
    }
    // Check for high score
    if (newScore > getHighScore()) {
      setHighScore(newScore);
      showCelebration();
    }
  },
  level: (newLevel, oldLevel) => {
    if (newLevel > oldLevel) {
      playSound('levelup.mp3');
      showLevelUpBanner(newLevel);
    }
  }
});
```

 

### Example 4: Stopping a Watch

```javascript
const data = state({ value: 0 });

const stopWatching = watch(data, {
  value: (newVal, oldVal) => {
    console.log(`Changed: ${oldVal} → ${newVal}`);
  }
});

data.value = 1; // "Changed: 0 → 1"
data.value = 2; // "Changed: 1 → 2"

// Stop watching
stopWatching();

data.value = 3; // Nothing logged - watch stopped
data.value = 4; // Nothing logged - watch stopped
```

 

### Example 5: Form Field Validation on Change

```javascript
const form = state({
  email: '',
  password: '',
  username: ''
});

watch(form, {
  email: (newEmail) => {
    const isValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(newEmail);
    Elements.emailError.update({
      hidden:      isValid || !newEmail,
      textContent: !isValid && newEmail ? 'Please enter a valid email' : ''
    });
  },
  password: (newPassword) => {
    const strength = calculatePasswordStrength(newPassword);
    Elements.passwordStrength.update({
      textContent: strength.label,
      className:   `strength-${strength.level}`
    });
  },
  username: async (newUsername) => {
    if (newUsername.length >= 3) {
      const available = await checkUsernameAvailability(newUsername);
      Elements.usernameStatus.update({
        textContent: available ? '✓ Available' : '✗ Taken',
        className:   available ? 'available' : 'taken'
      });
    }
  }
});
```

 

## Watch with DOM Helpers

### Example 1: Status Badge Updates

```javascript
const user = state({
  status: 'online',
  lastActive: Date.now()
});

watch(user, {
  status: (newStatus, oldStatus) => {
    // Update status badge
    Elements.update({
      statusBadge: {
        textContent: newStatus,
        className: `badge badge-${newStatus}`
      },
      statusIcon: {
        className: `icon icon-${newStatus}`
      }
    });

    // Update all status indicators in the page
    Collections.ClassName.userStatus.update({
      textContent: newStatus
    });

    // Log status change
    console.log(`User went from ${oldStatus} to ${newStatus}`);
  }
});
```

 

### Example 2: Cart Item Count Badge

```javascript
const cart = state({
  items: []
});

// Watch items array length by using a computed
computed(cart, {
  itemCount: function() {
    return this.items.length;
  }
});

watch(cart, {
  itemCount: (newCount, oldCount) => {
    Elements.cartBadge.update({ textContent: newCount, hidden: newCount === 0 });

    if (newCount > oldCount) {
      Elements.cartBadge.classList.add('bounce');
      setTimeout(() => Elements.cartBadge.classList.remove('bounce'), 300);
    }
  }
});
```

 

### Example 3: Theme Transition

```javascript
const app = state({
  theme: 'light'
});

watch(app, {
  theme: (newTheme, oldTheme) => {
    // Add transition class
    document.body.classList.add('theme-transitioning');

    // Update theme
    document.body.classList.remove(`theme-${oldTheme}`);
    document.body.classList.add(`theme-${newTheme}`);

    // Update all theme-aware elements
    Collections.ClassName.themeToggle.update({
      textContent: newTheme === 'dark' ? '☀️' : '🌙'
    });

    // Remove transition class after animation
    setTimeout(() => {
      document.body.classList.remove('theme-transitioning');
    }, 300);

    // Save preference
    localStorage.setItem('theme', newTheme);
  }
});
```

 

## Real-World Examples

### Example 1: Undo/Redo System

```javascript
const document = state({
  content: ''
});

const history = [];
let historyIndex = -1;

watch(document, {
  content: (newContent, oldContent) => {
    // Don't record if we're undoing/redoing
    if (isUndoRedo) return;

    // Remove any "future" history if we're adding after an undo
    history.splice(historyIndex + 1);

    // Add to history
    history.push({
      before: oldContent,
      after: newContent,
      timestamp: Date.now()
    });
    historyIndex = history.length - 1;

    // Update UI
    Elements.undoBtn.disabled = historyIndex < 0;
    Elements.redoBtn.disabled = true; // No future to redo
  }
});

let isUndoRedo = false;

function undo() {
  if (historyIndex < 0) return;

  isUndoRedo = true;
  document.content = history[historyIndex].before;
  historyIndex--;
  isUndoRedo = false;

  Elements.undoBtn.disabled = historyIndex < 0;
  Elements.redoBtn.disabled = false;
}

function redo() {
  if (historyIndex >= history.length - 1) return;

  isUndoRedo = true;
  historyIndex++;
  document.content = history[historyIndex].after;
  isUndoRedo = false;

  Elements.undoBtn.disabled = false;
  Elements.redoBtn.disabled = historyIndex >= history.length - 1;
}
```

 

### Example 2: Real-Time Sync Indicator

```javascript
const syncState = state({
  status: 'synced', // 'synced', 'syncing', 'error', 'offline'
  lastSync: null,
  pendingChanges: 0
});

watch(syncState, {
  status: (newStatus, oldStatus) => {
    // Update sync indicator
    const icons = {
      synced: '✓',
      syncing: '↻',
      error: '✗',
      offline: '○'
    };

    Elements.update({
      syncIcon: {
        textContent: icons[newStatus],
        className: `sync-icon sync-${newStatus}`
      },
      syncText: {
        textContent: newStatus.charAt(0).toUpperCase() + newStatus.slice(1)
      }
    });

    // Show notification on error
    if (newStatus === 'error' && oldStatus !== 'error') {
      showNotification('Sync failed. Will retry automatically.', 'error');
    }

    // Show success on recovery
    if (newStatus === 'synced' && oldStatus === 'error') {
      showNotification('Connection restored!', 'success');
    }
  },
  pendingChanges: (newCount) => {
    Elements.pendingBadge.update({ textContent: newCount, hidden: newCount === 0 });
  }
});
```

 

### Example 3: Authentication State Handler

```javascript
const auth = state({
  isLoggedIn: false,
  user: null,
  token: null
});

watch(auth, {
  isLoggedIn: (loggedIn, wasLoggedIn) => {
    if (loggedIn && !wasLoggedIn) {
      // Just logged in
      showWelcomeMessage(auth.user.name);
      loadUserPreferences();
      initializeWebSocket();

      Elements.update({ loginBtn: { hidden: true }, userMenu: { hidden: false } });
    } else if (!loggedIn && wasLoggedIn) {
      // Just logged out
      showLogoutMessage();
      clearUserData();
      disconnectWebSocket();

      Elements.update({ loginBtn: { hidden: false }, userMenu: { hidden: true } });

      // Redirect to login page
      window.location.href = '/login';
    }
  },
  user: (newUser, oldUser) => {
    if (newUser) {
      Elements.update({
        userName: { textContent: newUser.name },
        userAvatar: { src: newUser.avatar },
        userEmail: { textContent: newUser.email }
      });
    }
  },
  token: (newToken, oldToken) => {
    if (newToken) {
      // Set up API headers
      setAuthHeader(newToken);
      // Refresh token before expiry
      scheduleTokenRefresh(newToken);
    } else {
      // Clear API headers
      clearAuthHeader();
    }
  }
});
```

 

### Example 4: Data Change Auditing

```javascript
const sensitiveData = state({
  creditLimit: 5000,
  accessLevel: 'standard',
  permissions: []
});

watch(sensitiveData, {
  creditLimit: (newLimit, oldLimit) => {
    // Log to audit trail
    logAudit({
      action: 'CREDIT_LIMIT_CHANGE',
      oldValue: oldLimit,
      newValue: newLimit,
      user: getCurrentUser(),
      timestamp: new Date().toISOString()
    });

    // Alert for significant changes
    if (Math.abs(newLimit - oldLimit) > 1000) {
      sendAlertEmail('Large credit limit change detected');
    }
  },
  accessLevel: (newLevel, oldLevel) => {
    logAudit({
      action: 'ACCESS_LEVEL_CHANGE',
      oldValue: oldLevel,
      newValue: newLevel,
      user: getCurrentUser(),
      timestamp: new Date().toISOString()
    });

    // Require re-authentication for elevated access
    if (getAccessRank(newLevel) > getAccessRank(oldLevel)) {
      requireReAuthentication();
    }
  }
});
```

 

### Example 5: Progress Tracker with Milestones

```javascript
const progress = state({
  percent: 0,
  milestonesReached: []
});

const MILESTONES = [25, 50, 75, 100];

watch(progress, {
  percent: (newPercent, oldPercent) => {
    Elements.update({
      progressBar:  { style: { width: `${newPercent}%` } },
      progressText: { textContent: `${Math.round(newPercent)}%` }
    });

    // Check for milestone crossing
    MILESTONES.forEach(milestone => {
      if (newPercent >= milestone && oldPercent < milestone) {
        // Just crossed a milestone
        showMilestoneAchieved(milestone);
        progress.milestonesReached.push(milestone);

        if (milestone === 100) {
          showCompletionCelebration();
        }
      }
    });

    // Update progress bar color based on percentage
    let color = '#3498db'; // blue
    if (newPercent >= 75) color = '#27ae60'; // green
    else if (newPercent >= 50) color = '#f39c12'; // orange
    else if (newPercent >= 25) color = '#e74c3c'; // red

    Elements.progressBar.style.backgroundColor = color;
  }
});
```

 

## Common Patterns

### Pattern 1: Debounced Watch Actions

```javascript
let searchTimeout;

watch(searchState, {
  query: (newQuery, oldQuery) => {
    // Clear previous timeout
    clearTimeout(searchTimeout);

    // Debounce search
    searchTimeout = setTimeout(() => {
      performSearch(newQuery);
    }, 300);
  }
});
```

 

### Pattern 2: Comparison-Based Actions

```javascript
watch(metrics, {
  value: (newVal, oldVal) => {
    const change = newVal - oldVal;
    const percentChange = (change / oldVal) * 100;

    if (percentChange > 10) {
      showAlert('Significant increase detected!');
    } else if (percentChange < -10) {
      showAlert('Significant decrease detected!');
    }
  }
});
```

 

### Pattern 3: External System Sync

```javascript
watch(localState, {
  data: (newData, oldData) => {
    // Sync to server
    api.updateData(newData).catch(err => {
      // Rollback on error
      console.error('Sync failed, rolling back');
      localState.data = oldData;
    });
  }
});
```

 

### Pattern 4: Analytics Tracking

```javascript
watch(userActions, {
  currentPage: (newPage, oldPage) => {
    analytics.track('page_view', {
      page: newPage,
      previousPage: oldPage,
      duration: calculateTimeOnPage(oldPage)
    });
  }
});
```

 

### Pattern 5: Storage Persistence

```javascript
watch(preferences, {
  theme: (newTheme) => {
    localStorage.setItem('theme', newTheme);
  },
  language: (newLang) => {
    localStorage.setItem('language', newLang);
  },
  fontSize: (newSize) => {
    localStorage.setItem('fontSize', newSize);
  }
});
```

 

## Important Notes

### 1. Watch Doesn't Run Initially

```javascript
const data = state({ value: 'initial' });

watch(data, {
  value: (newVal, oldVal) => {
    console.log(`Changed: ${oldVal} → ${newVal}`);
  }
});

// Nothing logged yet - callback only runs on CHANGES

data.value = 'updated';
// Now logs: "Changed: initial → updated"
```

 

### 2. Same Value Doesn't Trigger Watch

```javascript
const data = state({ value: 5 });

watch(data, {
  value: (newVal, oldVal) => {
    console.log('Value changed!');
  }
});

data.value = 5; // Nothing logged - same value
data.value = 10; // "Value changed!"
data.value = 10; // Nothing logged - same value
```

 

### 3. Watch Returns Cleanup Function

```javascript
const data = state({ count: 0 });

const cleanup = watch(data, {
  count: (newVal) => console.log('Count:', newVal)
});

data.count = 1; // "Count: 1"

// Stop watching
cleanup();

data.count = 2; // Nothing logged
```

 

### 4. Watching Nested Properties

```javascript
// For nested properties, watch computed values
const user = state({
  profile: {
    name: 'Alice',
    email: 'alice@example.com'
  }
});

computed(user, {
  userName: function() { return this.profile.name; },
  userEmail: function() { return this.profile.email; }
});

watch(user, {
  userName: (newName, oldName) => {
    console.log(`Name: ${oldName} → ${newName}`);
  }
});

user.profile.name = 'Bob'; // "Name: Alice → Bob"
```

 

## Summary

**What is `watch()`?**
A way to observe specific property changes with access to both old and new values.

**Why use it?**
- ✅ Targeted observation of specific properties
- ✅ Access to previous values for comparison
- ✅ Perfect for auditing and logging
- ✅ Ideal for undo/redo functionality
- ✅ Great for syncing with external systems

**Key Takeaway:**

```
effect()                     watch()
    |                           |
Automatic tracking          Explicit targeting
    |                           |
Knows "it changed"          Knows "what → what"
    |                           |
General reactions           Specific responses
```

**One-Line Rule:** Use `watch()` when you need to know both what a value was and what it became.

**The Magic Formula:**
```
state({ value }) + watch({ value: callback })
───────────────────────────────────────────
value changes → callback(newValue, oldValue)
```

**Best Practices:**
- Use for logging and auditing state changes
- Implement undo/redo with previous values
- Sync with external systems on specific changes
- Combine with computed for watching derived values
- Always store cleanup function if you need to stop watching

**Remember:** Watch is your targeted observer — it tells you exactly what changed and what it changed from!
