# Xcode Shortcuts & Tips

A collection of essential Xcode shortcuts and debugging tips to boost your productivity.

## Navigation & View

| Shortcut | Action |
| :--- | :--- |
| **`Cmd + Shift + O`** | **Open Quickly** - Fuzzy search for any file, class, or symbol. |
| **`Cmd + Shift + J`** | **Reveal in Project Navigator** - Shows the current file in the file tree. |
| **`Cmd + 0`** | Hide/Show **Navigator** panel (Left). |
| **`Cmd + Option + 0`** | Hide/Show **Inspector** panel (Right). |
| **`Cmd + Shift + Y`** | Toggle **Debug Area** (Bottom). |
| **`Cmd + Ctrl + F`** | Toggle **Fullscreen**. |
| **`Ctrl + 5`** | **File Dropdown** - Show related files/includes. |
| **`Ctrl + 6`** | **Method Dropdown** - Jump to a method in the current file. |
| **`Cmd + Ctrl + Left/Right`** | **Navigate Back/Forward** - Go to previous/next cursor position. |

## Editing & Refactoring

| Shortcut | Action |
| :--- | :--- |
| **`Cmd + Ctrl + E`** | **Edit All in Scope** - Highlights every instance of a variable in scope. Type to rename all at once. |
| **`Cmd + /`** | **Toggle Comment** - Comment/uncomment selected lines. |
| **`Ctrl + I`** | **Re-Indent** - Fix indentation for selected lines. |
| **`Cmd + [` / `Cmd + ]`** | **Indent / Outdent** lines. |
| **`Cmd + Option + Left/Right`** | **Fold / Unfold** code blocks. |
| **`Ctrl + Shift + Click`** | **Multi-Cursor Edit** - Edit multiple lines simultaneously. |
| **`Cmd + Shift + A`** | **Command Palette** (Xcode 15+) - Search for any Xcode command. |
| **`Cmd + Shift + L`** | **Library** - Insert code snippets, media, or colors. |

## Debugging (LLDB)

These commands are used in the Debug Console (bottom panel) when the app is paused at a breakpoint.

### `po` (Print Object)
Prints the description of an object.
```swift
(lldb) po user
// Prints: User(name: "John", email: "john@example.com")

(lldb) po self.view.subviews
// Shows all subviews in readable format
```

### `expr` (Expression)
Execute code on the fly to change state without recompiling.
```swift
(lldb) expr user.updateEmail("newemail@test.com")
(lldb) expr self.refreshUI()
(lldb) expr isLoginEnabled = true
```

### `v` (Frame Variable)
Shows all variables in the current scope (faster than `po`).
```swift
(lldb) v
// Shows: email = "test@example.com", password = "secret123", isValid = false
```

## Build & Run

| Shortcut | Action |
| :--- | :--- |
| **`Cmd + R`** | **Run** app. |
| **`Cmd + B`** | **Build** app. |
| **`Cmd + U`** | **Test** app. |
| **`Cmd + Shift + K`** | **Clean Build Folder**. |
| **`Ctrl + Cmd + R`** | **Run without Building** - Relaunches the last successful build immediately. |
