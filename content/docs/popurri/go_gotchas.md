---
title: "Go Gotchas"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# Introduction
Go is commonly mentioned as a language that is easy to jump into in part to it's reduced syntax, opinionated formatting and
small set of keywords. While it may or may not be the case it does have some particular gotchas which I personally feel
are worth pointing out as they might break with expectations coming from other languages, and in all cases I believe to
hurt the 'friendliness' of the language.

Some of these gotchas I have run into personally, and some I have read about. When the source is a publication, an
in-text citation is provided and the full citation may be found in the bibliography at the end of the text. 

# Shadowing
```go
package main

import (
    "fmt"
)

func main() {
 true := 10
 boolFlag := true

// You can also shadow package names, uncomment the next line to see
// fmt := "some value"

 fmt.Println(boolFlag)
}
```

# Slicing slices
```go
package main

import (
    "fmt"
)

func main() {
    nums := []int{1, 2, 3, 4}
    sharedIndex := 2
    startSlice := nums[:sharedIndex + 1] // Top of range is not inclusive
    endSlice := nums[sharedIndex:] // But lower range is inclusive

    fmt.Printf("Before Modification:\n\tNums: %v\n\tStartSlice: %v\n\tEndSlice: %v\n", nums, startSlice, endSlice)

    double(endSlice, 0) // &endSlice[0] == &nums[sharedIndex]

    fmt.Printf("After Modification:\n\tNums: %v\n\tStartSlice: %v\n\tEndSlice: %v\n", nums, startSlice, endSlice)

}

func double(ints []int, index int) {
    ints[index] = ints[index] * 2
}
```

# Closures
```go
package main

import (
    "fmt"
    "sync"
    //"time"
)

func main() {
    var wg sync.WaitGroup
    nums := []int{1, 2, 3}
    
    closure(nums, wg)
    wg.Wait()
}

func closure(nums []int, wg waitGroup) {
    for _, i := range nums {
        wg.Add(1)

        // Spawn routine to print index
        go func() {
            defer wg.Done()
            fmt.Printf("%d ", i)
        }()

        // Uncomment the following line to allow go routines time to capture i
        // before it increments
        //time.Sleep(time.Millisecond * time.Duration(3))
    }
}
```

# Returning Errors
```go
package main

import (
	"fmt"
)

type MyError string

func (e *MyError) Error() string {
	return string(*e)
}

func fallibleFunction() (bool, error) {
	var myError *MyError //nil is zero value for pointer
	return false, myError
}

func main() {
	if _, err := fallibleFunction(); err != nil {
		fmt.Println("Errored")
	} else {
		fmt.Println("No error")
	}
}
```

# Embedding
```go
package main

import (
    "fmt"
    "time"
)

type EmbeddedStruct struct {
    msg string
    time.Time
}


func main() {
    msgStruct := EmbeddedStruct{
        msg: "hello",
    }
    fmt.Printf("%v", msgStruct)
}
```

# Pointers in Structures
```go
package main

type PointerFieldStruct struct {
    ptr *int
}

func main() {
    invalidStruct := PointerFieldStruct{
        ptr: &5, // Compilation error, cannot take address of literals
    }

    x := 5
    validStruct2 := EmbeddPointer{
        prt: &x,
    }

    validStruct := PointerFieldStruct{
        ptr: valueToPointer(5),
    }

    
    _, _ = invalidStruct, validStruct
}

// Function takes any value and returns a pointer to the value. Intended for use
// with literal types (string, numbers, etc) to circumvent the limitation that
// address may not be taken for the aforementioned types.
func valueToPointer[T any](val T) *T {
    return &val
}
```

# Channels (writing/reading closed channels, ranging over channels that don't close, )

# Details not covered
Some details that aren't covered which might be of interest to the reader include:
- Floating point
- rune iteration of string (range for) vs byte iteration (indexed for)
- length of strings (runes being multibyte, length being byte based, "unicode/utf8")
- string comparisons (NFC vs NFD encoding, using "golang.org/x/text/unicode/norm")
- pointer channels (make(chan Type) vs make(chan *T))

# Bibliography
Bodner, Jon. _Learning Go: An Idiomatic Approach to Real-World Go Programming_. First ed., O'Reilly, 2022.  
Harsanyi, Teiva. _100 Go Mistakes and How to Avoid Them_. Manning Publications Co, 2022.  
Tebeka, Miki. Go _Brain Teasers_. Edited by Margaret Eldridge, P1.0 ed., Pragmatic Bookshelf, 2021.  