### hbase的compact

***

> compact的意思是压实、压紧

> 在向hbase中put数据时，数据会先放进内存中。当数据占用了一定内存时，会进行flush。每flush一次，会生成一个HFile文件。（如果内存中的数据来自不同的region或不同的列族，因该会生成多个HFile吧？）
>
> 到了后面，同一个Store中会有大量的HFile。同一个字段的不同版本可能会在不同的HFile中，这样不利于检索，所以我们要将他们合并。

> compact分为两种类型，俗称小合并和大合并。



#### MinorCompaction

> MinorCompaction会将临近的若干个较小的HFile合并为一个较大的HFile。但不会清理过 期和删除的数据。
>
> 所以MinorCompaction的作用仅仅是减少同一个0store中的HFile个数



#### MajorCompaction

> MajorCompaction是将一个Store中的所有HFile合并为一个大HFile，并且会清理过期和删除的数据。
>
> 过期的数据指的是同一个字段中，时间戳较小的那些cell。删除的数据指的是类型为delete的那些数据
>
> 在flush阶段可能会删除时间戳较小的那些cell，但类型为delete的那些cell一定是在大合并中被删的。



#### 如何Compact

> 我们可以在hbase的客户端通过compact或major_compact命令进行手动合并。当一个Store中的HFile大于三时，无论执行哪个命令，都是执行大合并
>
> 也可以通过修改配置文件，让HRegion自动合并

