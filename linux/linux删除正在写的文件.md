http://www.forthxu.com/blog/article/40.html

多进程环境下，打开同一个文件，进行读写操作过程中，如果其中一个进程删除这个文件，那么，另外正在读写这个文件会发生什么呢？

- 因为文件被删除了，正在读写的进程发生异常？
- 正在读写的进程仍然正常读写，好像没有发现发生了什么？

Linux 是通过 link 的数量来控制文件删除，只有当一个文件不存在任何 link 的时候，这个文件才会被删除。

每个文件都有 2 个 link 计数器 —— i_count 和 i_nlink。i_count 的意义是当前使用者的数量，i_nlink 的意义是介质连接的数量；或者可以理解为 i_count 是内存引用计数器，i_nlink 是硬盘引用计数器。再换句话说，当文件被某个进程引用时，i_count 就会增加；当创建文件的硬连接的时候，i_nlink 就会增加。

对于 rm 而言，就是减少 i_nlink。这里就出现一个问题，如果一个文件正在被某个进程调用，而用户却执行 rm 操作把文件删除了，会出现什么结果呢？

当用户执行 rm 操作后，ls 或者其他文件管理命令不再能够找到这个文件，但是进程却依然在继续正常执行，依然能够从文件中正确的读取内容。这是因为，`rm` 操作只是将 i_nlink 置为 0 了；由于文件被进程引用的缘故，i_count 不为 0，所以系统没有真正删除这个文件。i_nlink 是文件删除的充分条件，而 i_count 才是文件删除的必要条件。

基于以上只是，大家猜一下，如果在一个进程在打开文件写日志的时候，手动或者另外一个进程将这个日志删除，会发生什么情况？

是的，数据库并没有停掉。虽然日志文件被删除了，但是有一个进程已经打开了那个文件，所以向那个文件中的写操作仍然会成功，数据仍然会提交。

下面，告诉大家如何恢复那个删除的文件。

例如，你删除了tcpdump.log，执行lsof | grep tcpdump.log，你应该能看到这样的输出：

tcpdump 2864 tcpdump 4w REG 253,0 0 671457 /root/tcpdump.log (deleted)

然后：

cp /proc/2864/fd/4 /root/tcpdump.log