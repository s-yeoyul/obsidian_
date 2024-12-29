### 1.
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

