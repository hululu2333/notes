### 将数据上传到hdfs并压缩

***

> 主要是创建从磁盘到程序的输入流，再创建从程序到hdfs的输出流。这个输出流使用的是带压缩功能的包装流
>
> 然后将这两个流对接

``` java
/**
 * 在把文件上传到hdfs中的同时进行压缩，以节省空间并加快传输效率
 * 解决的是hdfs的储存效率和传输效率的问题
 */
public class TestCompression {
    public static void main(String[] args) throws IOException {
        Configuration conf=new Configuration();
        conf.set("fs.defaultFS","hdfs://192.168.220.50:9000");

        FileInputStream fis=new FileInputStream("D:\\long.txt");


        FileSystem fs=FileSystem.get(conf);
        FSDataOutputStream fsos=fs.create(new Path("./long.txt.gz"));

        //创建压缩编码器工厂对象
        CompressionCodecFactory ccf=new CompressionCodecFactory(conf);
        CompressionCodec cc=ccf.getCodec(new Path("./long.txt.gz"));

        //根据编码器创建输出流,这是一个包装流
        CompressionOutputStream cos=cc.createOutputStream(fsos);

        //将输入流和输出流对接
        IOUtils.copyBytes(fis,cos,1024,true);

        fsos.close();
        fs.close();

    }
}

```

