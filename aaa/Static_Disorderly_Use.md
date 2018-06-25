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

  这两个单元测试在 [关于SS注册中心ZK客户端的封装](https://saaavsaaa.github.io/aaa/S_S_ZK_Registry_Center.html) 关于Holder的start方法使用await带时间参数的实现处提到了。zk的连接使用了CountDownLatch，在实例化zk之后await，在收到已连接的事件后countDown。这里用了await(long timeout, TimeUnit unit)，测试方法就是在zk回调事件的监听处休眠。
  
  这两个单元测试单独执行什么问题都没有，但是一起执行，上面那个就报错。我各种想不通，在各种调整和加out print之后发现，如果改变单元测试执行顺序，这两个就都能过，那就基本确定了两个单元测试看似没有关系，但实际上可以互相影响，原本是公用的TestClient，公用的close，都改成独立的了，那就问题只有可能是回调时监听的线程了，于是就在回调处输出了线程id，发现果然是这里有问题。
  
  在testNotStart先执行的情况下，回调线程休眠时间比等待时间长，于是连接未成功，assert成立。这个时候testStart单元测试开始执行，由于等待时间比回调中休眠时间长，所以应该countDown后返回成功的，但是上一个回调线程的休眠在这时候结束了，把countDown给执行了，但是休眠还没有结束，连接没建立成功，所以单元测试执行失败了。那为什么先执行的单元测试的线程会影响后执行的单元测试呢，原因就很明显了，CountDownLatch实例一定是static的，翻过去一看，果然。记录下来提醒一下自己，细节决定成败。

-----

微信公众号：

![Image](/ppp/20170902204445.jpg)
