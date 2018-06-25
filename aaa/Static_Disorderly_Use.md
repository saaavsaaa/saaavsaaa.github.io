  被一个忘了什么原因写上去的static坑了两个多小时，记之以自勉。
  
  问题开始于两个单元测试：
  
-----

    @Test
    public void testStart() throws IOException, InterruptedException {
        IClient testClient = new TestClient(new ClientContext(TestSupport.SERVERS, TestSupport.SESSION_TIMEOUT));
        boolean result = testClient.start(10000, TimeUnit.MILLISECONDS);
        assert result;
        System.out.println(result);
        testClient.close();
    }
    
    @Test
    public void testNotStart() throws IOException, InterruptedException {
        TestClient testClient = new TestClient(new ClientContext(TestSupport.SERVERS, TestSupport.SESSION_TIMEOUT));
        boolean result = testClient.start(100, TimeUnit.MILLISECONDS);
        assert !result;
        System.out.println(result);
        testClient.close();
    }

-----

  这两个单元测试在 [关于SS注册中心ZK客户端的封装](https://saaavsaaa.github.io/aaa/S_S_ZK_Registry_Center.html) 关于Holder的start方法使用await带时间参数的实现处提到了。zk的连接使用了CountDownLatch，在实例化zk之后await，在收到已连接的事件后countDown。这里用了await(long timeout, TimeUnit unit)
  
  这两个单元测试单独执行什么问题都没有，但是一起执行，上面那个就报错。我各种想不通，在各种调整和加out print之后发现，如果改变单元测试执行顺序，这两个就都能过，那就基本确定了两个单元测试看似没有关系，但实际上可以互相影响，
