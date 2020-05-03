# java7
## try-with-resources
-  使用范围：所有实现了新增AutoCloseable接口的类
-  应用场景：文件操作、JDBC连接、httpclient工具等所有实现了Closeable（该接口实现AutoCloseable）、AutoCloseable接口的类（比如redis、kafka、消息中间件等一些需要连接后需要关闭的类） 
  
旧代码-示例
```
    	
        InputStream in = null;
		OutputStream out = null;
		try{
			in = new FileInputStream(new File(""));
		    out = new FileOutputStream(new File(""));
		} catch (IOException|BusinessException e) {
			
		} finally {
			try {
				if(in!=null) {
					in.close();
				}
			} catch (Exception e) {
				
			}
			try {
				if(out!=null) {
					out.close();
				}
			} catch (Exception e) {
				
			}
		}	
```
新代码
```javacode
        //file
        try(InputStream in = new FileInputStream(new File(""));
    			OutputStream out = new FileOutputStream(new File(""))) {
    			
    	} catch (IOException|BusinessException e) {
    			
    	}
    	//httpclient
    	 try ( CloseableHttpClient httpClient= HttpClients.createDefault()){
    	 }catch (Exception e) {
    			
    	}
    	//jdbc
	    try (Connection conn=ds.getConnection();
				 PreparedStatement pstmt = conn.prepareStatement(sql);
				 ResultSet rs =  pstmt.executeQuery();
		) catch (Exception e) {
    			
    	}
```
## 数字加入下划线支持
-  应用场景：超长或超大数用作分隔便于快速识别，下划线无其他含义
  
代码-示例
```
        short i= 5_0_0;
		int j = 500_00_5;
		double k = 5454.454_2;
		float l = 5454.454_2F;
```

## switch支持字符串
-  应用场景：多重if else if判断时，增加代码可读性


##  fork/join
-  新增抽象类ForkJoinTask和各种继承ForkJoinTask的类来达到获得不同的子任务结果例如RecursiveAction 为无返回值子类，RecursiveTask<T>为有返回值子类
-  原理：把一个大任务分割成多个小任务执行并把结果合并
-  应用场景：需要处理大数据量数据时，例如批量短信发送，可分割处理；递归调用。

![image](https://images2018.cnblogs.com/blog/905730/201807/905730-20180711145448299-68610441.png)

代码示例
```
/***
 *
 * @ClassName:CountTask
 * @author:hujunhua
 * @date:2019年10月14日 下午5:16:10
 *
 */
public class TestTask extends RecursiveTask<Long> {

	private static final Logger log = LoggerFactory.getLogger(TestTask.class);

	/**
	 * @Fields serialVersionUID:(描述)
	 */
	private static final long serialVersionUID = 1L;

	private static final int EXEC_NUM = 10; // 每次处理条数
	
	private static final long EXEC_LONG_TIME = 0;//毫秒

	private List<Long> numList;

	public TestTask(List<Long> numList) {
		this.numList = numList;
	}

	@Override
	protected Long compute() {
		long sum = 0;
		//log.info("" + numList.size());
		if (numList.size() <= EXEC_NUM) {
			for (int i = 0; i < numList.size(); i++) {
				sum += numList.get(i);
				try {
					Thread.sleep(EXEC_LONG_TIME);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		} else {
			TestTask leftTask = new TestTask(numList.subList(0, EXEC_NUM));
			TestTask rightTask = new TestTask(numList.subList(EXEC_NUM, numList.size()));
			// 执行子任务
			leftTask.fork();
			rightTask.fork();
			// 等待子任务执行完，并得到其结果
			Long rightResult = rightTask.join();
			Long leftResult = leftTask.join();
			// 合并子任务
			sum = leftResult + rightResult;
		}
		return sum;
	}

	public static void main(String[] args) throws ExecutionException, InterruptedException {
		List<Long> numList = new ArrayList<Long>();
		for (int i = 1; i <= 1000; i++) {
			numList.add(i+0l);	
		}
		long s = System.currentTimeMillis();
		long num = 0;
		for (int i = 0; i < numList.size(); i++) {
			num += numList.get(i);
			Thread.sleep(EXEC_LONG_TIME);
		}
		System.out.println("单线程"+num+"用时："+(System.currentTimeMillis()-s));
		s = System.currentTimeMillis();
		ForkJoinPool forkJoinPool = new ForkJoinPool(64);//默认线程池大小为8，可指定为2的指数 大小
		TestTask testTask = new TestTask(numList);
		ForkJoinTask<Long> forkJoinTask = forkJoinPool.submit(testTask);
		System.out.println(forkJoinTask.get()+"用时："+(System.currentTimeMillis()-s));

	}
}

```


  