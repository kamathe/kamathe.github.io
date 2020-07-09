# Golang Notes
### `5 July 2020`


## Some random Golang notes and sample code snippets 


- [import package without using](#import-package-without-using-it)
- [directly using packages from github](#directly using packages from github)
- [Different print functions](#different-print-functions)
- [Capturing command-line args](#capturing-command-line-args)
- [Input Ouput without using fmt package](#input-ouput-without-using-fmt-package)
- [logging to files](#logging-to-files)
- [Errors and Error handling](#errors-and-error-handling)
- [Compile to object file and archive file](#compile-to-object-file-and-archive-file)
- [Garbage collection stats print](#garbage-collection-stats-print)
- [Golang execution environment details](#golang-execution-environment-details)
- [Cross compilation](#cross-compilation)
- [Print Node tree information](#print-node-tree-information)
- [Go build versbose info](#go-build-versbose-info)
- [Golang basic types](#golang-basic-types)



## import package without using it

```
# you can put an _ before the package name

import (
	"fmt"
	_ "os"
)
```


## directly using packages from github

```
# you can import package from github as below

import (
	"fmt"
	"github.com/user/project"
)

# before using above you need to do go get

go get -v github.com/user/project

# the package gets installed in $GOROOT

$HOME/go/src/github

# delete intermediate files of downloaded go package

go clean -i -v -x github.com/user/project
rm -rf $HOME/go/src/github.com/user/project
```


## Different print functions


```

# simple prints

fmt.Print()		# no new line
fmt.Println() 	# new line towards the end
fmt.Printf() 	# formatting with type of data like %s,%d etc

# S prints for creating strings based on given format

fmt.Sprint()
fmt.Sprintln()
fmt.Sprintf()

# F prints used for writing to files using io.Writer

fmt.Fprint()
fmt.Fprintln()
fmt.Fprintf()
```


## Capturing command-line args

```

# capture CLI args to a program

os.Args

# check length of args

len(os.Args)

# accessing individual elements

os.Args[0]	# program itself
os.Args[1]	# first argument
os.Args[2]	# second argument

# assign args to a variable

arguments := os.Args

# iterating over args
```


## Input Ouput without using fmt package


```

# using io package and os package
# first param for io.WriteString should be io.Writer interface
# second param should be slice of bytes to write to output

io.WriteString(os.Stdout, "test string")
io.WriteString(os.Stderr, "error string")

# keep waiting for user input, repeat entered input on screen

var f \*os.File
f = os.Stdin
defer f.Close()

scanner := bufio.NewScanner(f)
for scanner.Scan() {
	fmt.Println(">> ", scanner.Text())
}
```



## Logging to files


```

# various log functions

log.Print()
log.Println()
log.Printf()

# fatal

log.Fatalf()
log.Fatalln()

# panic

log.Panic()
log.Panicln()
log.Panicf()


# how to log to syslog

sysLog, err := syslog.New(syslog.LOG_INFO, os.Args[0])

log.SetOutput(sysLog)
log.Println("test log line")


# when you want to exit your program suing log.Fatal
# only exit status

sysLog, err := syslog.New(syslog.LOG_INFO, os.Args[0])
log.Fatal(sysLog)

# when you want to print stack trace and then exit


sysLog, err := syslog.New(syslog.LOG_INFO, os.Args[0])
log.Panic(sysLog)
```



## Errors and Error handling

```

# introduce new errors

err := errors.New("test error message")
return err

# checking errors

if err.Error() == "test error message" {
	do something
}

# handling error, printing to screen

if err != nil {
	fmt.Println(err)	# print to screen
	os.Exit(1)
}

# handling error, send err message to loggin service

if err != nil {
	log.Println(err)	# send to logs
	os.Exit(1)
}

# handling error, terminate program

if err != nil {
	panic(err)			# panicking
	os.Exit(1)
}
```


## Compile to object file and archive file

```

# compile to object file (.o) exntention

go tool compile hello.go


# view disassembly of object file 

go tool objdump hello.o

# compile to archive file (.a) extention

go tool compile -pack hello.o


# read contents of archive file

ar t hello.go

# extract contents of archive file
# check other options via go doc cmd/pack

go tool pack x hello.a

# dump generate assembly to screen before compiling

go tool compile -S hello.go

```


## Garbage collection stats print

```

# need to import runtime library for this to work

import runtime

# need to initialize a special variable

var mem runtime.MemStats

# then use the above mem variable to print stats
# you can wrap this in a function which accepts mem as arg
# and then do some memory intesive ops and then keep calling
# the function to check what current GC stats are

mem.Alloc
mem.TotalAlloc
mem.HeapAlloc
mem.NumGC


# another way of doing this is while invoking the program
# set a env variable before running the program
# this provides info heap sizes like 85->85->0 etc

GODEBUG=gctrace=1 go run hello.go
```


## Golang execution environment details


```

# we need to import runtime

import runtime

# print env details using the following

runtime.Compiler
runtime.GOARCH
runtime.Version()		# version of golang
runtime.NumCPU()
runtime.NumGoroutines()
```



## Cross compilation

```

# cross compile for different archs

GOOS=linux GOARCH=amd64 go build hello.go

# output assembly
# FUNCDATA and PCDATA are directives used by GC

GOOS=linux GOARCH=arm64 go tool compile -S hello.go

```


## Print Node tree information


```

# print node tree information

go tool compile -W hello.go

# also we can grep before and after

go tool compile -W hello.go | grep before
go tool compile -W hello.go | grep after

# or search variable and func names

go tool compile -W hello.go | grep <var-name>
go tool compile -W hello.go | grep <func-name>


```



## Go build versbose info

```

# print verbose details of go build command using -x

go build -x hello.go

```



## Golang basic types


```

# basic types

bool
string


# float

float32
float64

# complex

complex64
complex128

# other

byte	# alias for uint8
rune	# alias for uint32

# numeric types

int8	# (-128 to 127)
int16	# (-32768 to 32767)
int32	# (-2147483648 to 214748647)
int64	# (-9223372036854775808 to 9223372036854775807)


uint8	# (0 to 255)
unit16	# (0 to 65535)
uint32	# (0 to 4294967295)
uint64	# (0 to 18446744073709551615)
```






