### 1. `Assign`
`assign` 은 **continuous assignment**라고 불리며, signal의 값이 wire를 통해 전달된다. rvalue의 값이 변할 때 마다 assignment가 연속적으로 진행되기 때문에 이러한 명칭이 붙었다.
즉 **combinational logic**으로 이해하면 된다...!!
만약 `assign out = a + b;` 로 선언하면 이는 a와 b를 Input으로 받는 Adder를 구현한 것이라 생각하면 된다.
> A _continuous_ assignment assigns the right side to the left side _continuously_, so any change to the RHS is immediately seen in the LHS.

```verilog
module top_module( output one );
    assign one = 1'b1;
endmodule
```

`1'b1` 은 1 bit binary 1을 의미한다.
### 2. Wire
![](https://i.imgur.com/oAcIWmo.png)

```verilog
module top_module( input in, output out );
	assign out = in;
endmodule
```

### 3. Multiple wires
![](https://i.imgur.com/n10gMMn.png)

`assign` 의 순서는 상관없다. **assign은 connection을 의미하는 것이지 값을 copy하는 행위를 의미하는 것이 아니기 때문!** 

```verilog
module top_module( 
    input a,b,c,
    output w,x,y,z );
	assign w = a;
    assign x = b;
    assign y = b;
    assign z = c;
endmodule
```

### 4. Basic gates
#### Inverter
![](https://i.imgur.com/9u6tdDt.png)

```verilog
module top_module( input in, output out );
	assign out = ~in;
endmodule
```

#### AND
![](https://i.imgur.com/Eg8nZNs.png)
```verilog
module top_module( 
    input a, 
    input b, 
    output out );
	assign out = a & b;
endmodule
```
#### NOR
![](https://i.imgur.com/ptG5E0E.png)

```verilog
module top_module( 
    input a, 
    input b, 
    output out );
    assign out = ~(a | b);
endmodule
```

#### XNOR
`^` : bit-wise XOR operation

![](https://i.imgur.com/6IwIjZP.png)

```verilog
module top_module( 
    input a, 
    input b, 
    output out );
    assign out = ~(a ^ b);
endmodule
```

### 5. Declaring wires
![](https://i.imgur.com/JNr6FRW.png)

```verilog
module top_module(
    input a,
    input b,
    input c,
    input d,
    output out,
    output out_n   ); 
	
    wire c1, c2, c3;
    
    assign c1 = a & b;
    assign c2 = c & d;
    assign c3 = c1 | c2;
    
    assign out = c3;
    assign out_n = ~c3;
    
endmodule
```

### 6. Vector
wire에 n bit 신호를 담아서 연결하고 싶을 수도 있다.
`wire [99:0] vec;` 으로 선언하면 0번째 부터 99번째 index를 갖는 wire 벡터가 선언된다.
instance의 이름 앞에 dimension이 먼저 온다.
그리고 endianness를 통일해야 한다. 즉 한 wire/reg를 \[last:first](little-endian)로 선언했으면 일관성을 맞춰야 한다는 이야기.

![](https://i.imgur.com/cteo60Q.png)
![](https://i.imgur.com/fuThYMR.png)

```verilog
module top_module ( 
    input wire [2:0] vec,
    output wire [2:0] outv,
    output wire o2,
    output wire o1,
    output wire o0  ); // Module body starts after module declaration

    assign o0 = vec[0];
    assign o1 = vec[1];
    assign o2 = vec[2];
    assign outv = vec;
endmodule
```

만약 명시적으로(=explicitly) 타입을 선언하지 않은 변수가 있다면 기본적으로 1 bit wire로 가정한다.
```verilog
wire [2:0] a, c; // Two vectors
assign a = 3'b101; // a = 101
assign b = a; // b = 1, implicitly-created wire
assign c = b; // c = 001 <-- bug, because a != c
```

그리고 앞서 비트 범위를 변수명 앞에 선언하는 것을 기억할 것이다. 이를 **packed**라고 부른다. 즉 n개의 비트가 묶여있다는 뜻이다. 신호 자체가 n비트인 경우 packed vector를 이용하면 되겠지만, 메모리의 경우에는?
n개의 독립적인 메모리가 필요한 경우에도 그렇게 선언을 할 필요는 없을 것이다. 이럴때는 **unpacked**로 선언하면 되고, 비트 범위가 변수명 뒤에 붙는다.

```verilog
reg [7:0] mem [255:0];
reg mem2 [28:0];
```

첫 번째 line은 256칸 메모리를 unpacked로 선언한 것이고, 각 칸은 8bit 이다(packed).
두번째는 29bit 메모리를 선언한 것이고 각 칸은 1 bit이다. 

부분적으로 벡터를 선택할 수도 있다.
32bit vector의 endianness를 바꾸는 모듈은 다음과 같다.
```verilog
module top_module( 
    input [31:0] in,
    output [31:0] out );//

    assign out[7:0] = in[31:24];
    assign out[15:8] = in[23:16];
    assign out[23:16] = in[15:8];
    assign out[31:24] = in[7:0];

endmodule
```
