# 🐹 Go (Golang) & Concurrency Reference Cheatsheet

A concise, high-density reference guide for Go syntax, concurrency primitives, interfaces, generics, error handling, standard library patterns, and CLI tooling.

---

## 🚀 Basics & Control Flow

### Variables & Declarations
```go
var name string = "Go"
var age int = 15
count := 42                     // Short variable declaration (inside functions)
const MaxRetries = 5            // Untyped constant
const (
    StatusPending = iota        // 0
    StatusActive                // 1
    StatusFailed                // 2
)
```

### Control Flow
```go
// If with short initialization statement
if val, ok := mapData[key]; ok {
    fmt.Println("Found:", val)
}

// Switch statement (no break required)
switch role := user.Role; role {
case "admin", "owner":
    allowAccess()
case "guest":
    denyAccess()
default:
    readOnlyAccess()
}

// For loops (Go's single iteration keyword)
for i := 0; i < 10; i++ { ... }           // Standard loop
for key, val := range myMap { ... }       // Range over map
for idx, item := range slice { ... }      // Range over slice
for { break }                             // Infinite loop
```

---

## 📦 Structs, Interfaces & Generics

### Structs & Methods
```go
type User struct {
    ID    int    `json:"id"`
    Email string `json:"email"`
}

// Value receiver (does not mutate caller)
func (u User) String() string {
    return fmt.Sprintf("User(%d: %s)", u.ID, u.Email)
}

// Pointer receiver (mutates caller, avoids struct copying)
func (u *User) UpdateEmail(newEmail string) {
    u.Email = newEmail
}
```

### Interfaces & Type Assertions
```go
type Stringer interface {
    String() string
}

// Type Assertion
var i any = "hello"
str, ok := i.(string)

// Type Switch
switch v := i.(type) {
case int:
    fmt.Printf("Integer: %d
", v)
case string:
    fmt.Printf("String: %s
", v)
default:
    fmt.Printf("Unknown type
")
}
```

### Generics (Go 1.18+)
```go
type Number interface {
    ~int | ~int64 | ~float64
}

func Sum[T Number](numbers []T) T {
    var total T
    for _, num := range numbers {
        total += num
    }
    return total
}
```

---

## ⚡ Concurrency & Synchronization

### Channels & Goroutines
```go
// Unbuffered channel (synchronous block until read)
ch := make(chan int)

// Buffered channel (capacity 10)
bufCh := make(chan string, 10)

// Goroutine worker pattern
go func(msg string) {
    ch <- len(msg)
}("hello")

result := <-ch // Receive from channel

// Multiplexing with select
select {
case val := <-ch:
    fmt.Println("Received:", val)
case <-time.After(1 * time.Second):
    fmt.Println("Timed out")
}
```

### Sync Package (`WaitGroup` & `Mutex`)
```go
var (
    mu    sync.RWMutex
    cache = make(map[string]string)
    wg    sync.WaitGroup
)

// WaitGroup synchronization
wg.Add(1)
go func() {
    defer wg.Done()
    // execute background work
}()
wg.Wait()

// RWMutex usage
mu.RLock()
val := cache["key"]
mu.RUnlock()
```

### Context Package
```go
// Timeout context for HTTP requests or DB queries
ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
defer cancel()

req, _ := http.NewRequestWithContext(ctx, "GET", "https://api.example.com", nil)
```

---

## 🛡️ Error Handling & Defers

### Error Inspection & Wrapping
```go
var ErrNotFound = errors.New("resource not found")

func FetchItem(id int) (*Item, error) {
    if id <= 0 {
        return nil, fmt.Errorf("invalid item ID %d: %w", id, ErrNotFound)
    }
    return &Item{ID: id}, nil
}

// Error inspection with errors.Is & errors.As
if errors.Is(err, ErrNotFound) {
    // Handle not found
}

var pathErr *os.PathError
if errors.As(err, &pathErr) {
    fmt.Println("Failed path:", pathErr.Path)
}
```

### Defer & Panic Handling
```go
func ReadFile(filename string) error {
    f, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer f.Close() // Executed on function exit in LIFO order

    return nil
}
```

---

## 🛠️ CLI Tooling & Commands

| Command | Description |
|---|---|
| `go mod init <module>` | Initialize a new Go module |
| `go mod tidy` | Add missing dependencies and prune unused modules |
| `go build -o app main.go` | Compile binary for current platform |
| `GOOS=linux GOARCH=amd64 go build` | Cross-compile binary for Linux 64-bit |
| `go test -v ./...` | Run unit tests recursively in verbose mode |
| `go test -bench=. -benchmem` | Run benchmarks and output memory allocations |
| `go test -race ./...` | Run tests with dynamic data race detector enabled |
| `go vet ./...` | Run standard Go static code analysis |
| `golangci-lint run` | Run comprehensive fast linters runner |
