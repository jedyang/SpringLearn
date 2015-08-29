#Spring的事务管理

###数据库事务基础知识
- 数据库一般采用重执行日志来保证事务的原子性，一致性和持久性。  
采用数据库锁机制来保证事务的隔离性。

- 数据并发问题：
	1. 脏读。A事务读取并使用了B事务还没提交的更改数据。（oracle数据库不会出现这个问题）
	2. 不可重复读。A读取数据，B提交更改，然后A事务读取了B事务已经提交的更改数据，导致A事务前后两次读到的数据不一样。
	3. 幻象读。A先统计数据。B提交新增数据。A再统计数据。导致A事务两次统计结果不一样。   
	注意，不可重复读是修改数据，加行级锁避免。幻象读是新增数据，加表级锁避免。
	4. 第一类丢失更新。  
	A事务撤销时，将已经提交的B事务的更新数据覆盖回去了。  
	5. 第二类丢失更新。  
	A事务直接覆盖掉B事务的提交数据。

- JDBC事务支持

			Connection conn = null;
		
		        try
		        {
		            // 获取数据连接
		            conn = DriverManager.getConnection("");
		            // 关闭自动提交
		            conn.setAutoCommit(false);
		            // 设置事务隔离级别
		            conn.setTransactionIsolation(Connection.TRANSACTION_SERIALIZABLE);
		            Statement stmt = conn.createStatement();
		            int i = stmt.executeUpdate("update sql1");
		
		            Savepoint point1 = conn.setSavepoint("point1");
		
		            i = stmt.executeUpdate("update sql2");
		
		            conn.rollback(point1); // 可以回滚到一个savepoint
		
		            conn.commit();
		        }
		        catch (SQLException e)
		        {
		            // TODO Auto-generated catch block
		            e.printStackTrace();
		            try
		            {
		                conn.rollback();
		            }
		            catch (SQLException e1)
		            {
		                // TODO Auto-generated catch block
		                e1.printStackTrace();
		            }
		        }



### ThreadLocal
- spring使用ThreadLocal来解决线程安全问题。  
当使用ThreadLocal来维护变量时，ThreadLocal为每个线程维护一个线程独立的变量副本，这样每个线程操作自己的变量副本，解决多线程问题。
和同步加锁解决多线程相比，这是一种空间换时间的做法。

		public class ThreadLocalBase
		{
		
		    private static ThreadLocal<Integer> num = new ThreadLocal<Integer>()
		    {
		        @Override
		        public Integer initialValue()
		        {
		            return 0;
		        }
		
		    };
		
		    public int getNextNum()
		    {
		        num.set(num.get() + 1);
		
		        return num.get();
		    }
		
		    private static class Client extends Thread
		    {
		        private ThreadLocalBase base;
		
		        public Client(ThreadLocalBase base)
		        {
		            this.base = base;
		        }
		
		        @Override
		        public void run()
		        {
		            for (int i = 0; i < 3; i++)
		            {
		                System.out.println("thread:" + Thread.currentThread().getName()
		                        + "    num:" + base.getNextNum());
		            }
		        }
		    }
		
		    public static void main(String[] args)
		    {
		        ThreadLocalBase base = new ThreadLocalBase();
		        Client c1 = new Client(base);
		        Client c2 = new Client(base);
		        Client c3 = new Client(base);
		
		        c1.start();
		        c2.start();
		        c3.start();
		    }
		
		}
		
		
