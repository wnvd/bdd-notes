Constants must be known at compile time. They are usually declared with 
static value:
```go

const myInt = 33

```

This is valid
```go

const firstName = "naved"
const lastName = "wani"

fmt.println(firstName + " " + lastName) 

```

However, this is not valid

```go

const currentTime = time.Now()

```

- Clean Interfaces:
Writing clean interfaces is *hard*.
Simple can become complex very quickly

1
Keep Interfaces Small
Interfaces are meant to define the minimal behaviour 
necessary to accurately represent an idea or concept.

Here is the example of the standard HTTP package of a large 
interface that's good example of defining minimal behavior:

```go

type File interface {
    io.Closer
    io.Reader
    io.Seeker
    Readdir(count int) ([]os.FileInfo, error)
    Stat() (os.FileInfo, error)
}

```

Any type that satisfies the interface's behaviour can be considered by the HTTP package as a *File*. This is convenient because
the HTTP package doesn't need to know if it's dealing with a file on disk, a network buffer or a simple `[]byte`

2
Interfaces Should Have No Knowledge of satisfying Types:

An interface should define what is necessary for other types to classify as a member of that interface. They shouldn’t be aware of any types that happen to satisfy the interface at design time.

```go

type car interface {
	Color() string
	Speed() int
	IsFiretruck() bool
}

```

Color() and Speed() make perfect sense, they are methods confined to the scope of a car. IsFiretruck() is an anti-pattern. We are forcing all cars to declare whether or not they are firetrucks. In order for this pattern to make any amount of sense, we would need a whole list of possible subtypes. IsPickup(), IsSedan(), IsTank()… where does it end??

Instead, the developer should have relied on the native functionality of type assertion to derive the underlying type when given an instance of the car interface. Or, if a sub-interface is needed, it can be defined as:

```go

type firetruck interface {
	car
	HoseLength() int
}

```
the firetruck here inherits methods from car and adds methods required to make the car to firetruck

3
Interfaces Are Not Classes
- Interfaces are not classes, they are slimmer.

- Interfaces don’t have constructors or deconstructors that require that data is created or destroyed.

- Interfaces aren’t hierarchical by nature, though there is syntactic sugar to create interfaces that happen to be supersets of other interfaces.

- Interfaces define function signatures, but not underlying behavior. Making an interface often won’t DRY up your code in regards to struct methods. For example, if five types satisfy the fmt.Stringer interface, they all need their own version of the String() function.


# Errors:

Go programs express errors with values. An Error is any type
that implements that simple built-in error interface.

```go

type error interface {
	Error() string
}

```

When something can go wrong in a function that function should 
return an error as its last return value. Any code that calls a
function that can return a error should handle errors by testing
whether the error is nil

```go

// Atoi converts a stringified number to an integer
i, err := strconv.Atoi("42b")
if err != nil {
    fmt.Println("couldn't convert:", err)
    // because "42b" isn't a valid integer, we print:
    // couldn't convert: strconv.Atoi: parsing "42b": invalid syntax
    // Note:
    // 'parsing "42b": invalid syntax' is returned by the .Error() method
    return
}
// if we get here, then the
// variable i was converted successfully

```
 A nil error detects success; a non-nil error denotes failure.

When you return a non-nil error in Go, it's conventional to return the "zero" values of all other return values.


# Panic:
We a function calls panic the entire program crashes and prints a stack tree.

As a rule of thumb do not use panic.

The `panic` function yeets controls out of the current function and up the 
call stack until it reaches a function that defers a recover. If no function
calls `recover` the `goroutine` (often the entire program) crashes.

```go

func enrichUser(userID string) User {
	user, err := getUser(userID)
	if err != nil {
		panic(err)
	}
	return user
}

func main() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("recovered from panic: ", r)
		}
	}()

	/*
		 this panics, but the defer/recover block catches it
		 a truly astonishingly bad way to handle errors.
	*/

	enrichUser("123")
}


```

If I have a truly unrecoverable error, I use `log.Fatal` to print a
message and exit the program.

# Slices
Slices hold references to an underlying array, and if you assign one slice
to another they both refer to *same* array.
If a function takes a slice argument any changes it makes to the elements
of the slice will be visible to the caller, same as passing a pointer
to the underlying array

If you look the `Read` function below it takes a slice of bytes rather
then a pointer and a count; the length within the slice sets an upper
limit of how much data to read.

```go

func  (f *File) Read(buf []bytes) (n int, err error)

```

So here is something to be aware when using a slice.

When slices are only declared they default to `nil` i.e
`var mySlice []int` this is declaration if no data is added it will
be `nil`

Here slice is declared and initialized of it defaults to its 
corresponding zero value

` mySlice := []int{}` this is empty slice


When you make a slice like this `mySlice := [][]int{}` you make an 
empty slice and when you will try to access it like this `mySlice[i][j]`
you will get a runtime error because you are trying to access something
that does not exist. 

To fix this you need to use `make()` that will allocate space for 
slice or you need to use `append()` to add elements to the slice.

If their is an array you can make a slice of that array like below:

```go
var arr = [3]int{1, 2, 3}

sliceOfArray := arr[:]
```


# Range
Go provides syntactic sugar iterate easily over elements of a slice.

# append()
This is something that you need to be careful with, append() function
changed the underlying array so it is always a bad idea on using append()
on anything other then itself

```go

// don't do this!
someSlice := append(slice, element)

```

- example 1:
```go

a := make([]int, 3)
fmt.Println("len of a", len(a))
fmt.Println("cap of a", cap(a))
/*
len of a = 3
cap of a = 3
*/

b := append(a, 4)
fmt.Println("appending 4 to b from a")
fmt.Println("b: ", b)
fmt.Println("addr of b", &b[0])
/*

appending 4 to b from a
b: [0,0,0,4]
addr of b: 0x44a0c0

*/

c := append(a, 5)
fmt.Println("appending 5 to c from a")
fmt.Println("addr of c:", &c[0])
fmt.Println("a:", a)
fmt.Println("b:", b)
fmt.Println("c:", c)
// appending 5 to c from a
// addr of c: 0x44a180
// a: [0 0 0]
// b: [0 0 0 4]
// c: [0 0 0 5]

```

- example 2
This example is important; 
the append() function only creates a new array when there isn't any 
capacity left (remember len is the number of element in the slice not 
the actual size), We create i with a length of 3 and a capacity of 8
, which means we can append 5 items before a new array is automatically
created.

```go

i := make([]int, 3, 8)
fmt.Println("len of i:", len(i))
fmt.Println("cap of i:", cap(i))
// len of i: 3
// cap of i: 8

j := append(i, 4)
fmt.Println("appending 4 to j from i")
fmt.Println("j:", j)
fmt.Println("addr of j:", &j[0])
// appending 4 to j from i
// j: [0 0 0 4]
// addr of j: 0x454000

g := append(i, 5)
fmt.Println("appending 5 to g from i")
fmt.Println("addr of g:", &g[0])
fmt.Println("i:", i)
fmt.Println("j:", j)
fmt.Println("g:", g)
// appending 5 to g from i
// addr of g: 0x454000
// i: [0 0 0]
// j: [0 0 0 5]
// g: [0 0 0 5]


```
again, TO AVOID BUGS LIKE THIS, YOU SHOULD ALWAYS USE THE `append()` 
function on the same slice the result is assigned to

```go

mySlice := []int{1,2,3} // len 3, cap 3
mySlice := append(mySlice, 4)

```

# Maps

NOTE:
like slices, maps are also passed by reference into functions. This 
means that when a map is passed into a function we write, we can make
changes to the original - we don't get a copy.

Any thing can be used as value in a map *but* keys are more restrictive.

Maps keys my be used of any type that is comparable.
It is obvious that strings, ints and other basic types should be 
available as map keys but perhaps unexpected are *struct keys*.

Structs can be used to key data by multiple dimensions,for example
this maps of maps can be used to tally web page hits by country

```go

hits := make(map[string]map[sting]int)

```
this above is a map of string to (map of string to int). Each key
of the outer map is the path to a web page its own inner map.
Each inner map key is a two-letter country code. This expression retrieves the number of times an Australian has loaded the documentation page

```go

n := hits["/doc/"]["au"]

```

Unfortunately, this approach becomes unwieldy when adding data, as for
any given outer key you must check if the inner map exists and create
it if needed.

```go

func add(m map[string]map[string]int, path, country string) {
    mm, ok := m[path]
    if !ok {
        mm = make(map[string]int)
        m[path] = mm
    }
    mm[country]++
}
add(hits, "/doc/", "au")

```

On the other hand, a design that uses a single map with  a struct key
does away with all that complexity:

```go
type Key struct {
	path, country string
}

hits := make(map[Key]int)


hits[Key{"/", "vn"}]++

```

This approach is much consistent, and can also be used as follows:

```go
n := hits[Key{"/ref/spec", "ch"}]
```

To below pattern is quite common with check with if statement
```go

names := map[string]int{}
missingNames := []string{}

// this 'ok' check in if block is 
// a common pattern
if _, ok := names["Denna"]; !ok {
	// if the key does not exist yet
	// append the name to the missingNames
	missingNames = append(missingNames, "Denna")
}

```

If you access a missing key in a map where the value type is a struct, Go returns the zero value for that struct—so all fields are at their zero values, not nil.

Only pointer types, slices, maps, functions, interfaces, and channels can have nil as a zero value. Structs themselves do not have a nil value unless you're working with pointers to structs.

# Pointers
A pointer is a variable that stores location of another variable

```go

var int a = 26
var p *int // pointer to an integer

p := &a // you can also define a pointer like this.

```

## References
It's possible to define empty pointer.
```go

var p *int // this is an empty pointer

fmt.Printf("value of the p: %v\n", p)

```

Its zero value is nil, which means it does not  point to any memory address.
Empty Pointer are also called "nil pointer"

Instead of starting with a nil pointer, its' common to use `&` operator to get pointer
to its operand:

```go

myString := "hello"
myStringPtr := &myString

fmt.Printf("value of myStringPtr: %v\n", myStringPtr); // this will be some hex value

```

## Deference
The `*` operator dereferences a pointer to get the original value.

```go

*myStringer = "world" // setting myStringer value through value

fmt.Printf("value of myStringer: %s\n", *myStringr) // read myStringer through the pointer value

````

Unlike c, Go has not pointer arithmetic.

Some to pay attention to:

```go
func replaceMessageWord(message *string, oldWord string, newWord string) {
	if strings.Contains(*message, oldWord) {
		/* here you are taking the value that *message pointer points to
		 * and than you are assigning the result again to the memory address
		 * that *message points to
		 * 
		 *
		 * only doing strings.ReplaceAll(*message, oldWord, newWorld) will not 
		 * work because you are not doing anything with the result value.
		 *
		*/
		*message = strings.ReplaceAll(*message, oldWord, newWord)
	}
}


```

# Fields of Pointers
When functions receives a pointer to a struct, you might try to access
a filed like this and encounter an error:
```go

msgTotal := *analytics.MessagesTotal //errr

```
Instead, access it like you would normally do - using selector expression
```go

msgTotal := *analytics.MessageTotal
/* 
* this happens because you end dereference a non pointer because
* '.' has higher precedence than '*'. We look up 'MessageTotal' value 
* first then end up dereferencing it (which is a non pointer)
*/

```

This approach is the recommended, simplest way to access struct field in
Go. and is shorthand for:

```go

(*analytics).MessageTotal

```

## Pointer Receivers
A receiver type on a method can be a pointer

Methods with pointer receivers can modify the value to which the receiver
points. Since methods often need to modify their receiver, pointer receivers are more common than value receivers. However, methods with pointer 
receivers don't require that a pointer is used to call the method. The
pointer will automatically be derived the value.


Methods with pointer receivers don't require that a pointer is used to
call the method. The pointer will automatically be derived from the value.

```go
type circle struct {
	x int
	y int
	radius int
}

func (c *circle) grow() {
	c.radius * = 2
}

func main () {
	c := circle {
		x: 1,
		y: 2,
		radius: 4,
	}

	// notice c is not a pointer in the calling function
	// but the method still gains access to a pointer to c
	c.grow()
	fmt.Println(c.radius) // prints 8
}

```

# Pointer Performance
Before even thinking about using pointers to optimize your code, use pointers when you need a shared reference to a value; otherwise, just use values.

Interestingly, local non-pointer variables are generally faster to pass around than pointers because they're stored on the stack, which is faster to access than the heap. Even though copying is involved, the stack is so fast that it's no big deal.

Once the value becomes large enough that copying is the greater problem, it can be worth using a pointer to avoid copying. That value will probably go to the heap, so the gain from avoiding copying needs to be greater than the loss from moving to the heap.

One of the reasons Go programs tend to use less memory than Java and C# programs is that Go tends to allocate more on the stack.

Typically values are more performant to pass to function instead of pointer (especially for small amount of data).


# Packages
A package named "main" has an entry point at the `main()` function. A `main`
package is compiled into an executable program.

A package by any other name is a "library package". Libraries have no entry
point. Libraries simple export functionality that can be used by other 
packages.

## package naming
By *convention*, a package's name is the same as the last element of its
import path. For Instance, the `math/rand` package comprises files that 
begin with the path `package rand`

That said, package names aren't required to match their import path.
For example I could write a new package with the path `github.com/wagslane/rand` and name the package `random`. `package random`
While this is possible But is *discouraged*.

## One Package / Directory
A directory of Go code can have at *most* one package. All `.go` files
in a single directory must all belong to the same package. If they don't
an error will be thrown by the compiler. This is true for main and library
packages alike.


## Modules
In go packages are directories/folders. Functions types, variables and
constants defined in one source files are visible to *all other source
files within the same package (directory)*

A repository contains one or more modules. A module is a collection
of Go packages that are released together.

#### One module per Repo (usually)
A file named go.mod at the root of the project declares the module.
It contains

- the module path
- the version of the go language your project has
- Optionally, any external package dependencies you project has

Each module's path not only *serves as an import path prefix* for the 
packages within but also indicates where the *go command* should look
to download it. For example, to download the module `golang.org/x/tools`,
the go command  would consult the repository located at `https://golang.org/x/tools`

```go

An "import path" is a string used to import a package. A package's import path is its module path joined with its subdirectory within the module. For example, the module github.com/google/go-cmp contains a package in the directory cmp/. That package's import path is github.com/google/go-cmp/cmp. Packages in the standard library do not have a module path prefix.

```

A module can contain many packages, but it begins wherever the `go.mod`
file is found. Everything inside that directory tree(unless it hits another `go.mod`) belongs to that module.

### Go Environment
#### Directory Structure

To recap how packages and modules work in your project directory structure:

You will have many git repositories on your machine (typically one per project).
- Each repository is typically a single module.
- Each repository contains one or more packages
- Each package consists of one or more Go source files in a single directory.
- The path to a package's directory determines its import path and where it can 
be downloaded from if you decide to host it on a remote version control system like GitHub or GitLab.


Below are the step to initialize a project in go.

```go

mkdir <project-name>
cd <project-name>

```

Inside the folder run following command

```bash

# notice <project-name> and <folder-name> above
# should be the same
go mod init github.com/<username>/<project-name>


```


--------------------------

## Custom Package 
Let's write a *custom package* (does not have main() function) 'bar'  and use it in our *main package* 'foo' (has main() function).

- Main Package (foo)
Create a `foo` directory and `cd` into it

```bash
mkdir foo
cd foo
```

After creating foo directory initialize go project.
```bash

go mod init github.com/wnvd/foo

```

Create `main.go` file in `foo`

```bash

touch main.go

```

And write the following code

```go
package main // has this main package

import "fmt"

func main() {
	fmt.Println("foo")
}

```

- Custom Package (bar)
Create a sibling directory (bar) in the parent directory of the (foo)
Your file structure should look like this.
```bash
(Parent directory)
|
|- foo/ # main package
|  |- main.go
|  |- go.mod
|- bar/ # custom package
   |- bar.go
   |- go.mod
```

After creating 'bar/' directory initialize module
```go

go mod init github.com/wnvd/bar

```
Then create a `bar.go` file in `bar/` directory

```go
package bar // as this 'bar' package now

// notice capital 'R' in the function name this 
// means function is accessible outside its out package 
// (in this case to foo)
func Reverse(s string) string {
	result := ""
	for _, val := range s {
		result = string(val) + result
	}
	return result
}
```

Now your main package `foo` should look like this.
```go
package main

import (
	"fmt"
	"github.com/wnvd/bar" // importing bar
)

func main() {
	fmt.Println(strings.Reverse("hello world"))
}


```

After this you have to update `go.mod` file to import the local
version of the `mystrings` dependency:

 Pay attention to 'replace' keyword is not advised, but can be useful
 to get up and running quickly. The proper way to create and depend on
 modules is to publish them to a remote repository. When you do that,
 the "replace" keyword can be dropped from the go.mod:

This only works with local development:
`replace github.com/wagslane/mystrings v0.0.0 => ../mystrings`

If we want the import to work for everyone, we need to make sure the 
dependency (bar in our case) actually exits on
`https://github.com/wnvd/bar`


`go.mod`
```go
module github.com/wnvd/foo

go 1.24.0

replace github.com/wnvd/bar v0.0.0 => ../bar

require (
	github.com/wnvd/bar v0.0.0
)

```

----------------------------------

Usually you would use: 
`go get <url-path>`

It will add the dependency to your go.mod file
and then you can import it using:

`import <alias-name> "<url-path>"`


## Clean Packages
*Learning to properly build small and reusable packages can take your
Go career to the next level*

### Rule of Thumbs
1 - Hide Internal Logic
If you're familiar with the pillars of OOP, this is a practice in 
*encapsulation*.

Oftentimes, an applications will have complex logic that requires a
lot of code. In almost every case the logic that the app cares about
can be exposed via an API, and most dirty work can be kept within
a package. For example, imagine we are building an application that 
needs to classify images. We could build a package:

```go
package classifier

// ClassifyImage classifies image as "hotdog" or "not hotdog"
func ClassifyImage(image []byte) (imageTypeString) {
	if hashotDogColors(image) && hasHotDogShape(image) {
		return "hotdog"
	} else {
		return "not hotdog"
	}
}

func hasHotDogShape(image []byte) bool {
	return true
}

func hasHotDogColors(image []byte) bool {
	return true
}
```
We create an API by only exposing the function(s) that the application-level needs to know about. All other logic is unexported to keep a clean separation of concerns. The application doesn’t need to know how to classify an image, just the result of the classification. Remember, in Go, functions with names starting with a lowercase letter are unexported and private to the package, while functions starting with an uppercase letter are exported and can be accessed externally.

2 - Don't Change APIs
The unexported functions within a package can and should change often 
for testing, refactoring and bug fixing.

A well-designed library will have a stable API so that users don't get breaking changes each time they update the package version. In Go, this means *not changing exported function’s signatures.*

3 - Don't Export Functions from the Main Package
A `main` package isn't a library, there's no need to export functions
from it.

4 - Packages should not know about dependents
Perhaps one of the most important and most broken rules is that a package
shouldn't know anything about its dependents.
In others words, a package should never have a specific knowledge about
a particular application that uses it.
(In short package needs to be self contained)


```go

// BAD design - package knows about a specific application
package database

func ConnectToUserDatabase() {
    // This function assumes it's being used in a user management application
    connectToDatabase("user_data")
}

func GetActiveUsers() []User {
    // This function is written specifically for a user dashboard feature
    // in a particular application
}

```

```go

// GOOD design - generic package with no knowledge of specific applications
package database

func Connect(databaseName string) Connection {
    // Generic function that works for any application
    return connectToDatabase(databaseName)
}

func Query(conn Connection, query string) Results {
    // Generic querying that any application can use
}

```

# Concurrency

- What is concurrency?
Concurrency is the ability to perform multiple task at the same time.
Typically our code is executed one line at a time, one after the other.
This is called *sequential execution* or *synchronous execution*.

```txt

Synchronous Exection
------------->------------->
   task 1         task 2




Asynchrnous Execution
---------------------->
	task 1
---------------------->
	task 2

```


If the computers we're running our code on has multiple cores, we can
even execute multiple tasks at exactly the same time. If we're running
on a single core, a single core executes code at *almost* the same time
by switching between tasks very quickly. Either way, the code we write
looks the same in go and takes advantage of whatever resources are 
available.

- How does concurrency work in Go?
Go designed to be concurrent.

It excels at performing many tasks simultaneously safely using 
simple syntax.

```go

go dosomething()

```
example above, dosomething() will be executed concurrently with the rest
of the code in the function. The `go` keyword is used to spawn a new
*goroutine*.



# Channels
Channels are typed, *thread-safe* queue. Channels allow different
*goroutines* to communicate with each other.

## Create a Channel
Like map and slices, channels must be created *before* use. They also
use the same `make` keyword

```go

ch := make(chan int)

```

## Send data to a channel

```go

ch <- 69

```

The `<-` operator is called the *channel operator*. Data flows in the 
direction of the arrow. This operation will *block* until another 
*goroutine* is ready to receive the value.

## Receive data from a channel

```go
v := <- ch
```
This reads and removes a value from the channel and saves it into the 
variable `v`. This operation will *block* until there is a value in
the channel to be used.

````go

package main

import (
	"time"
)

type email struct {
	body string
	date time.Time
}

func checkEmailAge(emails [3]email) [3]bool {
	isOldChan := make(chan bool)
	
	/**
	 * if we remove go keyword from the below line
	 * the program will deadlock. That is because when sending 
	 * value to 'isOld[<index>]' the `<- isOldChan` does not 
	 * have the value to send so it deadlocks.
	 *
	*/

	go sendIsOld(isOldChan, emails)

	isOld := [3]bool{}
	isOld[0] = <-isOldChan
	isOld[1] = <-isOldChan
	isOld[2] = <-isOldChan
	return isOld
}

// don't touch below this line

func sendIsOld(isOldChan chan<- bool, emails [3]email) {
	for _, e := range emails {
		if e.date.Before(time.Date(2020, 0, 0, 0, 0, 0, 0, time.UTC)) {
			isOldChan <- true
			continue
		}
		isOldChan <- false
	}
}

````

Useful channel types:
- `chan<-` means "send-only" (you can put values into the channel)
- `<-chan` would mean "receive-only" (you can only take value from the channel)
- `chan` without an arrow mean bidirectional (you can both send and receive)


Look at below code
```go

func waitForDBs(numDBs int, chanDB chan struct{}) {

	/*
	 * if all the data in not send from the channel 
	 * it will block. Here the channel only handles (send then negates)
	 *  first data that is received in the channel then *blocks* for the rest
	*/
	<-chanDB 


	/*
	 * you need send out all the data present inside the channel
	 * in this case you can use the for loop
	*/
	for numDBs > 0 {

		<- chanDB // each data that is being sends out to channel is handled here
		numDBs--
	}


}

```

## Buffered Channels 
Channels can optionally be buffered

You can provide a buffer length as the second argument to make() to 
create buffered channel
```go
	ch := make(chan int, 100) 
```
A buffer allows the channel to hold a fixed number of values before
sending blocks. This mean sending on a buffered channel only blocks
when the buffer is full, and receiving blocks only when the buffer is
empty. 

*In short*
If buffer is full you cannot send to it.
If buffer is empty you cannot receive from it

## Closing Channels

Channels can be explicitly  closed by senders.

```go
ch := make(chan int)

// do some stuff with the channel 

close(ch)
```
### Checking if a channel is closed
Similar to the `ok` value when accessing data in a map receivers can check 
the `ok` value when receiving from a channel to test if a channel was closed

```go

v, ok := <-ch

```
ok is false, if channel is empty and closed.


### Don't send on a closed channel
Sending on a close channel will cause a *panic*. A panic on the main goroutine
will cause the entire program to crash, and a panic in any other goroutine
will cause that goroutine to crash.

Closing is not necessary. There's nothing wrong with leaving channels open,
they'll still be garbage collected if they're unused. You should close
channels to indicate explicitly to a receiver that nothing else is going
to come across.


### Range
Channels can be ranged over 

```go

for item := range ch {
	// item is the next value received from the channel
}

```
The above example will receive values over the channel (blocking at each
iteration if nothing new is there) and will exit only when channel is closed.

Look at the example program below

```go

package main

func concurrentFib(n int) []int {
	ch := make(chan int)	
	go fibonacci(n, ch)

	/*
	* you can use len() on channel to see how many elements are
	* there in the channel
	*/
	fib := make([]int, len(ch))
	for  num := range ch {
		// here num is the next value in the channel
		fib = append(fib, num)
	}
	return fib
}

// don't touch below this line

func fibonacci(n int, ch chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		ch <- x
		x, y = y, x+y
	}
	close(ch)
}

```

# Select
Sometimes we have a single goroutine listening to multiple channels and
want to process data in the order it comes through each channel

A `select` statement is used to listen to multiple channels at the same
time. It is similar to a `switch` statement but for channels

```go

select {
	case i, ok := <- chanInts:
		fmt.Println(i)
	case s, ok := <- chanStrings:
		fmt.Println(s)
}

```

The first channel with a value ready to be received will fire and its body will execute.
If multiple channels are ready at the same time one is chosen randomly. The ok variable in
the example above refers to whether or not the channel has been closed by the sender yet.

```go

func logMessages(chEmails, chSms chan string) {
	for {
		select {
			case email, ok := <-chEmails:
				if !ok {
					return
				}
				logEmail(email)
			case sms, ok := <-chSms:
				if !ok {
					return
				}
				logSms(sms)
		}
	}
}

```

### Select default case
The `default` case in a `select` statement executes *immediately* if no
other channel has a value ready. A `default` case stops `select` statement
from blocking

```go

select {
	case v := <- ch:
		// use
	default:
		// receiving from ch would block
		// so do something else
}

```

### tickers
- `time.Tick()` is standard lib function that returns a channel
that sends a value on a given interval.
- `time.Atleast()` sends a value once after the duration has passed.
- `time.Sleep()` blocks the current goroutine for the specific duration
of time.

These functions take a `time.Duration` as an argument
```go

// if millisecond is not added it will default to nanoseconds.
time.Tick(500 * time.Millisecond) 

```

### Read Only Channels
The channel can be marked as read-only by casting it from
a `chan` to a `<-chan` type.
```go
func main() {
	ch := make(chan int)
	readCh(ch)
}

func readCh(ch <-chan int) {
	// ch can only be read from
	// in this function
}

```

### Write Only Channels
The channel can be marked as write-only by casting it from
a `chan` to a `chan<-` type.
```go
func main() {
	ch :make(chan int)
	writeCh(ch)
}

func writeCh(ch chan<- int) {
	// ch can only be written to
	// in this function
}

```

# Channels Review
###  A declared about Uninitialized Channel is Nil just like a slice

```go
var s []int // s is nil
var c chan string // c is nil


var s = make([]int, 5) //  s is initialized and not nil
var c = make(string, 5) // c is initialized an not nil

```

### A Send to a Nil Channel Block Forever :)

```go
var c chan string // c is nil

c <- "Let's ge started" // blocks

```

### A Receive from a Nil Channel Blocks Forever

```go

var c chan string // c is nil
fmt.Println(c) // blocks

```

### A Send to Closed Channel Panics

```go

var c = make(chan int, 100)
close(c)
c <- 1 // panic: send on closed channel

```

### A Receive from a Closed Channel Returns the Zero Value immediately

```go
var c = make(chan int, 100)
close(c)
fmt.Println(<-c) // 0

```

If you read from a nil channel, the receiver (goroutine) will block forever.
```go
package main

import "fmt"

func main() {
	var messages chan string // This channel is nil

	fmt.Println("Attempting to receive a message...")
	msg := <-messages // This is where the receiver will block forever
	fmt.Println("Received:", msg) // This line will never be reached
}
```

If you write to a nil channel, the sender (goroutine) will block forever.
```go
package main

import "fmt"

func main() {
	var responses chan int // This channel is nil

	fmt.Println("Attempting to send a response...")
	responses <- 42 // This is where the sender will block forever
	fmt.Println("Response sent!") // This line will never be reached
}
```

*if a program exits before its goroutines have completed, those goroutines 
will be killed silently*

# Take a look a blow function (important)
```go

package main

import (
	"fmt"
	"time"
)

func pingPong(numPings int) {
	pings := make(chan struct{})
	pongs := make(chan struct{})
	go ponger(pings, pongs)
	go pinger(pings, numPings)
	/*
	*
	* here if we just run the function here, nothing will print
	* because we are running three functions "ponger", "pinger" 
	* and anonymous function in async (simultaneously) and
	* pingPong function exits. The "ponger", "pinger" and annonymous
	* function run in background and fail silently
	* 
	* removing 'go' from anonymous function makes "pingPong" 
	* function stop from exiting till goroutines for "pinger" "ponger"
	* finish.
	*
	*/
	go func() {
		i := 0

		/*
		* Unlike ranging over slices or maps where you get 
		* keys/indices and values, when you range over a 
		* channel, you only get the values.
		*/
		for range pongs {
			fmt.Println("got pong", i)
			i++
		}
		fmt.Println("pongs done")
	}()
}

// don't touch below this line

func pinger(pings chan struct{}, numPings int) {
	sleepTime := 50 * time.Millisecond
	for i := 0; i < numPings; i++ {
		fmt.Printf("sending ping %v\n", i)
		pings <- struct{}{}
		time.Sleep(sleepTime)
		sleepTime *= 2
	}
	close(pings)
}

func ponger(pings, pongs chan struct{}) {
	i := 0
	for range pings {
		fmt.Printf("got ping %v, sending pong %v\n", i, i)
		pongs <- struct{}{}
		i++
	}
	fmt.Println("pings done")
	close(pongs)
}

func test(numPings int) {
	fmt.Println("Starting game...")
	pingPong(numPings)
	fmt.Println("===== Game over =====")
}

func main() {
	test(4)
	test(3)
	test(2)
}

```

# Mutexes In GO
Mutexes allow us to lock access to data. This ensures that we can control
which goroutine can access certain data at which time.

Go provides built-in implementation of a mutex with `sync.Mutex` type and
methods:
- `Lock()`
- `Unlock()`

We can protect a block of code by surrounding it with a call to `Lock`
and `Unlock` as shown on the `protected()` function below:

```go

func protected() {
	mu.Lock()
	defer mu.Unlock()
	// the rest of the function is protected
	// any other calls to 'mu.Lock()` will block
}

```

*Mutexes are powerful. Like most powerful things, they can also cause
many bugs if used carelessly*

### Maps are not *Thread-Safe*
Maps are not safe for concurrent use. If you have multiple goroutines
accessing the same map, and at least  one of them is writing to a 
map, you must *Lock* your maps with mutex.

*You lock the resource so only you can access it until you are finished*

Mutex is short of *mutual exclusion* and the conventional name for the 
data structure that provides it is "mutex", often abbreviated to "mu"

It's called *mutual exclusion* because a mutex excludes different threads
(or goroutines) from accessing the same data at the same time.


The principle problem that mutexes help use avoid is the concurrent
*read/write* problem. This problem arises when one thread is writing to
a variable while another thread is reading from that same variable at the same time.

When this happens, a Go program will panic because the reader could be reading
bad data while it's being mutated in place.

Mutexes allow us to safely access shared resources concurrently



# RW Mutex
The standard Library also exposes a `sync.RWMutex`

In addition to these methods:
- `Lock()`
- `Unlock()`

The `sync.RWMutex` also has these methods for concurrent reads:
- `RLock()`
- `RUnlock()`

The `sync.RWmutex` improves performance in read-intensive processes.
Multiple goroutines can safely read from the map simultaneously, as
many `RLock()` calls can occur at the same time. However, only one
goroutine can hold a `Lock()` and during this time all `RLock()` 
operations are blocked.

Maps are safe for concurrent read access, just not concurrent read/write
or write/write access. A read/write mutex allows all the readers to access
the map at the same time, but a writer will still lock out all other readers
and writers.

By using a sync.RWMutex, our program becomes more efficient. We can have as many readLoop() threads as we want, while still ensuring that the writers have exclusive access.

- Only one writers can access a RWMutex at once.
- infinite readers can access a RWMutex at once.
- When a writer calls `Lock()` all read goroutines trying to acquire `RLock()`
will be blocked until the writer calls `Unlock()`

# Generics in Go
Before generics we had to write the same code for each type.

# Type Parameters
Put simply, generics allow us to use variables to refer to a specific
types. This is an amazing feature because it allows us to write
abstract functions that drastically reduce code duplication.
```go

func splitAnySlice[T any](s []T) ([]T, []T) {
	mid := len(s)/2
	return s[:mid], s[mid:]
}

```
In the example above, T is the name of the type parameter for `splitAnySlice`
function and we've said that it must match the any constraint, which means
it can be anything. This makes sense because the body of the function doesn't
care about the types of things stored in the slice.

```go

firstInts, secondInts = splitAnySlice([]int{0, 1, 2, 3})
fmt.Println(firstInts, secondInts)

```
Creating a variable that's the zero value of a type is easy: 
`var myZeroInt int`
Its same with generics we just have a variable that represents the type:
`var myZero T`

# Why Generics?
- Generics Reduce Repetitive Code
You should care about generics because they mean you don't have to 
write as much code. It means you don't have to write same logic for
different type.

- Generics Are Used More Often in Libraries and Packages
Generics give devs an elegant way to write amazing utility packages.
While you will see and use generics in application it is far more 
common to see generics used in libraries and packages.
Libs and packages are intended to be used in many applications, so it
makes sense to write them in a more abstracted way.

# Constraints
Sometimes you need your generic function to know something about the
types it operates on.

Constraints are just interfaces that allow us to write generics that 
only operate within the constraints of a given interface type. In the
example above, the *any* constraints is the same as the empty interface
because it means the type in question can be anything.

### Creating a Custom Constraints
Let's take a look at the example of a `concat` function. It takes a slice
of values and concatenates the values into a string. This should work 
with *any type that can represent itself as a string,* even if it's not
a string under the hood. For example, a user struct can have a `.String()`
that returns a string with user's name and age.

```go

type stringer interface {
	String() string

}

func contact[T stringer](vals []T) string {
	result := ""
	for _, val := range vals {
		// this is where the .String() method
		// is used. That's why we need a more specific
		// constraints instead of the any constraints
		result += val.String()
	}
	return result
}

```

# Interface Type Lists (also known as union type constraints)

When generics were released, a new way of writing interfaces was also
released at the same time!
We can now simply list a bunch of types to a get a new interface/constraints

```go

// Ordered is a type constraint that matches any ordered type.
// An ordered type is one that supports the <, <=, >, and >= operators.
type Ordered interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
        ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 | ~uintptr |
        ~float32 | ~float64 |
        ~string
}

```

# Parametric Constraints

You interface definitions, which can later be used as constraints, can
accept type parameters as well.

```go

// The store interface represents a store that sells products.
// It takes a type parameter P that represents the type of products the store sells.
type store[P product] interface {
	Sell(P)
}

type product interface {
	Price() float64
	Name() string
}

type book struct {
	title  string
	author string
	price  float64
}

func (b book) Price() float64 {
	return b.price
}

func (b book) Name() string {
	return fmt.Sprintf("%s by %s", b.title, b.author)
}

type toy struct {
	name  string
	price float64
}

func (t toy) Price() float64 {
	return t.price
}

func (t toy) Name() string {
	return t.name
}

// The bookStore struct represents a store that sells books.
type bookStore struct {
	booksSold []book
}

// Sell adds a book to the bookStore's inventory.
func (bs *bookStore) Sell(b book) {
	bs.booksSold = append(bs.booksSold, b)
}

// The toyStore struct represents a store that sells toys.
type toyStore struct {
	toysSold []toy
}

// Sell adds a toy to the toyStore's inventory.
func (ts *toyStore) Sell(t toy) {
	ts.toysSold = append(ts.toysSold, t)
}

// sellProducts takes a store and a slice of products and sells
// each product one by one.
func sellProducts[P product](s store[P], products []P) {
	for _, p := range products {
		s.Sell(p)
	}
}

func main() {
	bs := bookStore{
		booksSold: []book{},
	}

    // By passing in "book" as a type parameter, we can use the sellProducts function to sell books in a bookStore
	sellProducts[book](&bs, []book{
		{
			title:  "The Hobbit",
			author: "J.R.R. Tolkien",
			price:  10.0,
		},
		{
			title:  "The Lord of the Rings",
			author: "J.R.R. Tolkien",
			price:  20.0,
		},
	})
	fmt.Println(bs.booksSold)

    // We can then do the same for toys
	ts := toyStore{
		toysSold: []toy{},
	}
	sellProducts[toy](&ts, []toy{
		{
			name:  "Lego",
			price: 10.0,
		},
		{
			name:  "Barbie",
			price: 20.0,
		},
	})
	fmt.Println(ts.toysSold)
}

```

# Enums
Go's type system just isn't as powerful.
More similar to C than Rust.
More concerned about simplicity than expressiveness.

In Go arrays are just values.

In Rust, the 

In Go the developer can choose to ignore error completely, he can 
even decide to use invalid data.
The support for enum in Rust makes it easier to write bug-free code

## Type Definitions
Here is something that can be done in TypeScript but not it Go

```typescript

type sendingChannel = "email" | "sms" | "phone";

function sendNotification(ch: sendingChannel, message: string) {
}

```
`sendingChannel`, a union type, can only be one of the values and if
you try to pass some other value like "slack" TS compiler will throw
error

In Go unfortunately, we don't have these *nice* things

```go
type sendingChannel string

const (
	Email sendingchannel = "email"
	SMS sendingchannel = "sms"
	Phone sendingchannel = "phone"
)

func sendingChanneNotificaion(ch: sendingChannel, message: string) {}

```
The above approach is a bit safer then using plane old `string` in Go. 
But it is not completely safe. Go will stop us from doing this

```go
sendingCh := "slack" // will have an implied type of 'string'
sendNotification(sendingCh, "hello") // error string is not of type 'sendingChannel'

```

But *will* not stop us from doing this
```go

// here "slack" is automatically implied as a 'sendinChannel' type
// which it is not
sendNotification("slack", "hello")

```

And will also not stop you from doing this
```go

sendingCh := "slack"
convertedSendingCh := sendingChannel(sendingCh)

func sendNotification(covertedSendingCh, "hello") {}

```
`sendingChannel` type is just a wrapper for a `string` and 
because we made some constraints of that type, most developers will 
just use those constants: we have made it easy to do the right thing
here. But unfortunately, Go still does not force us to do the right 
thing like TypeScript does.


## iota

Go has language feature, that when used with a type definition 

```go

const sendingChannel int

const (
	email sendingChannel = iota
	sms
	phone
)

```


NOTE: This is one of those things that just happened in go
```go

func (cfg *apiConfig) deleteChirpsByIdHandler(
	w http.ResponseWriter,
	r *http.Request,
) {

	token, err := auth.GetBearerToken(r.Header)
	if err != nil {
		log.Printf("Unable to get bearer token: %v", err)
		ErrorResponse(w, http.StatusUnauthorized, "Internal Server Error")
		return
	}

	// getting user uuid
	userUUID, err := auth.ValidateJWT(token, cfg.jwtSecret)
	if err != nil {
		log.Printf("Unable to validate jwt token: %v", err)
		ErrorResponse(w, http.StatusUnauthorized, "Unauthorized request")
		return
	}

	// getting chirp id provided in client url
	chirpId, err := uuid.Parse(r.PathValue("chirpId"))
	if err != nil {
		log.Printf("Unable to parse uuid: %v", err)
		ErrorResponse(w, http.StatusUnauthorized, "Internal Server Error")
		return
	}

	/*
	 * we have to do a lot of checking and send different error codes back,
	 * first check if the user, chirp exist and then delete the chirp with
	 * related to that user.
	 *
	 */
	_, err = cfg.database.GetUserById(r.Context(), userUUID)
	if err != nil {
		log.Printf("Unable to get user details from the database : %v", err)
		ErrorResponse(w, http.StatusNotFound, "Not Found")
		return
	}

	_, err = cfg.database.GetChirpById(r.Context(), chirpId)
	if err != nil {
		log.Printf("Unable to get chirp details from the database : %v", err)
		ErrorResponse(w, http.StatusNotFound, "Not Found")
		return
	}

	/*
	Go Gotcha, here if you don't declared error like 'err :=' it will be previous
	error variable which is nil you will end up sending StatusNoContent '204',
	So use 'err :=' inside if block because it has its own scope, you did not this
	declare err variable because previously LSP was declaring that no new variable
	has been declared and err variable already exists.
	*/

	if err := cfg.database.DeleteChirpById(r.Context(), database.DeleteChirpByIdParams{
		ID:     chirpId,
		UserID: userUUID,
	}); err != nil {
		log.Printf("Unable to get chirp details from the database : %v", err)
		ErrorResponse(w, http.StatusForbidden, "Forbidden")
		return
	}

	JSONResponse(w, http.StatusNoContent, nil)
}

```
