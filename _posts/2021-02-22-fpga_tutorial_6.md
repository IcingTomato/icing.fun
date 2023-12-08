--- 
layout: post
title: "The Tutorial for FPGA Player - Episode 5: Verilog - Part Two"
tags: embedded fpga zh-cn
---

本期，我们将继续讲解之前没有讲完的Verilog代码。上一期的教程已经介绍了Verilog最核心的一些操作，本期则将介绍一些有用的其它操作，他们最终也可以用核心的操作代码表示出来，但是通常而言编写起来更为简便。本期同样会介绍仿真工具的使用，在开发过程中非常有用。不过在开始新的内容之前，先来讲讲上一次留的作业。

# 上期练习

上期留了一个作业，就是实现第五期里面讲过的状态机。这里首先复述一下第五期的状态机：假设一个贩卖机，只卖矿泉水，价格定为2元，只接受1元硬币或者5角硬币，多不找零，设计一个状态机来描述它的行为。这个机器的话，它有两个输入，投入5角或者投入1元；以及一个输出，是否已经付了足够多的钱。

如同之前一样，假设表示投入5角硬币的信号叫a，表示投入1元硬币的信号叫b，输出是否已经付够钱的信号叫c。同时定义这个系统有S0-S4一共5个状态，分别表示当时已经投入了0、0.5、1、1.5和2元。

要用Verilog来实现这个状态，第一步肯定是先写一个整体的模块框架，再往里面加入东西。于是参考上期的声明module的方法，先写下如下的代码：

```verilog
module vending(
    input clk,
    input rst,
    input a,
    input b,
    output c);

endmodule
```

上面的代码定义了一个叫vending的模块，有四个输入，clk、rst、a和b，一个输出c，主体没有内容。clk提供时钟，rst提供复位。首先来考虑输出吧。状态机的输出是由当前状态决定的，所以需要有一个变量（触发器）来保存当前的状态，比如叫做state：

```verilog
reg [2:0] state;
```

有了state之后就可以描述输出的逻辑了。一种方法是直接用第五期的逻辑表达式：

```verilog
assign c = state[2] && !state[1] && !state[0];
```

另外一种方法则是使用always语句块（如上期所说，如果需要在always语句块中赋值，则被赋值的信号需要声明为reg类型，如这里需要把output c修改成output reg c）

```verilog
always @(*) begin
    if (state == 3’d4)
        c = 1‘b1; // 只有在S4输出1
    else
        c = 1’b0;
end
```

两者虽然写法不同，但是最终产生的电路是等效的，而且很有可能是相同的。接下来要处理的就是如何根据输入转换状态了。通常的做法是申明另外一个变量，用来保存即将进入的状态，随后设计两个always语句块，一个负责产生下一个状态，另外一个负责让状态机进入下一个状态。其中产生状态的语句块，应该是异步的，也就是用组合逻辑实现，这样等时钟到来的时候，下一个状态的值就已经是和输入对应的了；而进入下一个状态的语句块则应该是和时钟同步的，使用时序逻辑来实现。

```verilog
reg [2:0] next_state;

always @(*) begin
        next_state = 3’d0;
    if (state == 3’d0) begin
        if ((a == 0)&&(b == 0)) next_state = 3’d0;
        else if ((a == 0)&&(b == 1)) next_state = 3’d2;
        else if ((a == 1)&&(b == 0)) next_state = 3’d1;
    end
    else if (state == 3’d1) begin
        if ((a == 0)&&(b == 0)) next_state = 3’d1;
        else if ((a == 0)&&(b == 1)) next_state = 3’d3;
        else if ((a == 1)&&(b == 0)) next_state = 3’d2;
    end
    else if (state == 3’d2) begin
        if ((a == 0)&&(b == 0)) next_state = 3’d2;
        else if ((a == 0)&&(b == 1)) next_state = 3’d4;
        else if ((a == 1)&&(b == 0)) next_state = 3’d3;
    end
    else if (state == 3’d3) begin
        if ((a == 0)&&(b == 0)) next_state = 3’d3;
        else if ((a == 0)&&(b == 1)) next_state = 3’d4;
        else if ((a == 1)&&(b == 0)) next_state = 3’d4;
    end
    else if (state == 3’d4) begin
        next_state = 3’d4;
    end
end

always @(posedge clk, negedge rst)
begin
    if (!rst)
        state <= 3'd0;
    else
        state <= next_state;
end
```

注意到上面的代码中，产生下一状态的部分，其实就是对第五期的状态表的直接描述，而没有经过任何的化简（第五期内我们首先化简了逻辑）。这也就是用always语句块描述组合逻辑的一个优点，编写的代码可以更接近于要实现的功能一些，而不必须要是具体的门电路逻辑。那么至此，这个作业就写完了。需要指出的是，这只是一种可能的实现，不同的人写状态机有不同的风格，关于不同风格的写法和优劣各位可以自行搜索资料学习。

# Verilog中的其它语句

如之前所说，上期介绍了Verilog中的核心语句，不过Verilog还有一些其它的语句可以方便开发。比如前面的状态机中的if else语句，其实可以用case语句来代替：

```verilog
case (state)
    3'd0: if xxx yyy
    3'd1: if xxx yyy
endcase
```

概念上和C语言的switch语句类似，语法上也比较接近，不过不需要break，同一个条件下多条语句需要用begin end。整体格式如下：

```verilog
case (表达式)
    表达式: 语句
    表达式, 表达式: 语句
    表达式: begin
        语句
        语句
    end
    default: 语句
endcase
```

从上面的格式可以看到，同一个分支可以匹配多个条件，也可以有默认情况。使用case语句不见得会让代码更短，但是使用得当的话可以提高代码的可读性。

那么相比传统的编程语言，还有什么语句缺席了呢？循环语句。仔细考虑一下，Verilog真的需要循环语句吗？硬件中的循环是怎么实现的？循环需要如同之前状态机的结构，一个时钟信号输入，让状态触发器的数值变化，这样来实现类似循环的结构。听起来不应该是用一条语句来实现的东西吧？所以Verilog中就不应该需要循环语句了吧？

不过Verilog还是提供了循环语句。只是这种循环的功能很有限，如同之前的case语句一样，只是一种用于提高代码可读性的做法。Verilog中的循环也只是一种语义上的循环，并非真实的硬件循环。举一个例子，你有4对32位整数，希望把它们加起来：

```verilog
reg [31:0] i1a, i2a, i3a, i4a;
reg [31:0] i1b, i2b, i3b, i4b;
wire [31:0] i1c, i2c, i3c, i4c;

assign i1c = i1a + i1b;
assign i2c = i2a + i2b;
assign i3c = i3a + i3b;
assign i4c = i4a + i4b;
```

这样就能产生四个加法器，同时计算四组加法。但是这样，要更多组加法就要写更多行代码，显然循环会是一种比较好的简便写法。

```verilog
reg [31:0] ia[0:3];
reg [31:0] ib[0:3];
reg [31:0] ic[0:3];

always @(*) begin
    integer i;
    for (i = 0; i < 4; i = i + 1) begin
        ic[i] = ia[i] + ib[i];
    end
end
```

虽然在这个例子中实际的行数相比直接写更多，但是某些情况下（如要操作的数更多，单个操作更为复杂等等），可能是有助于提高可读性的。如同之前所说，这个并不会真正产生循环的硬件，只是语义上的循环。最终产生的硬件是和上面分开写的写法等效且可能是相同的，四个独立的加法器共同工作，而非一个加法器循环处理四组数字。

最后一个不算是语句，只是一个运算符，就是三目判断运算符? :。使用方法和C语言中一致，用在赋值当中，例如如下的用法

```verilog
assign a = b ? c : d;
```

等同于

```verilog
always@(*) begin
    if (b)
        a = c;
    else
        a = d;
end
```

同样，也是一种很好用的简便写法。

# Verilog中的双向信号

双向信号也是代码中需要使用的一个特性。通常来说，不建议在Verilog模块内部使用双向信号，模块间互联应该尽可能使用独立的输入和输出信号。但是，在和外界沟通的时候，如同数据总线一类的信号必须要是双向的。好在，Verilog支持了双向信号。

Verilog中，输入信号的关键词是input，输出信号的关键词是output，双向信号的关键词就很简单的是inout。通常处理双向信号的方法如下：

```verilog
inout signal; // 双向信号
wire direction; // 信号方向
wire signal_input; // 双向信号的输入
wire signal_output; // 双向信号的输出
assign signal = direction ? signal_output : z; // 当方向为输出时输出信号，否则设置为z
assign signal_input = direction ? x : signal; // 当方向为输入时输入信号，否则设置为x
```

需要注意的是，为了控制双向信号，需要一个额外的信号，用来指明信号的方向，这里的情况是定义为了 1 为输出，0为输入，当然这个可以自己修改。另外里面出现了两个以前没有出现过的信号状态，一个是z，一个是x。z表示高阻，或者也可以理解为不输出；x表示无效。当输入输出有多位的时候也应该使用多位的x和z，比如8‘bz表示8位高阻。需要读写的时候，所有的读取从signal_input读取，所有的写入写入进signal_output，随后设置direction为需要的方向。

# 仿真

很多玩单片机的朋友可能不喜欢仿真，觉得程序就是要烧写进板子运行才好玩。也确实是这样，但是就我个人的经验而言，Verilog中仿真重要的多。一来是初学者写代码不熟练时容易出现错误，而这类Bug可能并不容易发现；二来是Verilog的程序综合和实现（类似于软件中的编译吧）速度相比于软件而言慢得多，即使是很简单的代码通常也需要好几分钟才能完成，复杂的则需要数十分钟甚至数小时。一般以前调试小软件中那种修改一下编译测试一下是否修复的方法在这就并不适用了。在综合前最好先确认代码是能用的，而确认的方法就是通过仿真。本期就以上面的状态机为例，演示一下如何进行仿真。因为这个和所使用的FPGA有关，这里依然是分成Intel和Xilinx两部分演示。

## Intel

大体的流程和之前第三期时的流程是一致的，因为最终目的也都是把代码综合、实现之后下载到开发板当中运行。创建工程的步骤这里就不一一赘述，具体的可以参考第三期中的过程，这里只是讲几个重点。

首先在选择设备的界面，如果最终需要把设计烧录进FPGA测试，则必须要选择对应的型号，否则可以随意选择。DE10-Lite开发板上的器件型号为10M50DAF484C7G。

![](http://icing.fun/img/post/2021/02/22/6/1551048119359-6-1.png)

在新的工程中建立两个文件，一个是状态机的源代码，另外一个用于仿真的测试代码（test bench）。首先来建立状态机的源代码。直接按照上面的步骤输入代码，但是需要注意的是模块名需要和文件名一致。我这里文件名保存为lesson_7.v，模块名称也就需要相对应的修改成lesson_7。代码应该如下：

![](http://icing.fun/img/post/2021/02/22/6/1551048120706-6-2.png)

随后是仿真文件，或者叫testbench。Testbench也是一段Verilog代码，不过并不会被综合成硬件，只会在仿真器中运行，作用是测试需要真实下载到板子里的程序。本次要用的Testbench如下：

```verilog
module testbench();

    reg clk, rst, a, b;
    wire c;
    wire [2:0] state;
	 
    lesson_7 DUT(clk, rst, a, b, state, C);
	 
    initial
    begin
        // 复位
        a = 0; b = 0; rst = 0;
        #(5) clk = 1;
        #(5) clk = 0;
		  
        // 什么都不做
        a = 0; b = 0; rst = 1;
        #(5) clk = 1;
        #(5) clk = 0;

        // 投入5角
        a = 1; b = 0; rst = 1;
        #(5) clk = 1; 
        #(5) clk = 0;

        // 投入5角
        a = 1; b = 0; rst = 1;
        #(5) clk = 1; 
        #(5) clk = 0;
		  
        // 投入5角
        a = 1; b = 0; rst = 1;
        #(5) clk = 1; 
        #(5) clk = 0;
		  
        // 投入5角
        a = 1; b = 0; rst = 1;
        #(5) clk = 1; 
        #(5) clk = 0;

        // 检查是否已经被解锁
        if (c != 1) begin
             $display("测试1失败 Test 1 Failed");
             $finish;
        end
		  
        // 复位
        a = 0; b = 0; rst = 0;
        #(5) clk = 1;
        #(5) clk = 0;
		  
        // 检查是否已经被复位
        if (c != 0) begin
            $display("测试2失败 Test 2 Failed");
            $finish;
        end
		  
        // 投入1元
        a = 0; b = 1; rst = 1;
        #(5) clk = 1; 
        #(5) clk = 0;

        // 投入1元
        a = 0; b = 1; rst = 1;
        #(5) clk = 1; 
        #(5) clk = 0;
		  
        // 检查是否已经被解锁
        if (c != 1) begin
            $display("测试3失败 Test 3 Failed");
            $finish;
        end
		  
        $display("测试成功 Test Success");
        $finish;
	  
    end
endmodule
```

大体来说就是，先实例化要测试的模块。随后依次提供输入。提供输入后再使用if语句来测试输出是否符合预期，如果不符则提示失败并结束仿真。里面的#(5)表示延迟5ns。每条提示中都加入了中英文两种提示，原因是Intel这边用的ModelSim仿真工具并不能显示中文，只能加上英文输出；而Xilinx这边用的ISim没有这个问题。

点击菜单中的Tools-Run Simulation Tool-RTL Simulation运行仿真：

![](http://icing.fun/img/post/2021/02/22/6/1551048122442-6-3.png)

如果你遇到了没有指定仿真工具的错误，则需要进入设置（Tools-Options）里面手动指定，如下图：

![](http://icing.fun/img/post/2021/02/22/6/1551048124284-6-4.png)

仿真语言选择Verilog HDL。ModelSim启动后，选择菜单中的Compile – Compile… 选择需要仿真的文件，这里就是lesson_7.v和testbench.v两个文件：

![](http://icing.fun/img/post/2021/02/22/6/1551048125970-6-5.png)

编译完成后应该可以在work里看见这两个模块：

![](http://icing.fun/img/post/2021/02/22/6/1551048127802-6-6.png)

要开始仿真，在testbench上右键，选择Simulate。进入仿真界面后，可以选择需要的信号观察波形输出，比如这里选择所有的信号：

![](http://icing.fun/img/post/2021/02/22/6/1551048129661-6-7.png)

最后点击上方的Run开始仿真。因为目前的仿真还很简单，很快就会完成，会出现是否结束的提示。如果需要观察波形则点否，如果点是将会直接关闭仿真软件。从下面的输出可以看到测试成功完成了，状态机表现和预期一致。

![](http://icing.fun/img/post/2021/02/22/6/1551048131324-6-8.png)

那么至此我们已经知道这个状态机是可以用的了，可以烧写进板子测试了。不过别忘了，在烧写进板子之前，需要分配引脚定义。这里比如让LEDR0为c输出，SW1和SW0分别为a和b输入，而KEY0作为rst，KEY1作为clk，最后同时在LED3-LED1上输出当前的状态。

根据原理图，不难找到这些IO对应的引脚：

 - SW0 – C10
 - SW1 – C11
 - LEDR0 – A8
 - LEDR1 – A9
 - LEDR2 – A10
 - LEDR3 – B10
 - KEY0 – B8
 - KEY1 – A7

 这些硬件的电压都为3.3V（原理图里指出）。

将对应的引脚信息输入进Pin Planner，把电压都设定到3.3V，就完成了引脚分配。

![](http://icing.fun/img/post/2021/02/22/6/1551048134018-6-9.png)

完成后重新生成编程文件（Generate programming files），运行烧写工具（Tools-Programmer）烧写进板子即可。

## Xilinx

Xilinx这边的过程也是类似的，首先是建立工程，选择目标设备，随后建立文件编写代码。值得注意的是，Xilinx这边在建立仿真文件的时候需要选择Verilog Test Fixure：

![](http://icing.fun/img/post/2021/02/22/6/1551048135672-6-10.png)

随后便会提示这个testbench对应的是哪个模块，这里唯一的模块就是lesson_7，直接继续。可以注意到ISE自动生成了testbench的框架，我们只需要编写initial begin end内的测试输入输出即可，十分方便。在补充完成测试主体后应该如图：

![](http://icing.fun/img/post/2021/02/22/6/1551048138121-6-11.png)

在左边的任务窗格选择Simulation，选中Testbench，在下方的ISim Simulator中选择Simulate Behavioral Model（仿真行为模型）

如果没有出现错误，Isim会自动打开并且运行测试，应该可以直接在下方看到测试成功的提示，同时在上方也可以直接看到所有的测试波形。

![](http://icing.fun/img/post/2021/02/22/6/1551048140228-6-12.png)

现在我们已经简单测试了这个状态机是可以工作的，剩下的步骤也就是编辑UCF文件生成编程文件下载了。UCF文件同样是按照实际硬件的定义来：

```verilog
NET a LOC="AJ6";
NET a IOSTANDARD="LVCMOS33";
NET b LOC="AK7";
NET b IOSTANDARD="LVCMOS33";
NET clk LOC="U8";
NET clk IOSTANDARD="LVCMOS33";
NET rst LOC="V8";
NET rst IOSTANDARD="LVCMOS33";

NET c LOC="H18";
NET c IOSTANDARD="LVCMOS25";
NET state<0> LOC="L18";
NET state<0> IOSTANDARD="LVCMOS25";
NET state<1> LOC="G15";
NET state<1> IOSTANDARD="LVCMOS25";
NET state<2> LOC="AD26";
NET state<2> IOSTANDARD="LVCMOS25";
```

保存后生成编程文件，使用Tools-iMPACT下载至板子即可。

# 总结

本次的教程到这里也就差不多告一段落了。本期我们继续了上期没有完成的Verilog教学，讲解了一个简单的作业，并且在Intel和Xilinx两家的环境里对程序进行了仿真，这些将会是之后经常需要使用的功能。这里也给大家留个作业：想要把这个状态机的代码修改下，让它只需要投入1.5元就可以购买，需要怎么做呢？提示：根据你的实现，可能最少只需要修改1行代码即可。而下期开始我们将正式开始谈一谈CPU这个概念，也就是整个游戏机的核心。我们下次见。
