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

  这两个单元测试单独执行什么问题都没有，但是一起执行，上面那个就报错。我各种想不通，在各种调整和加out print之后发现，如果改
