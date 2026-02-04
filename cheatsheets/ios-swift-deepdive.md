# iOS / Swift Deep Dive Cheat Sheet

---

## 🧠 Memory Management

### ARC (Automatic Reference Counting)
Swift uses ARC to track and manage memory. Every time you create a class instance, ARC allocates memory. When that instance is no longer needed, ARC frees it.

**How it works:**
- Each object has a **retain count**
- `strong` reference → count +1
- Object deallocated when count reaches 0

### Reference Types

```swift
class Person {
    var name: String
    var friend: Person?  // Strong by default
    
    init(name: String) { self.name = name }
    deinit { print("\(name) is being deallocated") }
}
```

**Strong (default)**
```swift
var person: Person? = Person(name: "Aaron")
// Retain count = 1
person = nil
// Retain count = 0 → deallocated
```

**Weak** — Doesn't increase retain count, becomes nil when object deallocates
```swift
weak var delegate: SomeDelegate?
// Use for: delegates, parent references, breaking cycles
// Must be optional (can become nil)
// Must be var (value changes)
```

**Unowned** — Like weak but assumes object will always exist
```swift
unowned let owner: Person
// Use when: You're certain the referenced object outlives this one
// Crashes if accessed after deallocation
// Can be non-optional
```

### Retain Cycles (Memory Leaks)
When two objects hold strong references to each other:

```swift
class Person {
    var apartment: Apartment?
}

class Apartment {
    var tenant: Person?  // 💥 Retain cycle!
}

let aaron = Person()
let flat = Apartment()
aaron.apartment = flat
flat.tenant = aaron
// Neither can be deallocated!
```

**Fix with weak:**
```swift
class Apartment {
    weak var tenant: Person?  // ✅ No cycle
}
```

### Closures & Retain Cycles
Closures capture references strongly by default:

```swift
class ViewController {
    var name = "Main"
    
    func setupHandler() {
        // 💥 self captured strongly
        someService.onComplete = {
            print(self.name)
        }
    }
}
```

**Fix with capture list:**
```swift
someService.onComplete = { [weak self] in
    guard let self = self else { return }
    print(self.name)
}

// Or unowned if you're certain:
someService.onComplete = { [unowned self] in
    print(self.name)
}
```

---

## 🔄 Concurrency

### The Problem
- UI must run on **main thread**
- Network/disk operations block the thread
- Blocking main thread = frozen UI

### GCD (Grand Central Dispatch)
The "old" way — still widely used:

```swift
// Background work
DispatchQueue.global(qos: .background).async {
    let data = fetchDataFromNetwork()
    
    // Update UI on main thread
    DispatchQueue.main.async {
        self.label.text = data
    }
}
```

**Quality of Service (QoS):**
- `.userInteractive` — Immediate (animations)
- `.userInitiated` — User waiting (opening document)
- `.default` — Standard
- `.utility` — Long tasks with progress (downloads)
- `.background` — User not waiting (backup, sync)

### async/await (iOS 15+)
Modern, cleaner approach:

```swift
func fetchUser() async throws -> User {
    let url = URL(string: "https://api.example.com/user")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}

// Calling it:
Task {
    do {
        let user = try await fetchUser()
        // Already on main thread for UI updates in SwiftUI
        self.user = user
    } catch {
        print(error)
    }
}
```

**Parallel execution:**
```swift
async let profile = fetchProfile()
async let posts = fetchPosts()
async let friends = fetchFriends()

// All three run in parallel, await all:
let data = try await (profile, posts, friends)
```

### Actors (Thread-Safe Classes)
Actors protect mutable state from data races:

```swift
actor BankAccount {
    private var balance: Double = 0
    
    func deposit(_ amount: Double) {
        balance += amount
    }
    
    func getBalance() -> Double {
        return balance
    }
}

// Usage (must await):
let account = BankAccount()
await account.deposit(100)
let balance = await account.getBalance()
```

**@MainActor** — Runs on main thread:
```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var items: [Item] = []
    
    func load() async {
        let data = await fetchItems()  // Background
        items = data  // Main thread (guaranteed)
    }
}
```

---

## 🏗️ Architecture Patterns

### MVC (Model-View-Controller)
Apple's default, simple but can lead to "Massive View Controller":

```
┌─────────┐     ┌────────────┐     ┌──────┐
│  Model  │◄───►│ Controller │◄───►│ View │
└─────────┘     └────────────┘     └──────┘
```

- **Model** — Data & business logic
- **View** — UI (UIKit views, storyboards)
- **Controller** — Glues them together, handles events

**Problem:** Controllers grow huge, hard to test.

### MVVM (Model-View-ViewModel)
Better separation, testable:

```
┌─────────┐     ┌───────────┐     ┌──────┐
│  Model  │◄───►│ ViewModel │◄───►│ View │
└─────────┘     └───────────┘     └──────┘
                      │
                 (Data Binding)
```

```swift
// Model
struct User {
    let name: String
    let email: String
}

// ViewModel
class UserViewModel: ObservableObject {
    @Published var displayName: String = ""
    @Published var isLoading: Bool = false
    
    private let userService: UserService
    
    func loadUser() async {
        isLoading = true
        let user = await userService.fetchUser()
        displayName = user.name
        isLoading = false
    }
}

// View (SwiftUI)
struct UserView: View {
    @StateObject var viewModel = UserViewModel()
    
    var body: some View {
        if viewModel.isLoading {
            ProgressView()
        } else {
            Text(viewModel.displayName)
        }
    }
}
```

**Benefits:**
- ViewModel has no UIKit/SwiftUI imports → testable
- View is dumb — just renders state
- Clear data flow

---

## 🎨 SwiftUI State Management

### Property Wrappers

**@State** — Local value, owned by this view
```swift
struct CounterView: View {
    @State private var count = 0  // Source of truth
    
    var body: some View {
        Button("Count: \(count)") {
            count += 1
        }
    }
}
```

**@Binding** — Reference to state owned elsewhere
```swift
struct ChildView: View {
    @Binding var count: Int  // Borrowed from parent
    
    var body: some View {
        Button("Increment") { count += 1 }
    }
}

// Parent passes binding:
ChildView(count: $count)
```

**@StateObject** — Create & own an ObservableObject
```swift
struct ParentView: View {
    @StateObject var viewModel = MyViewModel()  // Created once
}
```

**@ObservedObject** — Observe an ObservableObject you don't own
```swift
struct ChildView: View {
    @ObservedObject var viewModel: MyViewModel  // Passed in
}
```

**@EnvironmentObject** — Shared across view hierarchy
```swift
// At root:
ContentView()
    .environmentObject(appState)

// Any child:
struct DeepChild: View {
    @EnvironmentObject var appState: AppState
}
```

**When to use what:**
| Wrapper | Use When |
|---------|----------|
| @State | Simple local value (toggles, text) |
| @Binding | Child needs to modify parent's state |
| @StateObject | View creates & owns the object |
| @ObservedObject | Object passed from parent |
| @EnvironmentObject | Shared app-wide state |

---

## 🌐 Networking

### URLSession Basics
```swift
func fetchUsers() async throws -> [User] {
    let url = URL(string: "https://api.example.com/users")!
    
    var request = URLRequest(url: url)
    request.httpMethod = "GET"
    request.addValue("application/json", forHTTPHeaderField: "Content-Type")
    
    let (data, response) = try await URLSession.shared.data(for: request)
    
    guard let httpResponse = response as? HTTPURLResponse,
          httpResponse.statusCode == 200 else {
        throw NetworkError.badResponse
    }
    
    return try JSONDecoder().decode([User].self, from: data)
}
```

### Codable (JSON Parsing)
```swift
struct User: Codable {
    let id: Int
    let name: String
    let email: String
    
    // Custom keys:
    enum CodingKeys: String, CodingKey {
        case id
        case name = "full_name"
        case email = "email_address"
    }
}

// Decode:
let user = try JSONDecoder().decode(User.self, from: jsonData)

// Encode:
let data = try JSONEncoder().encode(user)
```

---

## 💾 Data Persistence

### UserDefaults — Small key-value data
```swift
// Save
UserDefaults.standard.set("Aaron", forKey: "username")
UserDefaults.standard.set(true, forKey: "isDarkMode")

// Read
let name = UserDefaults.standard.string(forKey: "username")
let isDark = UserDefaults.standard.bool(forKey: "isDarkMode")
```

### Keychain — Secure data (tokens, passwords)
```swift
// Use a wrapper library like KeychainAccess
let keychain = Keychain(service: "com.yourapp")
try keychain.set(token, key: "authToken")
let token = try keychain.get("authToken")
```

### Core Data — Complex relational data
```swift
// Fetch
let request = NSFetchRequest<User>(entityName: "User")
request.predicate = NSPredicate(format: "age > %d", 18)
let users = try context.fetch(request)

// Create
let user = User(context: context)
user.name = "Aaron"

// Save
try context.save()
```

### SwiftData (iOS 17+) — Modern replacement
```swift
@Model
class User {
    var name: String
    var email: String
    
    init(name: String, email: String) {
        self.name = name
        self.email = email
    }
}

// In view:
@Query var users: [User]

// Insert:
modelContext.insert(User(name: "Aaron", email: "a@b.com"))
```

---

## 📱 App Lifecycle

### UIKit (AppDelegate / SceneDelegate)
```swift
class AppDelegate: UIResponder, UIApplicationDelegate {
    func application(_ application: UIApplication, 
                     didFinishLaunchingWithOptions: ...) -> Bool {
        // App launched
        return true
    }
}

class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    func sceneDidBecomeActive(_ scene: UIScene) {
        // Foreground, active
    }
    
    func sceneWillResignActive(_ scene: UIScene) {
        // About to go inactive (call, notification)
    }
    
    func sceneDidEnterBackground(_ scene: UIScene) {
        // Background — save state
    }
}
```

### SwiftUI
```swift
@main
struct MyApp: App {
    @Environment(\.scenePhase) var scenePhase
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .onChange(of: scenePhase) { phase in
            switch phase {
            case .active: print("Active")
            case .inactive: print("Inactive")
            case .background: print("Background")
            }
        }
    }
}
```

---

## 🧪 Testing

### Unit Tests
```swift
import XCTest
@testable import MyApp

class UserViewModelTests: XCTestCase {
    func testDisplayNameFormatting() {
        let vm = UserViewModel()
        vm.user = User(firstName: "Aaron", lastName: "Strickland")
        
        XCTAssertEqual(vm.displayName, "Aaron Strickland")
    }
    
    func testLoadingState() async {
        let vm = UserViewModel(service: MockUserService())
        
        XCTAssertFalse(vm.isLoading)
        
        await vm.loadUser()
        
        XCTAssertFalse(vm.isLoading)
        XCTAssertNotNil(vm.user)
    }
}
```

### UI Tests
```swift
func testLoginFlow() {
    let app = XCUIApplication()
    app.launch()
    
    app.textFields["emailField"].tap()
    app.textFields["emailField"].typeText("test@example.com")
    
    app.buttons["loginButton"].tap()
    
    XCTAssertTrue(app.staticTexts["Welcome"].exists)
}
```

---

*You know this stuff — you've shipped 5 apps. This is just a refresher. 💪*
