# ``ModelActorX``

ModelActorX is a Swift library that provides custom macros `ModelActorX` and `MainModelActorX` to enhance and extend the functionality of SwiftData's `ModelActor`. These macros offer additional flexibility by allowing developers to control the generation of initializers and to declare additional variables within actors and classes.

## Features

- **ModelActorX Macro**: Similar to SwiftData's `ModelActor`, with an added `disableGenerateInit` parameter to control initializer generation.
- **MainModelActorX Macro**: Generates a class running on the `MainActor`, ideal for main-thread operations, and uses `ModelContainer`'s `mainContext`.
- **Custom Initializers**: Ability to declare additional variables and pass them through custom initializers when `disableGenerateInit` is set to `true`.
- **Seamless Integration**: Designed to work seamlessly with SwiftData and existing Swift projects.

## Installation

### Swift Package Manager

Add ModelActorX to your project using Swift Package Manager. In your `Package.swift` file, add:

```swift
dependencies: [
    .package(url: "https://github.com/fatbobman/ModelActorX.git", from: "0.1.0")
]
```

Alternatively, in Xcode:

1. Go to **File** > **Add Packages...**
2. Enter the repository URL: `https://github.com/fatbobman/ModelActorX.git`
3. Follow the prompts to add the package to your project.

## Usage

### ModelActorX

The `ModelActorX` macro is used to define an actor with functionality similar to SwiftData's `ModelActor`. The key difference is the `disableGenerateInit` parameter, which, when set to `true`, prevents the automatic generation of an initializer. This allows you to declare additional variables and provide a custom initializer.

#### Basic Usage

```swift
@ModelActorX
actor DataHandler {
    func newItem(date: Date) throws -> PersistentIdentifier {
        let item = Item(timestamp: date)
        modelContext.insert(item)
        try modelContext.save()
        return item.persistentModelID
    }

    func getTimestampFromItemID(_ itemID: PersistentIdentifier) -> Date? {
        return self[itemID, as: Item.self]?.timestamp
    }
}
```

#### Custom Initializer

```swift
@ModelActorX(disableGenerateInit: true)
actor DataHandler1 {
    let date: Date

    func newItem() throws -> PersistentIdentifier {
        let item = Item(timestamp: date)
        modelContext.insert(item)
        try modelContext.save()
        return item.persistentModelID
    }

    func getTimestampFromItemID(_ itemID: PersistentIdentifier) -> Date? {
        return self[itemID, as: Item.self]?.timestamp
    }

    init(container: ModelContainer, date: Date) {
        self.date = date
        modelContainer = container
        let modelContext = ModelContext(modelContainer)
        modelExecutor = DefaultSerialModelExecutor(modelContext: modelContext)
    }
}
```

### MainModelActorX

The `MainModelActorX` macro is used to generate a class that runs on the `MainActor`. This is particularly useful for UI updates or any operations that need to be performed on the main thread. The generated class uses the `mainContext` from `ModelContainer`.

#### Basic Usage

```swift
@MainActor
@MainModelActorX
final class MainDataHandler {
    func newItem(date: Date) throws -> PersistentIdentifier {
        let item = Item(timestamp: date)
        modelContext.insert(item)
        try modelContext.save()
        return item.persistentModelID
    }

    func getTimestampFromItemID(_ itemID: PersistentIdentifier) -> Date? {
        return self[itemID, as: Item.self]?.timestamp
    }
}
```

#### Custom Initializer

```swift
@MainActor
@MainModelActorX(disableGenerateInit: true)
final class MainDataHandler1 {
    let date: Date

    func newItem() throws -> PersistentIdentifier {
        let item = Item(timestamp: date)
        modelContext.insert(item)
        try modelContext.save()
        return item.persistentModelID
    }

    func getTimestampFromItemID(_ itemID: PersistentIdentifier) -> Date? {
        return self[itemID, as: Item.self]?.timestamp
    }

    init(container: ModelContainer, date: Date) {
        self.date = date
        modelContainer = container
    }
}
```

## Testing Examples

### ModelActorXTests

```swift
struct ModelActorXTests {
    @Test func example1() async throws {
        let container = createContainer()
        let handler = DataHandler(modelContainer: container)
        let now = Date.now
        let id = try await handler.newItem(date: now)
        let date = await handler.getTimestampFromItemID(id)
        #expect(date == now)
    }

    @Test func example2() async throws {
        let container = createContainer()
        let now = Date.now
        let handler = DataHandler1(container: container, date: now)
        let id = try await handler.newItem()
        let date = await handler.getTimestampFromItemID(id)
        #expect(date == now)
    }
}
```

### MainModelActorXTests

```swift
@MainActor
struct MainModelActorXTests {
    @Test
    func test1() async throws {
        let container = createContainer()
        let handler = MainDataHandler(modelContainer: container)
        let now = Date.now
        let id = try handler.newItem(date: now)
        let date = handler.getTimestampFromItemID(id)
        #expect(date == now)
    }

    @Test
    func test2() async throws {
        let container = createContainer()
        let now = Date.now
        let handler = MainDataHandler1(container: container, date: now)
        let id = try handler.newItem()
        let date = handler.getTimestampFromItemID(id)
        #expect(date == now)
    }
}
```

## Parameters

### ModelActorX Macro

- `disableGenerateInit: Bool` (optional): Controls whether an initializer is automatically generated. Default is `false`.

### MainModelActorX Macro

- `disableGenerateInit: Bool` (optional): Controls whether an initializer is automatically generated. Default is `false`.

## Requirements

- Swift 6
- iOS 17.0 / macOS 14 / tvOS 17 / watchOS 10 or later