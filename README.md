### GoDivert
go语言使用开源divert库进程流量控制

基于https://github.com/williamfhe/godivert修复了该库的报错问题

### 例子
捕获和打印数据包

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
