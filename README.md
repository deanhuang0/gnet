# gnet

## Installation

```
go get -u github.com/izhw/gnet
```

## Features
Building net service quickly with functional options
* [x] TCP Server and Client
* [ ] WebSocket Server and Client
* [ ] Connection pool

## Quick start

#### Examples

* [tcp-server](https://github.com/izhw/gnet/tree/master/examples/tcp/server)
* [tcp-client](https://github.com/izhw/gnet/tree/master/examples/tcp/client)

#### TCP Server

```go
package main

import (
    "fmt"
    "log"
    
    "github.com/izhw/gnet"
    "github.com/izhw/gnet/tcp/tcpserver"
)

type ServerHandler struct {
    *gnet.NetEventHandler
}

func (h *ServerHandler) OnReadMsg(c gnet.Conn, data []byte) error {
    fmt.Println("read msg:", string(data))
    c.Write(data)
    return nil
}

func main() {
    s := tcpserver.NewServer("0.0.0.0:7777", &ServerHandler{})
    log.Fatal("Exit:", s.Serve())
}
```

#### TCP Client
* Sync mode
```go
package main

import (
    "fmt"
    "os"
    
    "github.com/izhw/gnet/tcp/tcpclient"
)

func main() {
    c, err := tcpclient.NewClient("127.0.0.1:7777")
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    defer c.Close()
    // todo...
    data := []byte("Hello world")
    resp, err := c.WriteRead(data)
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    fmt.Println("recv resp:", string(resp))
}
```
* Async mode
```go
package main

import (
    "fmt"
    "os"
    "time"
    
    "github.com/izhw/gnet"
    "github.com/izhw/gnet/tcp/tcpclient"
)

type AsyncHandler struct {
    *gnet.NetEventHandler
}

func (h *AsyncHandler) OnReadMsg(c gnet.Conn, data []byte) error {
    fmt.Println("read msg:", string(data))
    return nil
}

func main() {
    c, err := tcpclient.NewAsyncClient("127.0.0.1:7777", &AsyncHandler{})
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    defer c.Close()
    
    data := []byte("Hello world")
    for i := 0; i < 10; i++ {
        if err := c.Write(data); err != nil {
            fmt.Println("write err:", err)
            os.Exit(1)
        }
        time.Sleep(1 * time.Second)
    }
    fmt.Println("AsyncClient done")
}
```
#### Functional options for znet
for example:
```go
    s := tcpserver.NewServer("0.0.0.0:7777",
        NewServerHandler(),
        gnet.WithLogger(logger.NewSimpleLogger()),
        gnet.WithConnNumLimit(50),
        gnet.WithParseHeaderProtocol(protocol.EncodeProtoVarint, protocol.DecodeProtoVarint),
        ...,
    )
```
See `znet/options.go` for more options

