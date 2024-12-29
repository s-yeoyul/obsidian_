### 1.
`assign` 은 **continuous assignment**라고 불리며, signal의 값이 wire를 통해 전달된다. rvalue의 값이 변할 때 마다 assignment가 연속적으로 진행되기 때문에 이러한 명칭이 붙었다.
즉 **combinational logic**으로 이해하면 된다...!!
만약 `assign out = a + b;` 로 선언하면 이는 a와 b를 Input으로 받는 Adder를 구현한 것이라 생각하면 된다.

```verilog
module top_module( output one );
    assign one = 1;
endmodule
```

### 2.