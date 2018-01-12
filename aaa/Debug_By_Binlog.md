   线上报了个更新数据库的错：java.lang.IllegalStateException: 账户余额更新失败。其他细节没啥关系就不贴了，而且是钱的业务有些不能贴。

   翻了下代码，发现异常是因为更新返回了0行，然后代码里抛出来的。

```markdown
  
        if (resultFlag < 1) {
            throw new IllegalStateException("账户余额更新失败 [tradeSerial = " + tradeSerial + "]");
        }
        
```

   看了下逻辑和参数似乎不应该，不过这SQL的判断条件中有乐观锁的写法，就是where version=...。

   那应该就是这原因了，但是需要证明，于是去线上把binlog拿了下来，然后转成人能看的方式：

```markdown

        mysqlbinlog --base64-output=decode-rows -v  mysql-bin.000952 > aaa.txt
 
 ```
 
    复制到我本地：
  
```markdown
  
        scp aaa.txt aaa@192.168.3.2:/home/aaa/Code

``` 

    打开一看，发现果然有另外一个业务更新了同一条数据，导致version变化，于是更新语句执行失败。

-----

         #180111 20:38:03 server id 2792231848  end_log_pos 6551452 CRC32 0x3fb480f4 	Update_rows: table id 2163 flags: STMT_END_F
            ### UPDATE .....
            ### WHERE
            ........
            ### SET
            ........

-----

微信公众号：

![Image](/ppp/20170902204445.jpg)

