# Interview Cheat Sheet — Daffodil IT
**Date:** Feb 5, 2026 | **Role:** Mobile App Developer

---

## 🚀 Your Portfolio Highlights

| App | Stack | Talking Points |
|-----|-------|----------------|
| **eLOQ** | Swift, Kotlin, BLE | Chinese SDK integration, audit trails, migrated legacy Obj-C/Java |
| **Unify Messenger** | Swift, Kotlin, WebSocket | Secure messaging for prisons, compliance requirements |
| **Ordernise** | SwiftUI, CloudKit, StoreKit 2 | Built for personal need, multi-platform sync |
| **Powerful Reports** | Swift, Kotlin, Jetpack Compose | Analytics app for wife's business |
| **Oddly Enough** | React Native, Expo, Node.js | Cross-platform, content aggregation |

**Portfolio URL:** https://aaron-strickland.vercel.app

---

## 📱 iOS / Swift Quick Hits

### Architecture
- **MVC** — Model-View-Controller (Apple default)
- **MVVM** — Model-View-ViewModel (better testability)
- **Clean Architecture** — Use cases, repositories, separation of concerns

### SwiftUI vs UIKit
| SwiftUI | UIKit |
|---------|-------|
| Declarative | Imperative |
| @State, @Binding, @ObservedObject | Delegates, IBOutlet |
| Live previews | Storyboards/XIBs |
| iOS 13+ | All iOS versions |

### Memory Management
- **ARC** — Automatic Reference Counting
- **Strong** — Default, increases retain count
- **Weak** — Doesn't increase count, becomes nil
- **Unowned** — Like weak but crashes if nil (use when guaranteed lifetime)
- **Retain cycle** — Two objects holding strong refs → use weak/unowned

### Concurrency
```swift
// Modern (iOS 15+)
Task { await fetchData() }
async let result = fetch()

// GCD
DispatchQueue.main.async { }
DispatchQueue.global(qos: .background).async { }
```

### Common Patterns
- **Delegate** — One-to-one communication
- **Closures/Callbacks** — Async completion handlers
- **NotificationCenter** — One-to-many broadcast
- **Combine** — Reactive streams (Publisher/Subscriber)

### Core Data vs SwiftData
- **Core Data** — Mature, complex, works with UIKit
- **SwiftData** — Modern, Swift-native, iOS 17+

---

## 🤖 Android / Kotlin Quick Hits

### Architecture
- **MVVM** — ViewModel + LiveData/StateFlow
- **Clean Architecture** — Domain/Data/Presentation layers
- **Repository pattern** — Abstract data sources

### Jetpack Compose vs XML
| Compose | XML |
|---------|-----|
| Declarative | Imperative |
| remember, mutableStateOf | findViewById, DataBinding |
| @Composable functions | Activity/Fragment + layouts |

### Lifecycle
```
onCreate → onStart → onResume → [RUNNING]
→ onPause → onStop → onDestroy
```

### Coroutines
```kotlin
lifecycleScope.launch {
    val data = withContext(Dispatchers.IO) { fetchData() }
    updateUI(data)
}
```
- **Dispatchers.Main** — UI thread
- **Dispatchers.IO** — Network/disk
- **Dispatchers.Default** — CPU-heavy

### Common Components
- **ViewModel** — Survives config changes
- **LiveData/StateFlow** — Observable data holders
- **Room** — SQLite abstraction
- **Retrofit** — HTTP client
- **Hilt/Dagger** — Dependency injection

---

## 🔧 Cross-Platform (React Native)

- **JavaScript/TypeScript** core
- **Native bridge** for platform APIs
- **Expo** — Managed workflow, easier setup
- **Bare RN** — Full control, native modules
- **State:** useState, Redux, Zustand
- **Navigation:** React Navigation

---

## 💬 Behavioral Questions (STAR Method)

**S**ituation → **T**ask → **A**ction → **R**esult

### "Tell me about a challenging bug"
> **S:** eLOQ Android had a 20-second black screen after PIN entry.
> **T:** Users couldn't access the app, needed urgent fix.
> **A:** Profiled with Android Studio, found `verifyDevice()` blocking main thread. Moved to coroutine with `Dispatchers.IO`.
> **R:** Fixed the freeze, improved UX, modernized the codebase to Kotlin.

### "Tell me about working with a difficult API/SDK"
> **S:** eLOQ required Chinese manufacturer's BLE SDK.
> **T:** Documentation was poor, needed to integrate smart key programming.
> **A:** Worked directly with their engineers, reverse-engineered some parts, built wrapper classes.
> **R:** Successfully integrated, app now used across retail, education, healthcare.

### "Why mobile development?"
> Love the immediacy — build something, put it in someone's hand.
> Enjoy the constraints — performance, battery, offline.
> Seeing apps on the App Store with real users is satisfying.

### "Why Daffodil IT?"
> [Research them — what do they build? Who are their clients?]
> Interested in working with a team on larger projects.
> Want to grow beyond solo development.

---

## ❓ Questions to Ask Them

**About the role:**
- What does a typical project look like?
- What's the team structure? How many devs?
- iOS, Android, or both? Native or cross-platform?

**About the tech:**
- What's the current tech stack?
- Any plans to adopt SwiftUI/Compose more?
- How do you handle CI/CD?

**About growth:**
- How do you support developer learning?
- What does success look like in 6 months?

**Red flag detectors:**
- How's work-life balance here?
- What happened to the last person in this role?

---

## 🎯 Quick Reminders

- **Breathe** — Pause before answering
- **It's OK to say "I don't know"** — Follow with "but I'd approach it by..."
- **Show curiosity** — Ask follow-up questions
- **Be honest about experience** — Don't oversell
- **Bring examples** — Portfolio, GitHub, apps on your phone

---

## 📞 Logistics

- **Company:** Daffodil IT
- **Date:** Feb 5, 2026
- **Portfolio:** https://aaron-strickland.vercel.app
- **GitHub:** github.com/aarons2222
- **Email:** aaron.strickland@icloud.com

---

*Good luck! You've shipped 5 apps — you know your stuff. 💪*
