# Mobile Interview Cheat Sheet
## Android (Java) + iOS (Swift) Essentials

---

# 🤖 ANDROID (Java)

## Activity Lifecycle
```
onCreate → onStart → onResume → [RUNNING] → onPause → onStop → onDestroy
```
- **onCreate**: Initialize, setContentView, bind views
- **onStart**: Visible but not interactive
- **onResume**: Visible AND interactive (foreground)
- **onPause**: Losing focus (dialog, another activity)
- **onStop**: No longer visible (home button, new activity)
- **onDestroy**: Being killed (finish() or system)

**Rotation** triggers: onPause → onStop → onDestroy → onCreate → onStart → onResume

## The 4 Components
| Component | Purpose | Started By |
|-----------|---------|------------|
| **Activity** | Single screen with UI | Intent |
| **Service** | Background work, no UI | startService / bindService |
| **BroadcastReceiver** | Responds to system events | System or sendBroadcast |
| **ContentProvider** | Share data between apps | ContentResolver |

## Services: Started vs Bound
- **Started**: `startService()` → runs independently → call `stopSelf()` when done
- **Bound**: `bindService()` → client-server connection → dies when all unbind

## Threading
❌ **Never touch UI from background thread** — UI toolkit isn't thread-safe

Get back to main thread:
```java
runOnUiThread(() -> { /* update UI */ });
new Handler(Looper.getMainLooper()).post(() -> { });
```

## Context
- **Activity context**: Dies with Activity — use for UI, dialogs, toasts
- **Application context**: Lives forever — use for singletons, databases, long-lived objects

⚠️ Passing Activity context to singleton = **memory leak**

## AsyncTask Memory Leak
Inner class holds reference to outer Activity. If Activity dies but task runs → leak.

**Fix**: Use `static` inner class + `WeakReference<Activity>`

## Intent
- **Explicit**: Target specific component (`new Intent(this, OtherActivity.class)`)
- **Implicit**: Declare action, system finds handler (`ACTION_VIEW`, `ACTION_SEND`)

## Fragment Lifecycle
Same as Activity plus:
- **onAttach**: Connected to Activity
- **onCreateView**: Inflate layout
- **onDestroyView**: View destroyed (but fragment may live)
- **onDetach**: Disconnected from Activity

---

# 🍎 iOS (Swift)

## App Lifecycle (UIKit)
```
Not Running → Inactive → Active → Background → Suspended
```
AppDelegate methods:
- **didFinishLaunching**: App starting, setup
- **willResignActive**: About to lose focus (call coming)
- **didEnterBackground**: Now in background
- **willEnterForeground**: Coming back
- **didBecomeActive**: Now active, running

## ViewController Lifecycle
```
loadView → viewDidLoad → viewWillAppear → viewDidAppear → viewWillDisappear → viewDidDisappear
```
- **viewDidLoad**: Once, after view loaded — setup UI
- **viewWillAppear**: Every time before visible — refresh data
- **viewDidAppear**: Now visible — start animations
- **viewWillDisappear**: About to hide — save state
- **viewDidDisappear**: Hidden — stop tasks

## Memory Management (ARC)
**Automatic Reference Counting** — compiler inserts retain/release

- **strong** (default): Keeps object alive
- **weak**: Optional reference, becomes nil when deallocated
- **unowned**: Non-optional, assumes object outlives reference

## Retain Cycles
Two objects hold strong references to each other → neither deallocates → **leak**

Common in closures:
```swift
// ❌ Leak
someClosure { self.doThing() }

// ✅ Fixed
someClosure { [weak self] in self?.doThing() }
```

## Concurrency

### GCD (Grand Central Dispatch)
```swift
// Background work
DispatchQueue.global().async {
    let data = fetchData()
    // Back to main for UI
    DispatchQueue.main.async {
        self.updateUI(data)
    }
}
```

### async/await (iOS 15+)
```swift
Task {
    let data = await fetchData()  // Suspends, doesn't block
    updateUI(data)  // Automatically main thread in @MainActor
}
```

## Delegates vs Closures
- **Delegate**: Protocol, one-to-one, good for multiple callbacks
- **Closure**: Inline, good for single callback, watch for retain cycles

## Optionals
```swift
var name: String? = nil   // Might be nil
var name: String = ""     // Never nil

name!                     // Force unwrap — crashes if nil
name?                     // Optional chaining — returns nil if nil
name ?? "default"         // Nil coalescing — fallback value
if let name = name { }    // Safe unwrap
guard let name = name else { return }  // Early exit
```

## Common Patterns
- **MVC**: Model-View-Controller (Apple default, massive VC problem)
- **MVVM**: Model-View-ViewModel (better separation, Combine/bindings)
- **Coordinator**: Handles navigation outside VCs
- **Dependency Injection**: Pass dependencies in, don't create inside

---

# 🌐 GENERAL

## REST API
- **GET**: Read data
- **POST**: Create data
- **PUT**: Update (replace) data
- **PATCH**: Update (partial) data
- **DELETE**: Remove data

## HTTP Status Codes
- **2xx**: Success (200 OK, 201 Created)
- **4xx**: Client error (400 Bad Request, 401 Unauthorized, 404 Not Found)
- **5xx**: Server error (500 Internal Server Error)

## SOLID Principles
- **S**ingle Responsibility: One class, one job
- **O**pen/Closed: Open for extension, closed for modification
- **L**iskov Substitution: Subtypes replaceable for parent types
- **I**nterface Segregation: Small focused interfaces
- **D**ependency Inversion: Depend on abstractions, not concretions

## Git Basics
- `git pull` — fetch + merge remote changes
- `git stash` — temporarily save uncommitted changes
- `git rebase` — replay commits on new base (cleaner history)
- `git merge` — combine branches (preserves history)

---

*You've built apps in production. You know this stuff — now you can name it.* 💪
