# Sle
[[العربية]](readme.ar.md)

Standard Libraries Extensions. This library holds a collection of extensions for standard libraries
of Alusus programming language.

---

## Importing the Library

This library is organized into separate modules. To use a module, import it using `Apm.importFile` with
the package name as the first argument and the modules' paths as the second argument:

```
import "Apm";
Apm.importFile("Alusus/Sle", { "<module1_path>", "<module2_path>", ... });
```

**Available modules:**

| Module                | Description                                   |
|-----------------------|-----------------------------------------------|
| `Srl/Console.alusus`  | Console helpers (stealth input)               |
| `Srl/System.alusus`   | System helpers (language detection, env vars) |
| `Srl/Time.alusus`     | Time/date formatting                          |
| `Srl/enums.alusus`    | Enum macros                                   |
| `Srl/Memory.alusus`   | Memory preallocation                          |
| `Srl/Net.alusus`      | HTTP requests                                 |
| `AstHelpers.alusus`   | AST generation helpers                        |

**Example:**

```
import "Srl/Console";
import "Srl/String";
import "Apm";
Apm.importFile("Alusus/Sle", { "Srl/Console.alusus", "Srl/Time.alusus" });
use Srl;

def password: String = Console.getStealthString();
def datetime: String = Time.getCurrentStringDateTime();
```

---

## Srl/misc - Miscellaneous Helpers

### charsPtrOrDefault

```
macro charsPtrOrDefault [val, default]
```

Returns the provided CharsPtr value, or the provided default if the value is 0.

---

## Srl/System - System Helpers

### getSystemLanguage

```
function getSystemLanguage(): String
```

Returns the language code of the system's language (e.g., "en", "ar").

### envVarOrDefault

```
macro envVarOrDefault [varName, defaultVal]
```

Fetch an environment variable or return a default string if the env var is not defined.

**Parameters**
* `varName` (CharsPtr) - name of the environment variable.
* `defaultVal` (CharsPtr) - fallback value if the env var is unset.

---

## Srl/Console - Console Helpers

### getStealthChar

```
function getStealthChar(): Int
```

Read a single character from the console without echoing it to the screen. This function temporarily disables terminal
echo and canonical mode, reads one character, then restores the original terminal settings.

**Returns**

An `Int` representing the character code read from the console.

**Platform Notes**
* On Windows, this uses the `_getch` function.
* On Unix-like systems, this uses termios to temporarily disable echo.

### getStealthString

```
function getStealthString(): String
```

Read a string from the console securely without displaying the input. This function reads characters one by one using
`getStealthChar()` and handles special keys:
* **Enter** (newline or carriage return): Completes input and returns the string.
* **Backspace/Delete**: Removes the last character, with proper UTF-8 multi-byte character support.

**Returns**

A `String` containing the entered text.

**Example**

See [Examples/get_password.alusus](Examples/get_password.alusus) for a complete example.

---

## Srl/Time - Time/Date Helpers

### getCurrentStringDateTime

```
function getCurrentStringDateTime(): String
```

Return the current system date and time as a formatted string.

**Returns**

A `String` in the format: `YYYY-MM-DD HH:MM:SS`

### getStringDateTime

```
function getStringDateTime(timestamp: ArchInt): String
```

Return the date and time for the given timestamp as a formatted string.

**Parameters**
* `timestamp` (ArchInt) - Unix timestamp.

**Returns**

A `String` in the format: `YYYY-MM-DD HH:MM:SS`

---

## Srl/enums - Enum Helpers

Macros for creating integer-based and string-based enumerations.

### setupIntEnum

```
macro setupIntEnum
```

Sets up an integer-based enum inside a class. This macro adds a `val` field and comparison/assignment handlers.

### enumIntValue

```
macro enumIntValue [name, v]
```

Defines an enum value with the given name and integer value.

**Parameters**
* `name` - The name of the enum value.
* `v` - The integer value.

### setupStringEnum

```
macro setupStringEnum
```

Sets up a string-based enum inside a class. This macro adds a `val` field (of type String) and the necessary
initialization, comparison, and assignment handlers.

### enumStringValue

```
macro enumStringValue [name, val]
```

Defines an enum value with the given name and string value.

**Parameters**
* `name` - The name of the enum value.
* `val` - The string value.

**Example**

```
class MyIntEnum {
    setupIntEnum[];
    enumIntValue[VAL1, 1];
    enumIntValue[VAL2, 2];
}

class MyStringEnum {
    setupStringEnum[];
    enumStringValue[VAL1, "value1"];
    enumStringValue[VAL2, "value2"];
}
```

See [Examples/enum_example.alusus](Examples/enum_example.alusus) for a complete example.

---

## Srl/Memory - Memory Preallocation Helpers

Provides memory preallocation functionality for performance optimization in scenarios where many small allocations
are made. Instead of calling the system allocator for each allocation, memory is preallocated in blocks.

### initializePreallocation

```
func initializePreallocation
```

Initialize the preallocation system. Must be called before using preallocation features.

### cleanupPreallocation

```
func cleanupPreallocation
```

Cleanup and reset the preallocation system.

### startPreallocation

```
func startPreallocation(size: ArchInt, enlargementSize: ArchInt): ptr
```

Start a preallocation block with the given initial size and enlargement size.

**Parameters**
* `size` (ArchInt) - Initial block size in bytes.
* `enlargementSize` (ArchInt) - Size for subsequent blocks if more memory is needed.

**Returns**

A pointer to the preallocation block (used for `endPreallocation`).

### endPreallocation

```
func endPreallocation(block: ptr)
func endPreallocation(block: ptr, printLog: Bool)
```

End a preallocation block and free all memory allocated within it.

**Parameters**
* `block` (ptr) - The block pointer returned by `startPreallocation`.
* `printLog` (Bool) - If true, prints statistics about allocations.

### runWithPreallocation

```
func runWithPreallocation(size: ArchInt, enlargementSize: ArchInt, printLog: Bool, toRun: closure())
```

Convenience function that runs a closure with preallocation enabled, automatically handling setup and cleanup.

**Parameters**
* `size` (ArchInt) - Initial block size.
* `enlargementSize` (ArchInt) - Enlargement size for additional blocks.
* `printLog` (Bool) - Whether to print allocation statistics.
* `toRun` (closure()) - The closure to execute with preallocation.

**Example**

See [Examples/memory_example.alusus](Examples/memory_example.alusus) for a complete example.

---

## Srl/Net - Network Helpers

Provides HTTP request functionality using libcurl.

### Request Class

```
class Request
```

A class for making HTTP requests.

**Constructors**

```
handler this~init(url: String)
handler this~init(url: String, contentType: String)
handler this~init(url: String, authKey: String, authType: String)
handler this~init(url: String, authKey: String, authType: String, contentType: String)
```

**Properties**
* `url: String` - The request URL.
* `authKey: String` - Authentication key/token.
* `authType: String` - Authentication type (e.g., "Bearer").
* `contentType: String` - Content type (default: "application/json").
* `responseBody: String` - Response body after request completes.
* `responseHttpStatus: Int` - HTTP status code of the response.
* `verbose: Bool` - Enable verbose output (default: true).

**Methods**

#### get

```
handler this.get(): Bool
```

Perform an HTTP GET request.

#### post

```
handler this.post(postedData: ptr[array[Char]]): Bool
handler this.post(postedData: ptr[array[Char]], outputFilename: ptr[array[Char]]): Bool
```

Perform an HTTP POST request.

#### put

```
handler this.put(data: ptr, dataSize: ArchInt): Bool
```

Perform an HTTP PUT request with raw data.

#### putFile

```
handler this.putFile(filename: String): Bool
```

Perform an HTTP PUT request with file contents.

#### delete

```
handler this.delete(): Bool
```

Perform an HTTP DELETE request.

#### addHeader

```
handler this.addHeader(header: String)
```

Add a custom HTTP header to the request.

#### sendEmail

```
handler this.sendEmail(
    receivers: Array[String],
    sender: String,
    subject: String,
    body: String,
    bodyType: String,
    password: String
): Bool
```

Send an email via SMTP.

**Example**

See [Examples/net_example.alusus](Examples/net_example.alusus) for a complete example.

---

## AstHelpers - AST Generation Helpers

These macros insert AST nodes at preprocessing time via `Spp.astMgr`.

### strLiteralFromVal

```
macro strLiteralFromVal [val]
macro strLiteralFromVal [val, default]
```

Insert a string-literal AST from a raw `CharsPtr`. The second form uses `default` if `val` is null.

### strLiteralFromEnvVar

```
macro strLiteralFromEnvVar [varName]
macro strLiteralFromEnvVar [varName, default]
```

Create a string literal from an environment variable. The second form provides a fallback value.

### intLiteralFromInt

```
macro intLiteralFromInt [val]
```

Insert an integer-literal AST from a numeric value.

### intLiteralFromCharsPtr

```
macro intLiteralFromCharsPtr [val]
macro intLiteralFromCharsPtr [val, default]
```

Insert an integer-literal AST from a string. The string must contain a number.

### intLiteralFromEnvVar

```
macro intLiteralFromEnvVar [varName]
macro intLiteralFromEnvVar [varName, default]
```

Create an integer-literal AST from an environment variable string.

**Example**

See [Examples/ast_helpers.alusus](Examples/ast_helpers.alusus) for a complete example.
