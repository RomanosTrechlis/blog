[![Build Status](https://travis-ci.org/RomanosTrechlis/go-icls.svg?branch=master)](https://travis-ci.org/RomanosTrechlis/go-icls)
[![Go Report Card](https://goreportcard.com/badge/github.com/RomanosTrechlis/go-icls)](https://goreportcard.com/report/github.com/RomanosTrechlis/go-icls)
[![codecov](https://codecov.io/gh/RomanosTrechlis/go-icls/branch/master/graph/badge.svg)](https://codecov.io/gh/RomanosTrechlis/go-icls)

I usually have some trouble when it comes to creating a cli for an application. That, of course, led me to create my own library.

## ICLS

ICLS stands for Interactive Command Line Session. That means that when the program is executed, it creates its own prompt for entering commands.

For more information of the different types of cli's see [wiki](https://en.wikipedia.org/wiki/Command-line_interface#Operating_system_command-line_interfaces).

## go-icls

The library I can provide both an ICLS and a usual cli. The difference is that the cli takes a command, executes, and exits, while the ICLS return to a prompt instead of exiting.

### Run vs Execute

*Run()* and *Execute(cmd string)* are the definitions of two seperate methods. *Run()* has an infinate loop, waiting for user input, that gets passed to the *Execute(cmd string)* method. The *Execute(cmd string)* method takes as input a line, usually from os.Args.

### Parser

In the core of *Execute(cmd string)* method, lies the parser. It takes the line and returns the command name and a map[string]string for the flags.

### Command definition

Commands are defined beforehand. Every command is part of the *CLI struct*, and has a name, a short description, a long description, and a handler.

The handler is a func that takes a map[string]string, which is the flags that user passed along the command and returns an error:

```go
func(flags map[string]string) error
```

This is an example of a command definition:

```go
c := cli.New() // return a CLI struct without any commands
g := c.New("cmdName", "short description", "long description", func(flags map[string]string) {
	// do something
})
```

### Flag definition

With commands come flags that change the behavior of the command in very specific ways.

A flag has a name, an alias, a data type, a default value, a description, and a required flag. The generic method to add flags to a command is the following:

```go
func (c *command) Flag(name, alias, dataType string, defaultValue interface{}, description string, isRequired bool) error
```

The default value must be of type defined as the dataType of flag.

However, the data type can be ommited when using the specific type methods to define a flag:

```go
g.IntFlag("i", "intFlag", 0, "this is an int flag", false)
g.StringFlag("s", "stringFlag", "0", "this is a string flag", false)
g.BoolFlag("s", "stringFlag", "this is a string flag", false)
```

In bool flag definition, the default value is ommited entirally, because it is expected to be always false, since the existance of the flag in user input implies a true value.

## Example usage

Following is an example program that defines two commands and some flags for each command.

```go
package main

import (
	"fmt"

	"github.com/RomanosTrechlis/go-icls/cli"
)

func main() {
	// create a new cli
	c := cli.New()
	// add 'get' command to cli
	get := c.New("get", "get gets", "get gets", func(flags map[string]string) error {
		fmt.Println("This is the get command")
		return nil
	})
	// add 'get' command flags
	get.StringFlag("d", "", "", "directory name", false)
	get.StringFlag("t", "tetetetetetetet", "", "directory name", true)

	// add 'put' command to cli
	put := c.New("puttertesttest", "putter puts", "putter puts", func(flags map[string]string) error {
		i, err := c.IntValue("g", "puttertesttest", flags)
		if err != nil {
			return err
		}
		fmt.Printf("This is the putter command: %d\n", i)
		return nil
	})
	// add 'put' command flags
	put.StringFlag("f", "file", "", "filename to put", true)
	put.IntFlag("g", "int", 1, "int to put", false)

	c.Run()
}
```

And this is the output:

    $ go-icls
    > -h
    Usage:
    
            go-icls.exe <command> [options]
    
    Commands:
            get                   get gets
            puttertesttest        putter puts
    
    Use "go-icls.exe <command> -h" for more information about a command.
    > get -h
    usage: get [get flags]
    
    get gets
    
    Flags:
    
            -d
                                    directory name
            -h    --help
                                    prints out information about the command
            -t    --tetetetetetetet
                                    directory name (required: true)
    > get -d test
    This is the get command
    > quit

    $


In order to include go-icls functionality into an application, import:
```go
import "github.com/RomanosTrechlis/go-icls/cli"
```

Use the **quit** command to exit the interactive interface.

## TODO

- [ ] Have 100% test coverage.
- [ ] Add var name on printed help after non bool flag (thinking about it).
- [ ] Find a way to simplify the IntValue, BoolValue, etc methods.


Get the full code [here](https://github.com/RomanosTrechlis/go-icls)