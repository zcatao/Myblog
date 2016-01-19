title: quartus实现ram
date: 2016-01-19 18:56:44
tags: [FPGA,quartus]
categories:
---
>   最近课设利用FPGA做一个计时器，当然这么简单的课设实在是太恶心和无聊，那就加点料！来个记忆功能，就类似于市面上那种秒表可以计时每圈的时间一样。那这就得想办法在FPGA中综合出ram啦。

在实现过程中主要有两种方法，一种是利用quartus的LPM模块通过配置模块参数实现内置的LPM_ram模块，另外一种通过简单的自己手写的语句自己综合，但是注意自己直接写的话，最后综合可能使用的是FPGA中分布式的ram,也就是各LE中的寄存器模块，这样无疑会造成巨大的浪费，但无奈这样浅显易懂，更容易掌握LPM_RAM的工作方式。

### 先上一个简单的RAM 模型

``` verilog
module ram_1port(
    input [3:0] address;
    input       clk,
    input [23:0]data,
    input       wren,
    output[23:0]q
    )

    reg[23:0] mem[7:0];

    always @ (posedge clk) begin
        if(wren)
            mem[address] <= data;
    end

    assign q = mem[address];

endmodule
```
