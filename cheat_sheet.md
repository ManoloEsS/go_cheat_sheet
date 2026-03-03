# Cheat sheet for GO

## Sorting
```go
// MODERN WAY (Go 1.21+) - Use this in interviews!
import "slices"

ints := []int{5, 2, 6, 3, 1}
slices.Sort(ints)                    // [1 2 3 5 6]
strings := []string{"c", "a", "b"}
slices.Sort(strings)                  // [a b c]

// Check if sorted
isSorted := slices.IsSorted(ints)     // true

// Sort in reverse
slices.Sort(ints)
slices.Reverse(ints)                  // [6 5 3 2 1]

// Custom sort (by length)
words := []string{"apple", "kiwi", "banana"}
slices.SortFunc(words, func(a, b string) int {
    return len(a) - len(b)            // negative if a < b
})
```

## String manipulation
```go
import "strings"

s := "hello world"
len(s)                                // 11 (bytes, not runes!)

strings.Contains(s, "world")          // true
strings.HasPrefix(s, "hello")         // true
strings.HasSuffix(s, "world")         // true
strings.Index(s, "world")             // 6
strings.ToUpper(s)                    // "HELLO WORLD"
strings.ToLower(s)                     // "hello world"
strings.TrimSpace("  hi  ")           // "hi"
strings.Split("a,b,c", ",")           // ["a", "b", "c"]
strings.Join([]string{"a", "b"}, "-") // "a-b"
strings.ReplaceAll("foo bar", "bar", "baz") // "foo baz"

import "strconv"

i, _ := strconv.Atoi("42")            // string to int (returns int, error)
s := strconv.Itoa(42)                  // int to string
f, _ := strconv.ParseFloat("3.14", 64) // string to float
```

## Conversions and type assertions
```go
// Basic conversions
i := int(3.14)                        // 3 (truncates)
f := float64(42)                       // 42.0
s := string(65)                         // "A" (rune conversion)

// Type assertion
var x interface{} = "hello"
s, ok := x.(string)                    // s = "hello", ok = true
if ok { fmt.Println(s) }

// Type switch
switch v := x.(type) {
case string: fmt.Println("string:", v)
case int: fmt.Println("int:", v)
default: fmt.Println("unknown")
}
```

## Slice operations
```go
// Creation
slice := make([]int, 5)                // length 5, capacity 5
slice := make([]int, 3, 5)             // length 3, capacity 5
slice := []int{1, 2, 3}                 // literal

// Append
slice = append(slice, 4)                // [1 2 3 4]
slice = append(slice, 5, 6)             // [1 2 3 4 5 6]
slice = append(slice, anotherSlice...) // concatenate

// Copy
dst := make([]int, len(src))
copy(dst, src)

// Subslice
sub := slice[1:3]                       // [2 3] (shares underlying array!)

// Delete element at index i
slice = append(slice[:i], slice[i+1:]...)

// Delete without preserving order
slice[i] = slice[len(slice)-1]
slice = slice[:len(slice)-1]

// Clear slice
slice = slice[:0]                        // length 0, capacity unchanged
```

## Map operations
```go
// Creation
m := make(map[string]int)
m := map[string]int{"a": 1, "b": 2}

// CRUD
m["c"] = 3                               // insert/update
val := m["a"]                             // 1
val, exists := m["z"]                     // 0, false
delete(m, "a")                            // delete

// Iteration (order NOT guaranteed!)
for k, v := range m {
    fmt.Println(k, v)
}

// Check if key exists idiom
if val, ok := m["key"]; ok {
    fmt.Println("Found:", val)
}
```

## Loops & range
```go
// Classic for loop
for i := 0; i < 10; i++ { ... }

// While-style
i := 0
for i < 10 { i++ }

// Infinite loop
for { break }

// Range over slice
nums := []int{2, 4, 6}
for i, v := range nums { ... }           // i = index, v = value
for _, v := range nums { ... }            // value only
for i := range nums { ... }               // index only

// Range over map
for k, v := range map[string]int{"a":1} { ... }
```

## Error handling
```go
// Basic pattern
f, err := os.Open("file.txt")
if err != nil {
    fmt.Println("Error:", err)
    return
}
defer f.Close()

// Custom errors
errors.New("something went wrong")
fmt.Errorf("error at %d", 42)

// Multiple returns with error (common pattern)
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
```

## Defer
```go
// LIFO order
defer fmt.Println("third")                // runs at function exit
defer fmt.Println("second")
defer fmt.Println("first")                 // prints: first second third

// Common use: closing resources
f, _ := os.Open("file.txt")
defer f.Close()                            // runs when function returns

// Defer with function literal
defer func() {
    if r := recover(); r != nil {
        fmt.Println("Recovered:", r)
    }
}()
```

## Channels and goroutines
```go
// Create channel
ch := make(chan int)
ch := make(chan int, 10)                  // buffered

// Send/receive
ch <- 42                                    // send
val := <-ch                                 // receive

// Goroutine
go func() { fmt.Println("async") }()

// Select (like switch for channels)
select {
case msg1 := <-ch1:
    fmt.Println(msg1)
case msg2 := <-ch2:
    fmt.Println(msg2)
case <-time.After(1 * time.Second):
    fmt.Println("timeout")
default:
    fmt.Println("no activity")
}
```

## Context
```go
import "context"

ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
defer cancel()

select {
case <-ctx.Done():
    fmt.Println("timeout:", ctx.Err())
case result := <-doWork():
    fmt.Println(result)
}
```

## Struct tags & reflection
```go
type User struct {
    Name string `json:"name" validate:"required"`
    Age  int    `json:"age" validate:"min=0"`
}

// Read tags with reflect
t := reflect.TypeOf(User{})
field, _ := t.FieldByName("Name")
jsonTag := field.Tag.Get("json")            // "name"
```

## Empty struct (memory optimization)
```go
// Zero memory! Great for sets
set := make(map[string]struct{})
set["key"] = struct{}{}
_, exists := set["key"]                      // true

// Channel signal
done := make(chan struct{})
go func() { close(done) }()
<-done
```

## IOTA (enums)
```go
type Status int
const (
    Pending Status = iota  // 0
    Active                  // 1
    Inactive                // 2
    Deleted                 // 3
)
```

## Sync package
```go
var mu sync.Mutex
var data map[string]int

func write(key string, val int) {
    mu.Lock()
    defer mu.Unlock()
    data[key] = val
}

// RWMutex for read-heavy
var rw sync.RWMutex
func read(key string) int {
    rw.RLock()
    defer rw.RUnlock()
    return data[key]
}
```

## Generics
```go
func Map[T any, U any](s []T, f func(T) U) []U {
    result := make([]U, len(s))
    for i, v := range s {
        result[i] = f(v)
    }
    return result
}

// Usage
nums := []int{1, 2, 3}
doubled := Map(nums, func(x int) int { return x * 2 })
```

## Strings Builder
```go
import "strings"

var b strings.Builder // the builder

b.WriteByte('a') //use for byte and not caring about UTF-8
b.WriteRune('b') //use for UTF-8
b.WriteString("hello") //use for full string, no matter length

return b.String()//returns the string abhello
```
