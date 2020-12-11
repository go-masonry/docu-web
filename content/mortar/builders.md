---
title: "Builder Pattern"
date: 2020-12-10T13:08:23+02:00
draft: true
---

You probably used to [Functional Options](https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis) pattern.
But we found Builder pattern to be very useful, here we will explain why.

## Motivation

- "See" all the options without searching for them
- Partial Builders
  - Override previously set values

### Library usage within the organization

{{%panel%}}
```
+---------------------+            +-------------------------------+           +-------------------------+
|  Library Developer  | +--------> | Platform/Infra/Ops Developer  | +-------> |  Integration Developer  |
+---------------------+            +-------------------------------+           +-------------------------+

Develops the library to be used    Pre Configure the library specifically      Set final values
in different scenarios, expose     to their organization.                      according to specific use case
different options

     Library defaults                  Predefined defaults                       Override defaults
```
{{%/panel%}}

### Our Library
Let's pretend we are building a library. Our library will have a constructor.

{{%panel%}}
```golang
func NewLib(options ...Option) Library {}

type Option func(*libConf)
```
{{%/panel%}}

Configuration will be stored in `libConf`
{{%panel footer="There can also be a different kind of option that will look similar to this, but the idea is the same"%}}
```golang
type libConf struct {
    address string
    // other options omitted for clarity...
}

func Address(addr string) Option {
    return func(cfg *libConf) {
        cfg.address = addr
    }
}
```
{{%/panel%}}

### What is it about then?

The purpose of this pattern to "introduce" or "unite" between a library developer and the platform/infra developer as shown in the above chart.
We want to introduce predefined defaults that are specific to the organization/use case and are not just library specific.
Let's examine our Library again, we have to set an `address` since our library needs to connect to some server.
We can look at this problem from difference perspectives:

{{%panel header="Library Developer"%}}
The library developer have no idea about your IP address right ? Hence the exposed `Option` to set the address.
However to find that option or any others you will need to look at the source code, right ?
{{%/panel%}}

{{%panel header="Infrastructure/Platform developer within your organization"%}}
Infrastructure/Platform developer already setup this server and now needs to tell every "Integration developer" what that address is
{{%/panel%}}
{{%panel header="Integration Developer"%}}
Either knows or not about this server IP that was introduced by the Platform/Infrastructure team but still needs to have a way to override it.
Because this IP can change with time or there is a local server that is used during tests.
{{%/panel%}}

## Builder

{{%panel%}}
```golang
type LibBuilder interface {
    Address(addr string) LibBuilder
    // ... additional options omitted for clarity
    Build() Library
}
```
{{%/panel%}}

So far nothing new, you can even look at it as a set of Options without the ability to extend and not breaking the API. And you right, however it's not intended as a drop in replacement for Functional Options.

## Linked list

Builder implementation is based on a linked list
{{%panel%}}
```
                                               +-------------------------------+
                                               |                               |
                 +---------------------------->+         Configuration         |
                 |                             |                               |
                 |                             +-------+------------------+----+
                 |                                     ^                  ^
                 |                                     |                  |
       +---------+-----------+         +---------------+-----+         +--+------------------+
       |                     |         |                     |         |                     |
+----->+  Address Option     +-------->+  Max     Option     +-------->+  Min     Option     |
       |                     |         |                     |         |                     |
       +---------------------+         +---------------------+         +---------------------+
```
{{%/panel%}}

Each `Option` is presented as a function on the the builder interface and added a to a list of previous options.
{{%notice%}}Each Builder function is an *alias* to Functional Option{{%/notice%}}

### Overriding predefined defaults

Since it's a list of Options we can add a *new option* to the end of a list that will actually **override** a previous one.
{{%panel%}}
```
                                               +-------------------------------+
                                               |                               |
                 +---------------------------->+         Configuration         |
                 |                             |                               |
                 |                             +-------+------------------+----+
                 |                                     ^                  ^
                 |                                     |                  |
       +---------+-----------+         +---------------+-----+         +--+------------------+         +---------------------+
       |                     |         |                     |         |                     |         |                     |
+----->+  Address Option     +-------->+  Max     Option     +-------->+  Min     Option     +-------->+  Address Option     |
       |                     |         |                     |         |                     |         |                     |
       +---------+-----------+         +---------------------+         +---------------------+         +-----------+---------+
                 ^                                                                                                 |
                 |                                                                                                 |
                 +-----------------------------------------OVERRIDES-----------------------------------------------+
     +                                                                                           +
     |                                                                                           |
     +---------------------------------------Predefined defaults---------------------------------+

```
{{%/panel%}}

## Implementation

Finally let's just build it
{{%panel%}}
```golang
import (
	"fmt"
	"container/list"
)

type libConf struct {
	address string
}

type Library string
type LibBuilder interface {
	Address(addr string) LibBuilder
	// ... additional options omitted for clarity
	Build() Library
}
type libBuilder struct {
	ll *list.List
}

func Builder() LibBuilder {
	return &libBuilder{
		ll: list.New(),
	}
}
func (b *libBuilder) Address(addr string) LibBuilder {
	b.ll.PushBack(func(cfg *libConf) {
		fmt.Printf("using %s as an address\n", addr) // for debug
		cfg.address = addr
	})
	return b
}
func (b *libBuilder) Build() Library {
	var cfg = new(libConf) // empty conf
	// Iterate on the linked list
	for e := b.ll.Front(); e != nil; e = e.Next() {
		f := e.Value.(func(*libConf)) // extract and cast
		f(cfg)
	}
	// at this point cfg will be populated
	return Library(fmt.Sprintf("We are going to call [%s] address", cfg.address)) // or something similar
}
```
{{%/panel%}}

Once we have our builder we will use it as follows

{{%panel%}}
```golang
func makeLibrary(partialBuilder LibBuilder) Library {
    // here we have passed previously defined builder, one that have some defaults already in it
    // Now we can either use it as is or override it
    builder := partialBuilder.Address("5678")
    return builder.Build()
}

func main() {
	var builder = Builder().Address("1234") // predefined builder
	lib:=makeLibrary(builder)
	fmt.Println(lib)
}
```
{{%/panel%}}

Now if you [run it](https://play.golang.org/p/B_le8viHPKt) the output will be
{{%panel%}}
```
using 1234 as an address
using 5678 as an address
We are going to call [5678] address
```
{{%/panel%}}