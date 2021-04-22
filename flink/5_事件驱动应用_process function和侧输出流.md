## 事件驱动应用

### Process Functions

> 我们调用`map()`算子时需要传入`MapFunction`。那我们要调用`process()`算子时需要传入`ProcessFunction`
>
> `map()`算子的`MapFunction`可以对事件进行处理（对事件进行映射），`ProcessFunction`同样可以对事件进行处理，而且可以使用`Timer`和`State`。因此`ProcessFunction`是Flink 创建`Event-driven Applications`的基础。
>
> `ProcessFunction`和 `RichFlatMapFunction` 十分相似， 但是增加了 `Timer`。





### 旁路输出

> 