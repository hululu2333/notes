### flink自定义聚合函数

***

> 当我们使用Flink Table/SQL API时，可以通过sql语句对数据进行一些操作。Api中自带了一些常见的聚合函数，但有时这些自带的并不能满足我们的需求，我们需要自定义符合业务需求的聚合函数。



#### 简单示例

> 先从一个实际案例入手：设备随时上报状态，现在需要求出设备的当前最新状态。分析：设备上报状态会产生多条数据，现在只需要最新的状态数据即可，很明显这是多对一的聚合类型的操作，聚合逻辑是每次保留设备的最新状态与时间，下次设备上报数据时间与保留的数据时间进行比较，如果比其大则更新。

``` java
public class LatestTimeUdfextendsAggregateFunction<Integer,TimeAndStatus>{
	@Override
    publicTimeAndStatus createAccumulator(){
		return new TimeAndStatus();
	}
    
	public void accumulate(TimeAndStatus acc,Integer status,Long time){
		if(time > acc.getTimes()){
        	acc.setStatus(status);
            acc.setTimes(time);
		}
	}
    
	@Override
    publicInteger getValue(TimeAndStatus timeAndStatus){
		return timeAndStatus.getStatus();
	}
}
```

> 在Flink Table/SQL Api中自定义聚合函数需要继承AggregateFunction<T,ACC>， 其中T表示自定义函数返回的结果类型，在这里返回的是Integer，表示状态标识。ACC表示聚合的中间结果类型，TimeAndStatus存放时间与状态数据，
>
> 该函数有两个指定该类型的方法，getAccumulatorType与getResultType返回的都是TypeInformation类型，如果我们的T或者ACC是复杂类型Flink不能自动抽取的则需要手动指定。其每个方法定义如下：

> createAccumulator 表示创建一个中间结果数据，由于是以设备为维度那么对于每一个设备都会调用一次该方法；

> accumulate 表示将流入的数据聚合到createAccumulator创建的中间结果数据中，第一个参数表示的是ACC类型的中间结果数据，其他的表示自定义函数的入参，该方法可以接受不同类型、个数的入参，也就是该方法可以被重载，Flink会自动根据类型提取找到合适的方法。在这里接受的是Integer类型的设备状态与long类型的时间戳，处理逻辑就是与中间结果数据时间进行比较，如果比其大则将流入的时间与设备状态更新到中间结果中。另外在做一点补充accumulate的调用是相同维度的调用，即acc每次都是该维度的中间结果数据，入参也是该维度的数据；

> getValue 表示一次返回的结果，结果数据从acc中获取，这个方法在accumulate之后被调用。

> 对于自定义聚合函数来说至少需要createAccumulator、accumulate、getValue这三个方法，并且这三个方法是public 、not static的类型。接下来就可以直接注册然后使用：

``` scala
// scala示例，仅供参考
tabEnv.registerFunction("latestTimeUdf",newLatestTimeUdf())
val dw=tabEnv.sqlQuery(
"""select latestTimeUdf(status,times) st,devId from tbl1 group by devId
      """.stripMargin)
```

 

#### 撤回定义
> 撤回机制对于Flink来说是一个很重要的特性，在Flink SQL中可撤回机制解密中详细分析了撤回的实现，其中retract是一个不可或缺的环节，其表示具体的回撤操作，对于自定义聚合函数，如果其接受到的是撤回流那么就必须实现该方法，看下其定义：

publicvoid retract(ACC accumulator,[user defined inputs])
}
accumulator表示对应维度的中间结果数据，流入的数据表示需要撤回的数据，该方法表示将需要撤回的数据从中间结果中去除掉，Flink中默认实现了一些撤回的函数，例如SumWithRetractAggFunction：

def retract(acc:SumWithRetractAccumulator[T], value:Any):Unit={
if(value !=null){
      val v = value.asInstanceOf[T]
      acc.f0 = numeric.minus(acc.f0, v)
      acc.f1 -=1
}
}
表示求和类的撤回，将需要撤回的value从acc中减掉。
在AggregateFunction 还提供了另外两个函数merge与resetAccumulator，merge 用在session window group或者批处理中，需要做一个合并的过程，resetAccumulator 用在批处理中，表示对中间结果的重置。

在源码中的调用位置
由于是聚合类的操作，仍然以GroupAggProcessFunction 来分析，在这里会调用自定义函数，但是只能是在非窗口的聚合中，通过processElement方法看下其调用流程

中间结果acc是存储在状态中的，如果得到的状态为空，那么就会调用createAccumulator 方法

var accumulators = state.value()
var inputCnt = cntState.value()
if(null== accumulators){
if(!inputC.change){
return
}
      firstRow =true
      accumulators =function.createAccumulators()//初始化
}else{
      firstRow =false
}
state是ValueState类型的，acc就存储在这里面，说明acc是容错并且具有一致性的。

如果流入的数据是Insert类型就会调用accumulate方法，如果是Retract就调用retract方法，并且会调用getValue获取当前的结果数据

if(inputC.change){
      inputCnt +=1
// accumulate input
function.accumulate(accumulators, input)
function.setAggregationResults(accumulators, newRow.row)//会调用getValue
}else{
      inputCnt -=1
// retract input
function.retract(accumulators, input)
function.setAggregationResults(accumulators, newRow.row)//会调用getValue
}



#### 总结

> 自定义聚合函数是一个增量聚合的过程，中间结果保存在状态中，能够保证其容错与一致性语义。用户自定义聚合函数继承AggregateFunction即可，至少实现createAccumulator 、accumulate 、getValue这三个方法，其他方法都是可选的。