# client-go [![GoDoc](https://godoc.org/gopkg.in/bblfsh/client-go.v2?status.svg)](https://godoc.org/gopkg.in/bblfsh/client-go.v2) [![Build Status](https://travis-ci.org/bblfsh/client-go.svg?branch=master)](https://travis-ci.org/bblfsh/client-go) [![Build status](https://ci.appveyor.com/api/projects/status/5u1dutc8xdgfoh29?svg=true)](https://ci.appveyor.com/project/vmarkovtsev/client-go) [![codecov](https://codecov.io/gh/bblfsh/client-go/branch/master/graph/badge.svg)](https://codecov.io/gh/bblfsh/client-go)

[Babelfish](https://doc.bblf.sh) Go client library provides functionality to both
connect to the Babelfish server to parse code
(obtaining an [UAST](https://doc.bblf.sh/uast/specification.html) as a result)
and to analyse UASTs with the functionality provided by [libuast](https://github.com/bblfsh/libuast).

## Installation

The recommended way to install *client-go* is:

```sh
go get -d -u gopkg.in/bblfsh/client-go.v2/...
cd $GOPATH/src/gopkg.in/bblfsh/client-go.v2
make dependencies
```

Windows build is supported, though requires custom actions. Read [WINDOWS.md](WINDOWS.md).

## Example

This small example illustrates how to retrieve the [UAST](https://doc.bblf.sh/uast/specification.html) from a small Python script.

If you don't have a bblfsh server running you can execute it using the following command:

```sh
docker run --privileged --rm -d -p 9432:9432 --name bblfsh bblfsh/bblfshd
```

Please read the [getting started](https://doc.bblf.sh/user/getting-started.html) guide, to learn more about how to use and deploy a bblfsh server.

```go
client, err := bblfsh.NewClient("0.0.0.0:9432")
if err != nil {
    panic(err)
}

python := "import foo"

res, err := client.NewParseRequest().Language("python").Content(python).Do()
if err != nil {
    panic(err)
}

query := "//*[@roleImport]"
nodes, _ := tools.Filter(res.UAST, query)
for _, n := range nodes {
    fmt.Println(n)
}
```

```
Import {
.  Roles: Import,Declaration,Statement
.  StartPosition: {
.  .  Offset: 0
.  .  Line: 1
.  .  Col: 1
.  }
.  Properties: {
.  .  internalRole: body
.  }
.  Children: {
.  .  0: alias {
.  .  .  Roles: Import,Pathname,Identifier
.  .  .  TOKEN "foo"
.  .  .  Properties: {
.  .  .  .  asname: <nil>
.  .  .  .  internalRole: names
.  .  .  }
.  .  }
.  }
}

alias {
.  Roles: Import,Pathname,Identifier
.  TOKEN "foo"
.  Properties: {
.  .  asname: <nil>
.  .  internalRole: names
.  }
}

iter, err := tools.NewIterator(res.UAST)
if err != nil {
    panic(err)
}
defer iter.Dispose()

for node := range iter.Iterate() {
    fmt.Println(node)
}
```

Please read the [Babelfish clients](https://doc.bblf.sh/user/language-clients.html) guide section to learn more about babelfish clients and their query language.

## License

Apache License 2.0, see [LICENSE](LICENSE)
