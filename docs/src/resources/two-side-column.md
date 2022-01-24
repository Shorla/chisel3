<!Doctype html>
<html>
<title> Two side column </title>

```scala mdoc:invisible
import Chisel.Queue
import chisel3._
import chisel3.util.{DecoupledIO, switch, is}
import chisel3.stage.ChiselStage
import chisel3.experimental.ChiselEnum
import chisel3.util.{Cat, Fill, DecoupledIO}
```

<body>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <table border ="0">
        <h1>Creating a Module</h1>
        <tr>
            <td><b style="font-size:30px">Verilog</b></td>
            <td><b style="font-size:30px">Chisel</b></td>
         </tr>
<tr>
<td>

```
    module foo (
                    input  a,
                    output b
                )
                assign b = a;
            endmodule
```

</td>
    <td>

```scala mdoc
    class Foo extends Module {
    val a = Input(Bool())
    val b = Output(Bool())
    b := a
    }
```

</td>
</tr>
<tr>
<td>

```
module PassthroughGenerator(
  input        clock,
  input        reset,
  input  [9:0] io_in,
  output [9:0] io_out
);
  assign io_out = io_in; // @[main.scala 13:10]
endmodule

module PassthroughGenerator(
input         clock,
input         reset,
input  [19:0] io_in,
output [19:0] io_out
);
assign io_out = io_in; // @[main.scala 13:10]
endmodule
```
</td>
<td>

```scala mdoc:silent
class PassthroughGenerator(width: Int) extends Module {
val io = IO(new Bundle {
val in = Input(UInt(width.W))
val out = Output(UInt(width.W))
})
io.out := io.in
}
```
```scala mdoc
ChiselStage.emitVerilog(new PassthroughGenerator(10))
ChiselStage.emitVerilog(new PassthroughGenerator(20))
```
</td>
         </tr>
    </table>
</body>
</html>

# Parameterizing a Module

<html>
<body>
    <table border ="0">
          <tr>
            <td><b style="font-size:30px">Verilog</b></td>
            <td><b style="font-size:30px">Chisel</b></td>
         </tr>
         <tr>
<td>

```
module ParameterizedWidthAdder(
input [in0Width-1:0] in0,
input [in1Width-1:0] in1,
output [sumWidth-1:0] sum
);
parameter in0Width = 8;
parameter in1Width = 1;
parameter sumWidth = 9;

assign sum = in0 + in1;

endmodule
```

</td>
<td>

```
class ParameterizedWidthAdder(in0Width: Int, in1Width: Int, sumWidth: Int) extends Module {
  require(in0Width >= 0)
  require(in1Width >= 0)
  require(sumWidth >= 0)
  val io = IO(new Bundle {
    val in0 = Input(UInt(in0Width.W))
    val in1 = Input(UInt(in1Width.W))
    val sum = Output(UInt(sumWidth.W))
  })
  // a +& b includes the carry, a + b does not
  io.sum := io.in0 +& io.in1
}
```
</td>

</tr>
<tr>
<td>

```
module TestBench;
wire [31:0] sum;
ParameterizedWidthAdder  #(32, 32, 32) my32BitAdderWithTruncation (32'b0, 32'b0, sum);
endmodule
```
</td>
            <td>

```
val my32BitAdderWithTruncation = Module(new ParameterizedWidthAdder(32, 32, 32)
```
</td>
 <tr>
<td>

```scala mdoc:silent
class MyModule extends Module {
  val io = IO(new Bundle {
    val in  = Input(UInt(4.W))
    val out = Output(UInt(4.W))
  })

val two  = 1 + 1
println(two)
val utwo = 1.U + 1.U
println(utwo)

io.out := io.in
}
```
```scala mdoc:invisible
ChiselStage.emitVerilog(new MyModule)
```
</td>
<td>

```
2
UInt<1>(OpResult in MyModule)
module MyModule(
  input        clock,
  input        reset,
  input  [3:0] io_in,
  output [3:0] io_out
);
  assign io_out = io_in; // @[main.scala 19:10]
endmodule
```
</td>
         </tr>
    </table>
<html>
<body>

# Wire assignment

<html>
<body>
    <table border ="0">
          <tr>
            <td><b style="font-size:30px">Verilog</b></td>
            <td><b style="font-size:30px">Chisel</b></td>
         </tr>
         <tr>
<td>

```
wire [31:0] 
a = 32'd42; 
wire [31:0] 
b = 32'hbabecafe; 
wire [15:0] c; 
assign c = 16'b1;
```

</td>
<td>

```scala mdoc:silent


class MyWireAssignmentModule extends Module {
 val a = WireDefault(42.U(32.W))
 val b = WireDefault("hbabecafe".U(32.W))  
 val c = Wire(UInt(16.W)) 
  c := "b1".U;
}
```
```scala mdoc:invisible
ChiselStage.emitVerilog(new MyWireAssignmentModule)
```
</td> 
</tr>
<tr>
<td>

```
module MyWireAssignmentModule(
  input   clock,
  input   reset
);
endmodule
```


</td>
<td>

```scala mdoc:silent
class MyWireAssignmentModule2 extends Module {
 val a = WireDefault(42.U(32.W))
 val aa = 42.U(32.W)
 val b = WireDefault("hbabecafe".U(32.W))  
 val c = Wire(UInt(16.W)) 
 val d = Wire(Bool())
 val e = Wire(UInt())
 val f = WireDefault("hdead".U)
 val g = Wire(SInt(64.W))
 val h = WireDefault(5.asSInt(32.W))
 val i = WireDefault((3.S(16.W)))

c := "b1".U;
d := true.B
e := "b1".U(3.W);
g := -5.S
}
```
```scala mdoc:invisible
ChiselStage.emitVerilog(new MyWireAssignmentModule2)
```

</td>

</tr>
    </table>
<html>
<body>

# Register assignment

<html>
<body>
    <table border ="0">
          <tr>
            <td><b style="font-size:30px">Verilog</b></td>
            <td><b style="font-size:30px">Chisel</b></td>
         </tr>
         <tr>
<td>

```
module RegisterModule(
  input         clock,
  input         reset,
  input  [11:0] io_in,
  output [11:0] io_out
);
  assign io_out = io_in; // @[main.scala 18:10]
endmodule
```
</td>
<td>

```scala mdoc:silent
class RegisterModule extends Module {
  val io = IO(new Bundle {
    val in  = Input(UInt(12.W))
    val out = Output(UInt(12.W))
  })

  val registerWithInit = RegInit(42.U(12.W))
    registerWithInit := registerWithInit - 1.U
    io.out := io.in
}
```
```scala mdoc:invisible
ChiselStage.emitVerilog(new RegisterModule)
```
</td>
</tr>
</table>
<html>
<body>

# Case statement

<html>
<body>
    <table border ="0">
          <tr>
            <td><b style="font-size:30px">Verilog</b></td>
            <td><b style="font-size:30px">Chisel</b></td>
         </tr>
         <tr>
<td>

```
module CaseStatementModule(
  input        clock,
  input        reset,
  input  [2:0] a,
  input  [2:0] b,
  input  [2:0] c,
  input  [1:0] sel,
  output [2:0] out
);
  wire [2:0] _GEN_0 = 2'h2 == sel ? c : 3'h0; // @[main.scala 16:16 28:8 14:9]
  wire [2:0] _GEN_1 = 2'h1 == sel ? b : _GEN_0; // @[main.scala 16:16 24:8]
  assign out = 2'h0 == sel ? a : _GEN_1; // @[main.scala 16:16 19:8]
endmodule
```
</td>
<td>

```scala mdoc:silent
class CaseStatementModule extends Module {
   val a, b, c= IO(Input(UInt(3.W)))
    val sel = IO(Input(UInt(2.W)))
  val out = IO(Output(UInt(3.W)))
    out := 0.U
    
  switch (sel) {
  is ("b00".U) {
   out := a 
  }
  
  is ("b01".U) { 
   out := b
  }
  
  is ("b10".U) {
   out := c
  }
}
  };
```
```scala mdoc:invisible
ChiselStage.emitVerilog(new CaseStatementModule)
```
</td>
         </tr>
    </table>
<html>
<body>

# Case statement Enum Module 

<html>
<body>
    <table border ="0">
          <tr>
            <td><b style="font-size:30px">Verilog</b></td>
            <td><b style="font-size:30px">Chisel</b></td>
         </tr>
         <tr>
<td>

```
module CaseStatementModule(
  input        clock,
  input        reset,
  input  [2:0] a,
  input  [2:0] b,
  input        sel,
  output [2:0] out
);
  wire [2:0] _GEN_0 = sel ? b : 3'h0; // @[main.scala 22:16 30:8 20:9]
  assign out = ~sel ? a : _GEN_0; // @[main.scala 22:16 25:8]
endmodule
```
</td>
<td>


```scala mdoc:silent
class CaseStatementEnumModule1 extends Module {
  
  object AluMux1Sel extends ChiselEnum {
  val selectRS1, selectPC = Value
  }
  
import AluMux1Sel._
   val a, b = IO(Input(UInt(3.W)))
    val sel = IO(Input(AluMux1Sel()))
  val out = IO(Output(UInt(3.W)))
    out := 0.U
    
  switch (sel) {
  is (selectRS1) {
   out := a
  }
  is (selectPC) {
   out := b
  }
}
  };
```
```scala mdoc:invisible
ChiselStage.emitVerilog(new CaseStatementEnumModule1)
```
</td>
         </tr>
  <tr>
<td>

```
module CaseStatementEnumModule2 (input clk);
 
  typedef enum {INIT, IDLE, START, READY} t_state;
    t_state state;
    
    
 
    always @(posedge clk) begin
        case (state)
            IDLE    : state = START;
            START   : state = READY;
            READY   : state = IDLE ;
            default : state = IDLE ;
        endcase
    end
    
endmodule
```
</td>
<td>

```scala mdoc:silent
class CaseStatementEnumModule2 extends Module {
  
  object AluMux1Sel extends ChiselEnum {
   val INIT  = Value(0x03.U) 
    val IDLE  = Value(0x13.U) 
    val START = Value(0x17.U) 
    val READY = Value(0x23.U) 
  }
   import AluMux1Sel._
    val state = RegInit(INIT)
  val nextState = IO(Output(AluMux1Sel()))
  nextState := state

  
  switch (state) {
  is (INIT) {
   state := IDLE   
  }
  is (IDLE) {  
   state := START
  }
  is (START) { 
   state := READY
  }
 is (READY) { 
   state := IDLE
  }
  }
}
```
```scala mdoc:invisible
ChiselStage.emitVerilog(new CaseStatementEnumModule2)
```
</td>
<td>

```
module CaseStatementModule(
  input        clock,
  input        reset,
  output [5:0] nextState
);
`ifdef RANDOMIZE_REG_INIT
  reg [31:0] _RAND_0;
`endif // RANDOMIZE_REG_INIT
  reg [5:0] state; // @[main.scala 20:24]
  wire [5:0] _GEN_0 = 6'h23 == state ? 6'h13 : state; // @[main.scala 25:18 41:10 20:24]
  assign nextState = state; // @[main.scala 22:13]
  always @(posedge clock) begin
    if (reset) begin // @[main.scala 20:24]
      state <= 6'h3; // @[main.scala 20:24]
    end else if (6'h3 == state) begin // @[main.scala 25:18]
      state <= 6'h13; // @[main.scala 28:10]
    end else if (6'h13 == state) begin // @[main.scala 25:18]
      state <= 6'h17; // @[main.scala 33:10]
    end else if (6'h17 == state) begin // @[main.scala 25:18]
      state <= 6'h23; // @[main.scala 37:10]
    end else begin
      state <= _GEN_0;
    end
  end
// Register and memory initialization
`ifdef RANDOMIZE_GARBAGE_ASSIGN
`define RANDOMIZE
`endif
`ifdef RANDOMIZE_INVALID_ASSIGN
`define RANDOMIZE
`endif
`ifdef RANDOMIZE_REG_INIT
`define RANDOMIZE
`endif
`ifdef RANDOMIZE_MEM_INIT
`define RANDOMIZE
`endif
`ifndef RANDOM
`define RANDOM $random
`endif
`ifdef RANDOMIZE_MEM_INIT
  integer initvar;
`endif
`ifndef SYNTHESIS
`ifdef FIRRTL_BEFORE_INITIAL
`FIRRTL_BEFORE_INITIAL
`endif
initial begin
  `ifdef RANDOMIZE
    `ifdef INIT_RANDOM
      `INIT_RANDOM
    `endif
    `ifndef VERILATOR
      `ifdef RANDOMIZE_DELAY
        #`RANDOMIZE_DELAY begin end
      `else
        #0.002 begin end
      `endif
    `endif
`ifdef RANDOMIZE_REG_INIT
  _RAND_0 = {1{`RANDOM}};
  state = _RAND_0[5:0];
`endif // RANDOMIZE_REG_INIT
  `endif // RANDOMIZE
end // initial
`ifdef FIRRTL_AFTER_INITIAL
`FIRRTL_AFTER_INITIAL
`endif
`endif // SYNTHESIS
endmodule
```
</td>
         </tr>
    </table>
<html>
<body>

# SystemVerilog Interfaces

<html>
<body>
    <table border ="0">
          <tr>
            <td><b style="font-size:30px">Verilog</b></td>
            <td><b style="font-size:30px">Chisel</b></td>
            <td><b style="font-size:30px">Generated Verilog</b></td>
         </tr>
         <tr>
<td>Text comes here</td>
<td>

```scala mdoc:silent
class MyInterfaceModule extends Module {
val io = IO(new Bundle {
val in = Flipped(DecoupledIO(UInt(8.W)))
val out = DecoupledIO(UInt(8.W))
})

val tmp = Wire(DecoupledIO(UInt(8.W)))
tmp <> io.in
io.out <> tmp
io.out <> io.in
}
```
```scala mdoc:invisible
ChiselStage.emitVerilog(new MyInterfaceModule)
```
</td>
<td>

```
module MyInterfaceModule(
  input        clock,
  input        reset,
  output       io_in_ready,
  input        io_in_valid,
  input  [7:0] io_in_bits,
  input        io_out_ready,
  output       io_out_valid,
  output [7:0] io_out_bits
);
  assign io_in_ready = io_out_ready; // @[main.scala 17:12]
  assign io_out_valid = io_in_valid; // @[main.scala 17:12]
  assign io_out_bits = io_in_bits; // @[main.scala 17:12]
endmodule
```
</td>
         </tr>
    </table>
<html>
<body>

# Memory Modules

<html>
<body>
    <table border ="0">
          <tr>
            <td><b style="font-size:30px">Verilog</b></td>
            <td><b style="font-size:30px">Chisel</b></td>
            <td><b style="font-size:30px">Generated Verilog</b></td>
         </tr>
         <tr>
<td>Text comes here</td>

<td>

```scala mdoc:silent
class ReadWriteSmem extends Module {
  val width: Int = 32
  val io = IO(new Bundle {
    val enable = Input(Bool())
    val write = Input(Bool())
    val addr = Input(UInt(10.W))
    val dataIn = Input(UInt(width.W))
    val dataOut = Output(UInt(width.W))
  })
  val mem = SyncReadMem(1024, UInt(width.W))
  // Create one write port and one read port
  mem.write(io.addr, io.dataIn)
  io.dataOut := mem.read(io.addr, io.enable)
}
class ReadWriteMem extends Module {
  val width: Int = 32
  val io = IO(new Bundle {
    val enable = Input(Bool())
    val write = Input(Bool())
    val addr = Input(UInt(10.W))
    val dataIn = Input(UInt(width.W))
    val dataOut = Output(UInt(width.W))
  })
  val mem = Mem(1024, UInt(width.W))
  // Create one write port and one read port
  mem.write(io.addr, io.dataIn)
  io.dataOut := mem.read(io.addr)
}
```
```scala mdoc:invisible
ChiselStage.emitVerilog(new ReadWriteSmem)
ChiselStage.emitVerilog(new ReadWriteMem)
```
</td>

<td>

```
module ReadWriteSmem(
input         clock,
input         reset,
input         io_enable,
input         io_write,
input  [9:0]  io_addr,
input  [31:0] io_dataIn,
output [31:0] io_dataOut
);
`ifdef RANDOMIZE_MEM_INIT
reg [31:0] _RAND_0;
`endif // RANDOMIZE_MEM_INIT
`ifdef RANDOMIZE_REG_INIT
reg [31:0] _RAND_1;
reg [31:0] _RAND_2;
`endif // RANDOMIZE_REG_INIT
reg [31:0] mem [0:1023]; // @[main.scala 15:24]
wire  mem_io_dataOut_MPORT_en; // @[main.scala 15:24]
wire [9:0] mem_io_dataOut_MPORT_addr; // @[main.scala 15:24]
wire [31:0] mem_io_dataOut_MPORT_data; // @[main.scala 15:24]
wire [31:0] mem_MPORT_data; // @[main.scala 15:24]
wire [9:0] mem_MPORT_addr; // @[main.scala 15:24]
wire  mem_MPORT_mask; // @[main.scala 15:24]
wire  mem_MPORT_en; // @[main.scala 15:24]
reg  mem_io_dataOut_MPORT_en_pipe_0;
reg [9:0] mem_io_dataOut_MPORT_addr_pipe_0;
assign mem_io_dataOut_MPORT_en = mem_io_dataOut_MPORT_en_pipe_0;
assign mem_io_dataOut_MPORT_addr = mem_io_dataOut_MPORT_addr_pipe_0;
assign mem_io_dataOut_MPORT_data = mem[mem_io_dataOut_MPORT_addr]; // @[main.scala 15:24]
assign mem_MPORT_data = io_dataIn;
assign mem_MPORT_addr = io_addr;
assign mem_MPORT_mask = 1'h1;
assign mem_MPORT_en = 1'h1;
assign io_dataOut = mem_io_dataOut_MPORT_data; // @[main.scala 18:14]
always @(posedge clock) begin
if (mem_MPORT_en & mem_MPORT_mask) begin
mem[mem_MPORT_addr] <= mem_MPORT_data; // @[main.scala 15:24]
end
mem_io_dataOut_MPORT_en_pipe_0 <= io_enable;
if (io_enable) begin
mem_io_dataOut_MPORT_addr_pipe_0 <= io_addr;
end
end
// Register and memory initialization
`ifdef RANDOMIZE_GARBAGE_ASSIGN
`define RANDOMIZE
`endif
`ifdef RANDOMIZE_INVALID_ASSIGN
`define RANDOMIZE
`endif
`ifdef RANDOMIZE_REG_INIT
`define RANDOMIZE
`endif
`ifdef RANDOMIZE_MEM_INIT
`define RANDOMIZE
`endif
`ifndef RANDOM
`define RANDOM $random
`endif
`ifdef RANDOMIZE_MEM_INIT
integer initvar;
`endif
`ifndef SYNTHESIS
`ifdef FIRRTL_BEFORE_INITIAL
`FIRRTL_BEFORE_INITIAL
`endif
initial begin
`ifdef RANDOMIZE
`ifdef INIT_RANDOM
`INIT_RANDOM
`endif
`ifndef VERILATOR
`ifdef RANDOMIZE_DELAY
#`RANDOMIZE_DELAY begin end
`else
#0.002 begin end
`endif
`endif
`ifdef RANDOMIZE_MEM_INIT
_RAND_0 = {1{`RANDOM}};
for (initvar = 0; initvar < 1024; initvar = initvar+1)
mem[initvar] = _RAND_0[31:0];
`endif // RANDOMIZE_MEM_INIT
`ifdef RANDOMIZE_REG_INIT
_RAND_1 = {1{`RANDOM}};
mem_io_dataOut_MPORT_en_pipe_0 = _RAND_1[0:0];
_RAND_2 = {1{`RANDOM}};
mem_io_dataOut_MPORT_addr_pipe_0 = _RAND_2[9:0];
`endif // RANDOMIZE_REG_INIT
`endif // RANDOMIZE
end // initial
`ifdef FIRRTL_AFTER_INITIAL
`FIRRTL_AFTER_INITIAL
`endif
`endif // SYNTHESIS
endmodule
module ReadWriteMem(
input         clock,
input         reset,
input         io_enable,
input         io_write,
input  [9:0]  io_addr,
input  [31:0] io_dataIn,
output [31:0] io_dataOut
);
`ifdef RANDOMIZE_MEM_INIT
reg [31:0] _RAND_0;
`endif // RANDOMIZE_MEM_INIT
reg [31:0] mem [0:1023]; // @[main.scala 32:16]
wire  mem_io_dataOut_MPORT_en; // @[main.scala 32:16]
wire [9:0] mem_io_dataOut_MPORT_addr; // @[main.scala 32:16]
wire [31:0] mem_io_dataOut_MPORT_data; // @[main.scala 32:16]
wire [31:0] mem_MPORT_data; // @[main.scala 32:16]
wire [9:0] mem_MPORT_addr; // @[main.scala 32:16]
wire  mem_MPORT_mask; // @[main.scala 32:16]
wire  mem_MPORT_en; // @[main.scala 32:16]
assign mem_io_dataOut_MPORT_en = 1'h1;
assign mem_io_dataOut_MPORT_addr = io_addr;
assign mem_io_dataOut_MPORT_data = mem[mem_io_dataOut_MPORT_addr]; // @[main.scala 32:16]
assign mem_MPORT_data = io_dataIn;
assign mem_MPORT_addr = io_addr;
assign mem_MPORT_mask = 1'h1;
assign mem_MPORT_en = 1'h1;
assign io_dataOut = mem_io_dataOut_MPORT_data; // @[main.scala 35:14]
always @(posedge clock) begin
if (mem_MPORT_en & mem_MPORT_mask) begin
mem[mem_MPORT_addr] <= mem_MPORT_data; // @[main.scala 32:16]
end
end
// Register and memory initialization
`ifdef RANDOMIZE_GARBAGE_ASSIGN
`define RANDOMIZE
`endif
`ifdef RANDOMIZE_INVALID_ASSIGN
`define RANDOMIZE
`endif
`ifdef RANDOMIZE_REG_INIT
`define RANDOMIZE
`endif
`ifdef RANDOMIZE_MEM_INIT
`define RANDOMIZE
`endif
`ifndef RANDOM
`define RANDOM $random
`endif
`ifdef RANDOMIZE_MEM_INIT
integer initvar;
`endif
`ifndef SYNTHESIS
`ifdef FIRRTL_BEFORE_INITIAL
`FIRRTL_BEFORE_INITIAL
`endif
initial begin
`ifdef RANDOMIZE
`ifdef INIT_RANDOM
`INIT_RANDOM
`endif
`ifndef VERILATOR
`ifdef RANDOMIZE_DELAY
#`RANDOMIZE_DELAY begin end
`else
#0.002 begin end
`endif
`endif
`ifdef RANDOMIZE_MEM_INIT
_RAND_0 = {1{`RANDOM}};
for (initvar = 0; initvar < 1024; initvar = initvar+1)
mem[initvar] = _RAND_0[31:0];
`endif // RANDOMIZE_MEM_INIT
`endif // RANDOMIZE
end // initial
`ifdef FIRRTL_AFTER_INITIAL
`FIRRTL_AFTER_INITIAL
`endif
`endif // SYNTHESIS
endmodule
```
</td>
         </tr>
    </table>
<html>
<body>

# Operators

<html>
<body>
    <table border ="0">
          <tr>
            <td><b style="font-size:30px">Verilog</b></td>
            <td><b style="font-size:30px">Chisel</b></td>
            <td><b style="font-size:30px">Generated Verilog</b></td>
         </tr>
         <tr>
<td>text here
</td>
<td>

```scala mdoc:silent
class OperatorExampleModule extends Module {

  val x, y = IO(Input(UInt(32.W)))

  val a, b, c, d , e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, xx, yy,  z, zz, zzz, zzzz, zzzzz = IO(Output(UInt(32.W)))
  val aa, bb= IO(Output(Bool()))
  
  a := x + y
  b := x - y 
  c := x % y
  d := x * y
  e := x / y
  f := x % y
  g := x +% y
  h := x -% y
  i := x -& y
  j := x === y
  k := DontCare//x != y This one is a bit nonsensical -- this is a Scala comparison. This might be a bug in the cheat sheet.
  l := x =/= y 
  m := x & y
  n := x | y
  o := x ^ y
  p :=  ~x
  q := !x
  r := x(0) && y(0) // Must be a Bool type
  s := x(0) || y(0) // Must be a Bool type
  t := Cat(x, y)
  u := Mux(c(0), x, y)  // Selector must be a Bool type
  v := x >> y(2, 0)
  w := x << y(2, 0) // Can't do a 32-bit shift operator
  xx := x > y
  yy := x >= y
  z := x <= y
  zz := x(1) 
  zzz := x(1, 0) 
  zzzz:= Fill(3,x)
  zzzzz := x.andR
  aa := x.orR
  bb := x.xorR
} 
```
```scala mdoc:invisible
ChiselStage.emitVerilog(new OperatorExampleModule)
```
</td>
<td>

```
module OperatorExampleModule(
  input         clock,
  input         reset,
  input  [31:0] x,
  input  [31:0] y,
  output [31:0] a,
  output [31:0] b,
  output [31:0] c,
  output [31:0] d,
  output [31:0] e,
  output [31:0] f,
  output [31:0] g,
  output [31:0] h,
  output [31:0] i,
  output [31:0] j,
  output [31:0] k,
  output [31:0] l,
  output [31:0] m,
  output [31:0] n,
  output [31:0] o,
  output [31:0] p,
  output [31:0] q,
  output [31:0] r,
  output [31:0] s,
  output [31:0] t,
  output [31:0] u,
  output [31:0] v,
  output [31:0] w,
  output [31:0] xx,
  output [31:0] yy,
  output [31:0] z,
  output [31:0] zz,
  output [31:0] zzz,
  output [31:0] zzzz,
  output [31:0] zzzzz,
  output        aa,
  output        bb
);
  wire [32:0] _b_T = x - y; // @[main.scala 16:10]
  wire [63:0] _d_T = x * y; // @[main.scala 18:10]
  wire  _r_T_2 = x[0] & y[0]; // @[main.scala 32:13]
  wire  _s_T_2 = x[0] | y[0]; // @[main.scala 33:13]
  wire [63:0] _t_T = {x,y}; // @[Cat.scala 30:58]
  wire [38:0] _GEN_0 = {{7'd0}, x}; // @[main.scala 37:10]
  wire [38:0] _w_T_1 = _GEN_0 << y[2:0]; // @[main.scala 37:10]
  wire [95:0] _zzzz_T_1 = {x,x,x}; // @[Cat.scala 30:58]
  wire [31:0] _GEN_1 = x % y; // @[main.scala 17:10]
  wire [31:0] _GEN_2 = x % y; // @[main.scala 20:10]
  assign a = x + y; // @[main.scala 15:10]
  assign b = x - y; // @[main.scala 16:10]
  assign c = _GEN_1[31:0]; // @[main.scala 17:10]
  assign d = _d_T[31:0]; // @[main.scala 18:5]
  assign e = x / y; // @[main.scala 19:10]
  assign f = _GEN_2[31:0]; // @[main.scala 20:10]
  assign g = x + y; // @[main.scala 21:10]
  assign h = x - y; // @[main.scala 22:10]
  assign i = _b_T[31:0]; // @[main.scala 23:5]
  assign j = {{31'd0}, x == y}; // @[main.scala 24:5]
  assign k = 32'h0;
  assign l = {{31'd0}, x != y}; // @[main.scala 26:5]
  assign m = x & y; // @[main.scala 27:10]
  assign n = x | y; // @[main.scala 28:10]
  assign o = x ^ y; // @[main.scala 29:10]
  assign p = ~x; // @[main.scala 30:9]
  assign q = {{31'd0}, x == 32'h0}; // @[main.scala 31:5]
  assign r = {{31'd0}, _r_T_2}; // @[main.scala 32:5]
  assign s = {{31'd0}, _s_T_2}; // @[main.scala 33:5]
  assign t = _t_T[31:0]; // @[main.scala 34:5]
  assign u = c[0] ? x : y; // @[main.scala 35:11]
  assign v = x >> y[2:0]; // @[main.scala 36:10]
  assign w = _w_T_1[31:0]; // @[main.scala 37:5]
  assign xx = {{31'd0}, x > y}; // @[main.scala 38:6]
  assign yy = {{31'd0}, x >= y}; // @[main.scala 39:6]
  assign z = {{31'd0}, x <= y}; // @[main.scala 40:5]
  assign zz = {{31'd0}, x[1]}; // @[main.scala 41:6]
  assign zzz = {{30'd0}, x[1:0]}; // @[main.scala 42:7]
  assign zzzz = _zzzz_T_1[31:0]; // @[main.scala 43:7]
  assign zzzzz = {{31'd0}, &x}; // @[main.scala 44:9]
  assign aa = |x; // @[main.scala 45:11]
  assign bb = ^x; // @[main.scala 46:11]
endmodule
```
</td>
         </tr>
    </table>
<html>
<body>
