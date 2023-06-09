### GoDivert
go语言使用开源divert库进程流量控制

基于https://github.com/williamfhe/godivert修复了该库的报错问题

### 例子
### 捕获和打印数据包
```go
package main

import (
    "fmt"
    "github.com/longyixiao/godivert"
)

func main() {
    winDivert, err := godivert.NewWinDivertHandle("true")
    if err != nil {
        panic(err)
    }

    packet, err := winDivert.Recv()
    if err != nil {
        panic(err)
    }
    defer winDivert.Close()

    fmt.Println(packet)

    packet.Send(winDivert)

}
```
### 按IP阻止协议

```go
package main

import (
    "net"
    "time"
    "github.com/longyixiao/godivert"
)

var cloudflareDNS = net.ParseIP("1.1.1.1")

func checkPacket(wd *godivert.WinDivertHandle, packetChan <-chan *godivert.Packet) {
    for packet := range packetChan {
        if !packet.DstIP().Equal(cloudflareDNS) {
            packet.Send(wd)
        }
    }
}

func main() {
    winDivert, err := godivert.NewWinDivertHandle("icmp")
    if err != nil {
        panic(err)
    }
    defer winDivert.Close()

    packetChan, err := winDivert.Packets()
    if err != nil {
        panic(err)
    }

    go checkPacket(winDivert, packetChan)

    time.Sleep(1 * time.Minute)
}
```
禁止所有ICMP数据包在1分钟内达到1.1.1.1。

```bash
ping 1.1.1.1
```

###数据包计数

```go
package main

import (
    "fmt"
    "time"
    "github.com/longyixiao/godivert"
    "github.com/longyixiao/godivert/header"
)

var icmpv4, icmpv6, udp, tcp, unknown, served uint

func checkPacket(wd *godivert.WinDivertHandle, packetChan  <- chan *godivert.Packet) {
    for packet := range packetChan {
        countPacket(packet)
        wd.Send(packet)
    }
}

func countPacket(packet *godivert.Packet) {
    served++
    switch packet.NextHeaderType() {
    case header.ICMPv4:
        icmpv4++
    case header.ICMPv6:
        icmpv6++
    case header.TCP:
        tcp++
    case header.UDP:
        udp++
    default:
        unknown++
    }
}


func main() {
    winDivert, err := godivert.NewWinDivertHandle("true")
    if err != nil {
        panic(err)
    }

    fmt.Println("Starting")
    defer winDivert.Close()

    packetChan, err := winDivert.Packets()
    if err != nil {
        panic(err)
    }

    n := 50
    for i := 0; i < n; i++ {
        go checkPacket(winDivert, packetChan)
    }

    time.Sleep(15 * time.Second)

    fmt.Println("Stopping...")

    fmt.Printf("Served: %d packets\n", served)

    fmt.Printf("ICMPv4=%d ICMPv6=%d UDP=%d TCP=%d Unknown=%d", icmpv4, icmpv6, udp, tcp, unknown)
}

```
