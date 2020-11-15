### Hdfs的序列化和反序列化

***

> 序列化是指将JVM中的对象转换成字节数组，这样就能将对象进行传输或者持久化保存
>
> 反序列化则是将字节数组转换为对象



#### hdfs序列化和java序列化

##### java序列化

> 实现java序列化只需要在你要传输的对象的类后面加上impliments Serializable，十分的简单。
>
> 但是java序列化**比较重量级**，它会保留这个类的所有信息，序列化后的结果占用比较大的内存，不太试合在大数据中应用
>
> **对象不可重用**，内存开销较大

***

##### hadoop序列化

> hadoop的序列化比较轻量级，而且对象可重用。
>
> hadoop的序列化和反序列化需要自己实现



#### 将一个自定义的数据类型实现hadoop序列化

``` java
/**
 * 创建一个类，并让这个类可序列化，可比较。以便后期在hdfs上传输
 * 实现WritableComparable接口
 * 解决的是对象在hdfs中传输和进行mapreduce的问题
 */
public class StudentWritable implements WritableComparable<StudentWritable> {
    private IntWritable age;
    private Text name;
    private double egs;
    private List<String> strings;

    public StudentWritable(){
        age=null;
        name=null;
        strings=null;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        StudentWritable that = (StudentWritable) o;
        return Objects.equals(age, that.age) &&
                Objects.equals(name, that.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(age, name);
    }

    public Text getName() {
        return name;
    }

    public void setName(Text name) {
        this.name.set(name.toString());
    }

    public IntWritable getAge() {
        return age;
    }

    public void setAge(IntWritable age) {
        this.age.set(age.get());
    }

    @Override
    public int compareTo(StudentWritable o) {
        int ageCom=this.age.compareTo(o.age);
        int nameCom=this.name.compareTo(o.name);
        return ageCom!=0?ageCom:nameCom;
    }

    //将属性进行序列化
    @Override
    public void write(DataOutput out) throws IOException {
        
        /**
         * age是IntWritable型，也实现了WritableComparabe接口
         * 所以age中有write方法。out是一个输出流
         * age中的write方法有一行代码，它是out.WriteInt(value)。value是age中含的int值
         * 所以hadoop序列化的底层和java中的DataOutputStream一样
         */
        age.write(out); //等同于out.writeInt(age.get())
        
        name.write(out);
        
        //DoubleWritable dw=new DoubleWritable(this.egs);
        //dw.write(out);
        out.writeDouble(egs);//可以将上面两行写成这种形式，这样写的话，反序列化逻辑也要改变
        
        IntWritable iw=new IntWritable(strings.size());
        iw.write(out);
        for(String str:strings){
            new Text(str).write(out);
        }

    }

    //将对象按进入的顺序反序列化
    @Override
    public void readFields(DataInput in) throws IOException {
        age.readFields(in);

        //DoubleWritable dw=new DoubleWritable();
        //dw.readFields(in);
        this.egs=in.readDouble(); //因为序列化的逻辑变了

        IntWritable iw=new IntWritable();
        iw.readFields(in);
        int size=iw.get();

        strings.clear();
        for(int i=0;i<size;i++){
            Text t=new Text();
            t.readFields(in);
            strings.add(t.toString());
        }
    }


}
```



#### 将自定义数据类型的对象以序列文件的形式上传到hdfs

``` java
/**
 * 本来是想用ObjectOutputStream包装上ByteArrayOutputStream将StudentWritable对象以字节数组的形
 * 式输出到内存中。然后用FSDataOutputStream将字节数组输出到hdfs中。但ObjectOutputStream是java的
 * 流,需要你的类实现Serializable接口
 * 我们要把一个实现了Writable接口的对象序列化并上传至hdfs中，需要使用SequenceFile.Writer这个流
 */
public class Test {
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        conf.set("fs.defaultFS", "hdfs://hadoop01:9000");

        //指定序列文件的的储存目录
        SequenceFile.Writer.Option path=
                SequenceFile.Writer.file(new Path("./test.seq"));

        //指定序列文件key的数据类型
        SequenceFile.Writer.Option key=
                SequenceFile.Writer.keyClass(IntWritable.class);

        //指定序列文件value的数据类型
        SequenceFile.Writer.Option value=
                SequenceFile.Writer.valueClass(StudentWritable.class);

        //创建Writer对象
        SequenceFile.Writer writer=SequenceFile.createWriter(conf,path,key,value);

        System.out.println(writer);
        IntWritable iw = new IntWritable(1);
        StudentWritable sw = new StudentWritable();
        writer.append(new IntWritable(1),new StudentWritable());
        writer.hflush();
        writer.close();

    }
}
```

