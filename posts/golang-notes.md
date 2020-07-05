# Golang Notes
### `5 July 2020`


## Some random Golang notes and sample code snippets 

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


## Input/Ouput without using fmt package


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
err := errros.New("test error message")
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
