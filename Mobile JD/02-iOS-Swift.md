---
title: iOS Development with Swift
date: 2026-01-31
tags:
  - ios
  - swift
  - interview
---

# iOS Development with Swift

## Swift Fundamentals

### Optionals

```swift
var name: String? = nil  // Optional String

// Unwrapping
if let unwrapped = name {
    print(unwrapped)
}

// Guard statement
guard let unwrapped = name else { return }

// Nil coalescing
let displayName = name ?? "Unknown"

// Optional chaining
let count = name?.count
```

### Value Types vs Reference Types

```swift
// Struct (Value Type) - copied when assigned
struct Point {
    var x: Int
    var y: Int
}

// Class (Reference Type) - shared reference
class Person {
    var name: String
    init(name: String) { self.name = name }
}
```

### Closures

```swift
// Basic closure
let greet = { (name: String) -> String in
    return "Hello, \(name)"
}

// Trailing closure syntax
numbers.map { $0 * 2 }

// Escaping closures
func fetchData(completion: @escaping (Result) -> Void) {
    // Closure called after function returns
}
```

### Protocols

```swift
protocol Drawable {
    func draw()
}

extension Drawable {
    func draw() {
        print("Default drawing")
    }
}

struct Circle: Drawable {
    func draw() {
        print("Drawing circle")
    }
}
```

### Generics

```swift
func swap<T>(_ a: inout T, _ b: inout T) {
    let temp = a
    a = b
    b = temp
}
```

## UIKit Fundamentals

### View Controller Lifecycle

```
┌──────────────────┐
│ loadView()       │ ─── Create view hierarchy
└────────┬─────────┘
         ▼
┌──────────────────┐
│ viewDidLoad()    │ ─── View loaded into memory
└────────┬─────────┘
         ▼
┌──────────────────┐
│ viewWillAppear() │ ─── About to be visible
└────────┬─────────┘
         ▼
┌──────────────────┐
│ viewDidAppear()  │ ─── Now visible
└────────┬─────────┘
         │ User navigates away
         ▼
┌──────────────────────┐
│ viewWillDisappear()  │ ─── About to be hidden
└────────┬─────────────┘
         ▼
┌──────────────────────┐
│ viewDidDisappear()   │ ─── Now hidden
└──────────────────────┘
```

### Auto Layout

```swift
// Programmatic constraints
NSLayoutConstraint.activate([
    view.topAnchor.constraint(equalTo: superview.topAnchor, constant: 20),
    view.leadingAnchor.constraint(equalTo: superview.leadingAnchor, constant: 16),
    view.trailingAnchor.constraint(equalTo: superview.trailingAnchor, constant: -16),
    view.heightAnchor.constraint(equalToConstant: 50)
])

// Don't forget
view.translatesAutoresizingMaskIntoConstraints = false
```

### TableView & CollectionView

```swift
class MyViewController: UIViewController, UITableViewDataSource, UITableViewDelegate {
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return items.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath)
        cell.textLabel?.text = items[indexPath.row]
        return cell
    }
    
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        // Handle selection
    }
}
```

### Storyboard vs Programmatic UI

| Storyboard | Programmatic |
|------------|--------------|
| Visual design | Full control |
| Quick prototyping | Better for teams (no merge conflicts) |
| Segues for navigation | More flexible |
| Can be slow for large projects | Easier to reuse |

## Memory Management

### ARC (Automatic Reference Counting)

```swift
class Person {
    var apartment: Apartment?
}

class Apartment {
    weak var tenant: Person?  // Weak reference to break retain cycle
}

// Strong - default, increases reference count
// Weak - doesn't increase count, becomes nil when deallocated
// Unowned - doesn't increase count, assumes always valid
```

### Retain Cycles in Closures

```swift
class ViewController {
    var name = "Test"
    
    func setupClosure() {
        // Capture list to avoid retain cycle
        someClosure = { [weak self] in
            guard let self = self else { return }
            print(self.name)
        }
    }
}
```

## Networking

### URLSession

```swift
func fetchData() async throws -> [User] {
    let url = URL(string: "https://api.example.com/users")!
    let (data, response) = try await URLSession.shared.data(from: url)
    
    guard let httpResponse = response as? HTTPURLResponse,
          httpResponse.statusCode == 200 else {
        throw NetworkError.invalidResponse
    }
    
    return try JSONDecoder().decode([User].self, from: data)
}
```

### Codable

```swift
struct User: Codable {
    let id: Int
    let name: String
    let email: String
    
    enum CodingKeys: String, CodingKey {
        case id
        case name = "user_name"
        case email
    }
}
```

## CoreData

### Stack Setup

```swift
lazy var persistentContainer: NSPersistentContainer = {
    let container = NSPersistentContainer(name: "Model")
    container.loadPersistentStores { _, error in
        if let error = error {
            fatalError("CoreData error: \(error)")
        }
    }
    return container
}()

var context: NSManagedObjectContext {
    return persistentContainer.viewContext
}
```

### CRUD Operations

```swift
// Create
let user = User(context: context)
user.name = "John"
try context.save()

// Read
let request: NSFetchRequest<User> = User.fetchRequest()
request.predicate = NSPredicate(format: "name == %@", "John")
let users = try context.fetch(request)

// Update
user.name = "Jane"
try context.save()

// Delete
context.delete(user)
try context.save()
```

## SwiftUI Basics

```swift
struct ContentView: View {
    @State private var count = 0
    
    var body: some View {
        VStack {
            Text("Count: \(count)")
            Button("Increment") {
                count += 1
            }
        }
    }
}

// Property Wrappers
// @State - local state
// @Binding - two-way connection
// @ObservedObject - external observable object
// @EnvironmentObject - shared data
// @StateObject - owned observable object
```

## Debugging Tools

- **Xcode Instruments** - Performance profiling
- **Memory Graph Debugger** - Find retain cycles
- **View Debugger** - 3D view hierarchy
- **Breakpoints** - Conditional, symbolic
- **LLDB** - Command-line debugging

## Questions & Answers

> [!question]- Q1: What is the difference between `struct` and `class` in Swift?
> **Answer:** 
> | Struct | Class |
> |--------|-------|
> | Value type (copied) | Reference type (shared) |
> | No inheritance | Supports inheritance |
> | No deinitializer | Has deinitializer |
> | Memberwise initializer | Must define initializer |
> 
> Use structs by default; use classes when you need inheritance or reference semantics.

> [!question]- Q2: Explain ARC and how to avoid retain cycles.
> **Answer:** 
> ARC (Automatic Reference Counting) manages memory by tracking references to objects. When count reaches 0, object is deallocated.
> 
> Avoid retain cycles using:
> - `weak` - Reference becomes nil when object deallocates
> - `unowned` - Assumes object always exists
> - Capture lists in closures: `[weak self]`

> [!question]- Q3: What is the difference between `weak` and `unowned`?
> **Answer:** 
> - **weak**: Optional reference, becomes nil when object deallocates. Use when reference might become nil.
> - **unowned**: Non-optional, crashes if accessed after deallocation. Use when you're certain the reference will always be valid.

> [!question]- Q4: Explain the iOS View Controller lifecycle.
> **Answer:** 
> Key methods in order:
> 1. `loadView()` - Create view hierarchy
> 2. `viewDidLoad()` - View loaded, one-time setup
> 3. `viewWillAppear()` - About to appear
> 4. `viewDidAppear()` - Fully visible
> 5. `viewWillDisappear()` - About to hide
> 6. `viewDidDisappear()` - Hidden

> [!question]- Q5: What is the difference between `frame` and `bounds`?
> **Answer:** 
> - **frame**: Position and size relative to superview's coordinate system
> - **bounds**: Position and size in the view's own coordinate system
> 
> Frame changes when you move/resize a view. Bounds origin is usually (0,0) unless scrolled.

> [!question]- Q6: What are property wrappers in SwiftUI?
> **Answer:** 
> Property wrappers manage state in SwiftUI:
> - `@State` - Local mutable state
> - `@Binding` - Two-way connection to parent's state
> - `@ObservedObject` - External observable object
> - `@StateObject` - Owned observable object (created once)
> - `@EnvironmentObject` - Shared across view hierarchy

> [!question]- Q7: How does delegation pattern work in iOS?
> **Answer:** 
> Delegation allows one object to communicate with another through a protocol.
> 
> Steps:
> 1. Define a protocol with required methods
> 2. Create a weak delegate property
> 3. Call delegate methods when events occur
> 4. Conforming class implements the protocol
> 
> Example: UITableViewDelegate, UITextFieldDelegate

> [!question]- Q8: What is the difference between `escaping` and `non-escaping` closures?
> **Answer:** 
> - **Non-escaping** (default): Closure is called within the function's scope
> - **Escaping** (`@escaping`): Closure is stored and called after function returns
> 
> Use `@escaping` for async callbacks, stored closures, or completion handlers.

> [!question]- Q9: Explain CoreData stack components.
> **Answer:** 
> - **NSManagedObjectModel** - Data model (entities, attributes)
> - **NSPersistentStoreCoordinator** - Mediates between store and context
> - **NSManagedObjectContext** - Scratchpad for working with objects
> - **NSPersistentContainer** - Simplifies stack setup (iOS 10+)

> [!question]- Q10: What is GCD (Grand Central Dispatch)?
> **Answer:** 
> GCD manages concurrent operations using dispatch queues.
> 
> Queue types:
> - **Main queue** - UI updates (serial)
> - **Global queues** - Background work (concurrent)
> - **Custom queues** - Your own serial/concurrent queues
> 
> ```swift
> DispatchQueue.main.async { /* UI work */ }
> DispatchQueue.global(qos: .background).async { /* Background work */ }
> ```
