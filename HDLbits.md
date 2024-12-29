### 1.
`assign` 은 **continuous assignment**라고 불리며, signal의 값이 wire를 통해 전달된다. rvalue의 값이 변할 때 마다 assignment가 연속적으로 진행되기 때문에 이러한 명칭이 붙었다.

```verilog
module top_module( output one );
    assign one = 1;
endmodule
```



### 2.