[![Sponsor](https://img.shields.io/badge/Sponsor-💖-pink)](https://github.com/sponsors/giovanni1707)

[![Sponsor](https://img.shields.io/badge/Sponsor-PayPal-blue?logo=paypal)](https://paypal.me/GiovanniSylvain)

# `refs()` - Create Multiple Refs at Once

## Quick Start (30 seconds)

```javascript
// Create multiple refs in one call
const { count, name, isActive } = refs({
  count: 0,
  name: 'Alice',
  isActive: true
});

// Each is a separate ref
effect(() => {
  Elements.update({
    counterDisplay: { textContent: count.value },
    nameDisplay: { textContent: name.value },
    statusDisplay: { textContent: isActive.value ? 'Active' : 'Inactive' }
  });
});

count.value++; // ✨ Only counter updates
name.value = 'Bob'; // ✨ Only name updates
```

**That's it.** Instead of calling `ref()` multiple times, use `refs()` to create several refs at once. Less repetition, cleaner code.

 

## What is `refs()`?

`refs()` is **the bulk constructor for multiple ref() calls**. It takes an object of values and returns an object of refs.

Think of it as **ordering a combo meal** instead of ordering each item separately. Same result, less effort, one convenient call.

**In practical terms:** Instead of writing `const a = ref(1); const b = ref(2); const c = ref(3);`, you write `const { a, b, c } = refs({ a: 1, b: 2, c: 3 });`

 

## Syntax

```javascript
// Basic usage
const myRefs = refs(definitions);

// With destructuring (recommended)
const { count, name } = refs({
  count: 0,
  name: 'Alice'
});

// Without destructuring
const myRefs = refs({ count: 0, name: 'Alice' });
console.log(myRefs.count.value);  // 0
console.log(myRefs.name.value);   // 'Alice'

// Complex example
const {
  counter,
  username,
  isLoggedIn,
  theme
} = refs({
  counter: 0,
  username: '',
  isLoggedIn: false,
  theme: 'light'
});
```

**Parameters:**
- `definitions` - Object where keys are ref names and values are initial values

**Returns:**
- Object where each property is a `ref()` containing the initial value

 

## Why Does This Exist?

### The Problem Without refs()

Let's create several reactive primitives for a form:

```javascript
// ❌ Vanilla JavaScript - no reactivity
let username = '';
let email = '';
let password = '';
let isSubmitting = false;

function updateDisplay() {
  document.getElementById('username-display').textContent = username;
  document.getElementById('email-display').textContent = email;
  document.getElementById('submit-btn').disabled = isSubmitting;
}

// Manual updates required
document.getElementById('username-input').addEventListener('input', (e) => {
  username = e.target.value;
  updateDisplay(); // Must call manually!
});

document.getElementById('email-input').addEventListener('input', (e) => {
  email = e.target.value;
  updateDisplay(); // Must call manually!
});
```

**Problems:**
❌ **No reactivity** - Plain primitives, manual updates required
❌ **Repetitive** - Must call `updateDisplay()` everywhere
❌ **Easy to forget** - Missed update call = stale UI

### The Solution with Individual ref() Calls

```javascript
// ⚠️ Better, but repetitive
const username = ref('');
const email = ref('');
const password = ref('');
const isSubmitting = ref(false);

// Each ref needs separate declaration
// Lots of repetition
```

**Problems:**
⚠️ **Repetitive** - Many `ref()` calls
⚠️ **Verbose** - Each value on its own line
⚠️ **Not grouped** - Related values scattered

### The Best Solution with refs()

```javascript
// ✅ DOM Helpers + Reactive State with refs() - clean and concise
const { username, email, password, isSubmitting } = refs({
  username: '',
  email: '',
  password: '',
  isSubmitting: false
});

// All refs created in one call
// Clean, grouped, easy to read

effect(() => {
  Elements.update({
    usernameDisplay: { textContent: username.value },
    emailDisplay: { textContent: email.value },
    submitBtn: { disabled: isSubmitting.value }
  });
});

// Input handlers using bulk event binding
Elements.update({
  usernameInput: {
    addEventListener: ['input', (e) => {
      username.value = e.target.value;
      // ✨ Only username display updates
    }]
  },
  emailInput: {
    addEventListener: ['input', (e) => {
      email.value = e.target.value;
      // ✨ Only email display updates
    }]
  }
});
```

**What Just Happened?**

```
One refs() Call
    ↓
Creates Multiple Refs
    ↓
Each Independently Reactive
    ↓
Clean, Grouped, Maintainable
```

**Benefits:**
✅ **Less repetition** - One call instead of many
✅ **Grouped values** - Related refs together
✅ **Clean code** - Easy to read and maintain
✅ **Automatic reactivity** - Each ref is fully reactive
✅ **Independent tracking** - Changing one doesn't affect others

 

## Mental Model: Combo Order

Think of `refs()` like **ordering a combo meal** at a restaurant:

**Without refs() (Individual Orders):**
```
┌─────────────────────────────┐
│  Individual Orders          │
│                             │
│  "I'll have a burger"       │
│  "I'll have fries"          │
│  "I'll have a drink"        │
│  "I'll have dessert"        │
│                             │
│  Four separate transactions │
└─────────────────────────────┘
```

**With refs() (Combo Meal):**
```
┌─────────────────────────────┐
│  Combo Meal                 │
│                             │
│  "I'll have the combo #3"   │
│  ✓ Burger                   │
│  ✓ Fries                    │
│  ✓ Drink                    │
│  ✓ Dessert                  │
│                             │
│  One transaction, same items│
└─────────────────────────────┘
```

You get **the same refs**, just with **less repetition** and **cleaner code**.

 

## How Does It Work?

`refs()` is just a **convenience wrapper** that creates multiple `ref()` calls:

```
Your Call
    ↓
refs({ a: 1, b: 2, c: 3 })
    ↓
Internally Does:
    ↓
{
  a: ref(1),
  b: ref(2),
  c: ref(3)
}
    ↓
Returns: Object of refs
```

**Step-by-step:**

1️⃣ **You call refs with an object:**
```javascript
const myRefs = refs({ count: 0, name: 'Alice' });
```

2️⃣ **Internally creates individual refs:**
```javascript
// Like doing:
// {
//   count: ref(0),
//   name: ref('Alice')
// }
```

3️⃣ **You destructure (optional):**
```javascript
const { count, name } = myRefs;
```

4️⃣ **Each ref works independently:**
```javascript
count.value = 5;   // Only affects count
name.value = 'Bob'; // Only affects name
```

**Key Insight:**
`refs()` doesn't create anything special. It's just **syntactic sugar** for creating multiple `ref()` calls. Each returned ref is completely independent.

 

## Basic Usage

### Example 1: Counter with Multiple Values

```javascript
const { count, step, max } = refs({
  count: 0,
  step: 1,
  max: 100
});

effect(() => {
  Elements.update({
    countDisplay: { textContent: count.value },
    stepDisplay: { textContent: `Step: ${step.value}` },
    incrementBtn: { disabled: count.value >= max.value }
  });
});

// Increment using bulk event binding
Elements.update({
  incrementBtn: {
    addEventListener: ['click', () => {
      if (count.value < max.value) {
        count.value += step.value;
      }
    }]
  },
  changeStepBtn: {
    addEventListener: ['click', () => {
      step.value = step.value === 1 ? 5 : 1;
    }]
  }
});
```

 

### Example 2: Form Fields

```javascript
const { username, email, password, confirmPassword } = refs({
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
});

effect(() => {
  const passwordsMatch = password.value === confirmPassword.value;
  const emailValid = email.value.includes('@');

  Elements.update({
    emailError: {
      textContent: email.value && !emailValid ? 'Invalid email' : '',
      style: { display: email.value && !emailValid ? 'block' : 'none' }
    },
    passwordError: {
      textContent: confirmPassword.value && !passwordsMatch
        ? 'Passwords do not match'
        : '',
      style: { display: confirmPassword.value && !passwordsMatch ? 'block' : 'none' }
    }
  });
});

// Input handlers using bulk event binding
Elements.update({
  usernameInput: {
    addEventListener: ['input', (e) => { username.value = e.target.value; }]
  },
  emailInput: {
    addEventListener: ['input', (e) => { email.value = e.target.value; }]
  },
  passwordInput: {
    addEventListener: ['input', (e) => { password.value = e.target.value; }]
  },
  confirmPasswordInput: {
    addEventListener: ['input', (e) => { confirmPassword.value = e.target.value; }]
  }
});
```

 

### Example 3: UI Toggles

```javascript
const { darkMode, sidebarOpen, notificationsEnabled, autoSave } = refs({
  darkMode: false,
  sidebarOpen: true,
  notificationsEnabled: true,
  autoSave: true
});

effect(() => {
  document.body.classList.toggle('dark-mode', darkMode.value);

  Elements.update({
    sidebar: {
      style: { transform: sidebarOpen.value ? 'translateX(0)' : 'translateX(-100%)' }
    },
    darkModeToggle: {
      textContent: darkMode.value ? '☀️ Light' : '🌙 Dark'
    },
    notificationIcon: {
      textContent: notificationsEnabled.value ? '🔔' : '🔕'
    },
    autoSaveIndicator: {
      textContent: autoSave.value ? '✓ Auto-save ON' : 'Auto-save OFF'
    }
  });
});

// Toggle handlers using bulk event binding
Elements.update({
  darkModeToggle: {
    addEventListener: ['click', () => { darkMode.value = !darkMode.value; }]
  },
  sidebarToggle: {
    addEventListener: ['click', () => { sidebarOpen.value = !sidebarOpen.value; }]
  },
  notificationToggle: {
    addEventListener: ['click', () => { notificationsEnabled.value = !notificationsEnabled.value; }]
  },
  autoSaveToggle: {
    addEventListener: ['click', () => { autoSave.value = !autoSave.value; }]
  }
});
```

 

### Example 4: Filter Controls

```javascript
const { searchQuery, minPrice, maxPrice, category, sortBy } = refs({
  searchQuery: '',
  minPrice: 0,
  maxPrice: 1000,
  category: 'all',
  sortBy: 'name'
});

const products = [
  { name: 'Laptop', price: 999, category: 'electronics' },
  { name: 'Mouse', price: 29, category: 'electronics' },
  { name: 'Desk', price: 299, category: 'furniture' },
  { name: 'Chair', price: 199, category: 'furniture' }
];

effect(() => {
  let filtered = products.filter(p => {
    const matchesSearch = p.name.toLowerCase()
      .includes(searchQuery.value.toLowerCase());
    const matchesPrice = p.price >= minPrice.value && p.price <= maxPrice.value;
    const matchesCategory = category.value === 'all' || p.category === category.value;

    return matchesSearch && matchesPrice && matchesCategory;
  });

  if (sortBy.value === 'price') {
    filtered.sort((a, b) => a.price - b.price);
  } else {
    filtered.sort((a, b) => a.name.localeCompare(b.name));
  }

  Elements.update({
    productList: {
      innerHTML: filtered.map(p =>
        `<li>${p.name} - $${p.price}</li>`
      ).join('')
    },
    resultCount: { textContent: `${filtered.length} products` }
  });
});
```

 

### Example 5: Timer Controls

```javascript
const { hours, minutes, seconds, isRunning } = refs({
  hours: 0,
  minutes: 0,
  seconds: 0,
  isRunning: false
});

let interval;

effect(() => {
  const h = hours.value.toString().padStart(2, '0');
  const m = minutes.value.toString().padStart(2, '0');
  const s = seconds.value.toString().padStart(2, '0');

  Elements.update({
    timerDisplay: { textContent: `${h}:${m}:${s}` },
    startBtn: { disabled: isRunning.value },
    pauseBtn: { disabled: !isRunning.value }
  });
});

// Control buttons using bulk event binding
Elements.update({
  startBtn: {
    addEventListener: ['click', () => {
      isRunning.value = true;
      interval = setInterval(() => {
        seconds.value++;
        if (seconds.value >= 60) {
          seconds.value = 0;
          minutes.value++;
        }
        if (minutes.value >= 60) {
          minutes.value = 0;
          hours.value++;
        }
      }, 1000);
    }]
  },
  pauseBtn: {
    addEventListener: ['click', () => {
      isRunning.value = false;
      clearInterval(interval);
    }]
  },
  resetBtn: {
    addEventListener: ['click', () => {
      hours.value = 0;
      minutes.value = 0;
      seconds.value = 0;
      isRunning.value = false;
      clearInterval(interval);
    }]
  }
});
```

 

### Example 6: Pagination Controls

```javascript
const { currentPage, pageSize, totalItems } = refs({
  currentPage: 1,
  pageSize: 10,
  totalItems: 95
});

effect(() => {
  const totalPages = Math.ceil(totalItems.value / pageSize.value);
  const startItem = (currentPage.value - 1) * pageSize.value + 1;
  const endItem = Math.min(currentPage.value * pageSize.value, totalItems.value);

  Elements.update({
    pageInfo: {
      textContent: `Page ${currentPage.value} of ${totalPages}`
    },
    itemRange: {
      textContent: `Showing ${startItem}-${endItem} of ${totalItems.value}`
    },
    prevBtn: { disabled: currentPage.value === 1 },
    nextBtn: { disabled: currentPage.value === totalPages }
  });
});

// Navigation using bulk event binding
Elements.update({
  prevBtn: {
    addEventListener: ['click', () => {
      if (currentPage.value > 1) currentPage.value--;
    }]
  },
  nextBtn: {
    addEventListener: ['click', () => {
      const totalPages = Math.ceil(totalItems.value / pageSize.value);
      if (currentPage.value < totalPages) currentPage.value++;
    }]
  }
});
```

 

### Example 7: Color Mixer

```javascript
const { red, green, blue } = refs({
  red: 127,
  green: 127,
  blue: 127
});

effect(() => {
  const color = `rgb(${red.value}, ${green.value}, ${blue.value})`;
  const hex = '#' +
    [red.value, green.value, blue.value]
      .map(v => v.toString(16).padStart(2, '0'))
      .join('');

  Elements.update({
    colorPreview: {
      style: { backgroundColor: color }
    },
    colorRgb: { textContent: color },
    colorHex: { textContent: hex },
    redValue: { textContent: red.value },
    greenValue: { textContent: green.value },
    blueValue: { textContent: blue.value }
  });
});

// Sliders using bulk event binding
Elements.update({
  redSlider: {
    addEventListener: ['input', (e) => { red.value = parseInt(e.target.value); }]
  },
  greenSlider: {
    addEventListener: ['input', (e) => { green.value = parseInt(e.target.value); }]
  },
  blueSlider: {
    addEventListener: ['input', (e) => { blue.value = parseInt(e.target.value); }]
  }
});
```

 

### Example 8: Score Board

```javascript
const { homeScore, awayScore, quarter, timeLeft } = refs({
  homeScore: 0,
  awayScore: 0,
  quarter: 1,
  timeLeft: 720 // seconds
});

effect(() => {
  const minutes = Math.floor(timeLeft.value / 60);
  const seconds = timeLeft.value % 60;

  Elements.update({
    homeScoreDisplay: { textContent: homeScore.value },
    awayScoreDisplay: { textContent: awayScore.value },
    quarterDisplay: { textContent: `Q${quarter.value}` },
    timeDisplay: {
      textContent: `${minutes}:${seconds.toString().padStart(2, '0')}`
    },
    leadingTeam: {
      textContent: homeScore.value > awayScore.value ? 'Home Leads'
        : awayScore.value > homeScore.value ? 'Away Leads'
        : 'Tied'
    }
  });
});
```

 

### Example 9: Settings Panel

```javascript
const {
  volume,
  brightness,
  contrast,
  fontSize,
  animationSpeed
} = refs({
  volume: 50,
  brightness: 70,
  contrast: 100,
  fontSize: 16,
  animationSpeed: 1
});

effect(() => {
  Elements.update({
    volumeValue: { textContent: `${volume.value}%` },
    brightnessValue: { textContent: `${brightness.value}%` },
    contrastValue: { textContent: `${contrast.value}%` },
    fontSizeValue: { textContent: `${fontSize.value}px` },
    animationSpeedValue: { textContent: `${animationSpeed.value}x` }
  });

  // Apply settings
  document.documentElement.style.setProperty('--font-size', `${fontSize.value}px`);
  document.documentElement.style.setProperty('--animation-speed', `${animationSpeed.value}s`);
});
```

 

### Example 10: Chart Controls

```javascript
const {
  chartType,
  startDate,
  endDate,
  dataPoints,
  showLegend
} = refs({
  chartType: 'line',
  startDate: '2024-01-01',
  endDate: '2024-12-31',
  dataPoints: 50,
  showLegend: true
});

effect(() => {
  Elements.update({
    chartTypeLabel: { textContent: chartType.value.toUpperCase() },
    dateRange: {
      textContent: `${startDate.value} to ${endDate.value}`
    },
    dataPointsLabel: { textContent: `${dataPoints.value} points` },
    legendToggle: {
      textContent: showLegend.value ? 'Hide Legend' : 'Show Legend'
    }
  });

  // Trigger chart update
  updateChart({
    type: chartType.value,
    start: startDate.value,
    end: endDate.value,
    points: dataPoints.value,
    legend: showLegend.value
  });
});

function updateChart(config) {
  // Chart rendering logic...
}
```

 

## refs() vs Individual ref() Calls

### Approach 1: refs() (Recommended for Multiple)
```javascript
const { count, name, active } = refs({
  count: 0,
  name: 'Alice',
  active: true
});
```

### Approach 2: Individual ref() Calls
```javascript
const count = ref(0);
const name = ref('Alice');
const active = ref(true);
```

**Both are equivalent!** Choose based on preference and context.

**When to use refs():**
✅ Creating 3+ refs at once
✅ Related values that belong together
✅ You want grouped, clean declarations
✅ Configuration or settings values

**When to use individual ref():**
✅ Just 1-2 refs
✅ Values are unrelated
✅ Refs added at different times
✅ Personal preference for separate declarations

 

## refs() vs state()

### Using refs() (Multiple Independent Values)
```javascript
const { count, name, active } = refs({
  count: 0,
  name: 'Alice',
  active: true
});

// Each is independent
count.value++; // Only count changes
```

### Using state() (Related Object Properties)
```javascript
const user = state({
  count: 0,
  name: 'Alice',
  active: true
});

// All part of one object
user.count++; // Part of user object
```

**When to use refs():**
✅ Values are independent
✅ Each value needs separate tracking
✅ You prefer `.value` API
✅ Simple primitive values

**When to use state():**
✅ Values are related (e.g., user profile)
✅ You want direct property access
✅ Complex or nested data
✅ Values that change together

 

## Common Patterns

### Pattern 1: Form State

```javascript
const {
  email,
  password,
  rememberMe,
  isSubmitting
} = refs({
  email: '',
  password: '',
  rememberMe: false,
  isSubmitting: false
});
```

### Pattern 2: UI Flags

```javascript
const {
  isLoading,
  hasError,
  isDirty,
  isVisible
} = refs({
  isLoading: false,
  hasError: false,
  isDirty: false,
  isVisible: true
});
```

### Pattern 3: Filter State

```javascript
const {
  search,
  category,
  minPrice,
  maxPrice,
  sortBy
} = refs({
  search: '',
  category: 'all',
  minPrice: 0,
  maxPrice: 1000,
  sortBy: 'name'
});
```

 

## Key Takeaways

✅ **Bulk creation** - Create multiple refs in one call
✅ **Less repetition** - Cleaner than many ref() calls
✅ **Grouped values** - Related refs stay together
✅ **Independent tracking** - Each ref works separately
✅ **Destructuring friendly** - Easy to extract what you need
✅ **Syntactic sugar** - Same as individual ref() calls, just cleaner

 

## What's Next?

Now that you understand `refs()`, explore:

- **`ref()`** - Create single reactive primitives
- **`state()`** - For objects and complex data
- **`createCollection()`** - Reactive arrays with methods
- **`builder()`** - Chainable state construction

 

## Summary

`refs()` is the **bulk constructor for multiple refs**. Instead of calling `ref()` many times, you define all your refs in one object and destructure them.

**The magic formula:**
```
refs({ a: 1, b: 2, c: 3 })
=
{ a: ref(1), b: ref(2), c: ref(3) }
─────────────────────────────────
Multiple refs in one clean call
```

Think of it as **ordering a combo meal** — you get all the items you need in one convenient package, instead of ordering each separately. Same refs, less repetition, cleaner code.
