	Java对Hbase进行操作
	
	-------------------------------------------------------------------------------------------

	package cn.gyyx.app.wd.data.utils;
	
	import java.io.IOException;
	import org.apache.hadoop.conf.Configuration;
	import org.apache.hadoop.hbase.HBaseConfiguration;
	import org.apache.hadoop.hbase.HColumnDescriptor;
	import org.apache.hadoop.hbase.HTableDescriptor;
	import org.apache.hadoop.hbase.KeyValue;
	import org.apache.hadoop.hbase.client.Delete;
	import org.apache.hadoop.hbase.client.Get;
	import org.apache.hadoop.hbase.client.HBaseAdmin;
	import org.apache.hadoop.hbase.client.HTable;
	import org.apache.hadoop.hbase.client.Put;
	import org.apache.hadoop.hbase.client.Result;
	import org.apache.hadoop.hbase.client.ResultScanner;
	import org.apache.hadoop.hbase.client.Scan;
	import org.apache.hadoop.hbase.util.Bytes;
	
	public class HBaseUtil {
		
		private static Configuration config = null;
		
		static{
			Configuration conf = new Configuration();
			conf.set("hbase.zookeeper.quorum", "192.168.110.137");
			//连接zookeeper()
			conf.set("hbase.zookeeper.property.clientPort", "2181");
			//解决租约到期的问题 3000s
			conf.set("hbase.regionserver.scanner.timeout.period", "3000000");
			conf.set("hbase.rpc.timeout", "3000000");
			//解决超时问题
			conf.set("mapred.task.timeout", "3000000");
			//缓存扫描数据
			conf.set("hbase.client.scanner.caching", "10000");
			//创建hbaseconfig
			config = HBaseConfiguration.create(conf);
		}
		
		/**
		 * 
		 * @Method:creatTable
		 * @Description：TODO(创建表)
		 * @param tableName
		 * @param family
		 * @throws Exception
		 * void
		 */
		public static void creatTable(String tableName, String[] family)  
	            throws Exception {  
			HBaseAdmin admin = new HBaseAdmin(config);  
			HTableDescriptor desc = new HTableDescriptor(tableName);  
			for (int i = 0; i < family.length; i++) {  
			    desc.addFamily(new HColumnDescriptor(family[i]));  
			}  
			if (admin.tableExists(tableName)) {  
			    System.out.println("table Exists!");  
			    System.exit(0);  
			} else {  
			    admin.createTable(desc);  
			    System.out.println("create table Success!");  
			}  
	 	}
		
		/**
		 * 
		 * @Method:getResult
		 * @Description：TODO(根据rowkey查询表数据)
		 * @param tableName
		 * @param rowKey
		 * @return
		 * @throws IOException
		 * Result
		 */
		public static Result getResult(String tableName, String rowKey) throws IOException{  
			Get get = new Get(Bytes.toBytes(rowKey));  
			HTable table = new HTable(config, Bytes.toBytes(tableName));// 获取表  
			System.out.println("--------------------根据rowkey查询-----------------------"); 
			Result result = table.get(get);  
			for (KeyValue kv : result.list()) { 
				System.out.println("-------------------------------------------");
			    System.out.println("family:" + Bytes.toString(kv.getFamily()));  
			    System.out.println("qualifier:" + Bytes.toString(kv.getQualifier()));  
			    System.out.println("value:" + Bytes.toString(kv.getValue()));  
			    System.out.println("Timestamp:" + kv.getTimestamp());  
			    System.out.println("-------------------------------------------");  
			}  
			return result;  
	    	}  
		
		/**
		 * 
		 * @Method:getResultScann
		 * @Description：TODO(遍历表查询表数据)
		 * @param tableName
		 * @throws IOException
		 * void
		 */
		public static void getResultScann(String tableName) throws IOException {  
			Scan scan = new Scan();  
			ResultScanner rs = null;  
			HTable table = new HTable(config, Bytes.toBytes(tableName));
			System.out.println("--------------------遍历查询-----------------------"); 
			try {  
			    rs = table.getScanner(scan);  
			    for (Result r : rs) {  
				for (KeyValue kv : r.list()) {  
				    System.out.println("row:" + Bytes.toString(kv.getRow()));  
				    System.out.println("family:" + Bytes.toString(kv.getFamily()));  
				    System.out.println("qualifier:" + Bytes.toString(kv.getQualifier()));  
				    System.out.println("value:" + Bytes.toString(kv.getValue()));  
				    System.out.println("timestamp:" + kv.getTimestamp());  
				    System.out.println("-------------------------------------------");  
				}  
			    }  
			} finally {  
			    rs.close();  
			}  
	   	 }
		
		/**
		 * 
		 * @Method:getResultByColumn
		 * @Description：TODO(查询表中某一列)
		 * @param tableName
		 * @param rowKey
		 * @param familyName
		 * @param columnName
		 * @throws IOException
		 * void
		 */
		public static void getResultByColumn(String tableName, String rowKey,  
	            String familyName, String columnName) throws IOException {  
			HTable table = new HTable(config, Bytes.toBytes(tableName));  
			Get get = new Get(Bytes.toBytes(rowKey));  
			get.addColumn(Bytes.toBytes(familyName), Bytes.toBytes(columnName)); // 获取指定列族和列修饰符对应的列  
			Result result = table.get(get);  
			for (KeyValue kv : result.list()) {  
			    System.out.println("family:" + Bytes.toString(kv.getFamily()));  
			    System.out.println("qualifier:" + Bytes.toString(kv.getQualifier()));  
			    System.out.println("value:" + Bytes.toString(kv.getValue()));  
			    System.out.println("Timestamp:" + kv.getTimestamp());  
			    System.out.println("-------------------------------------------");  
			}  
	    	}
		
		/**
		 * 
		 * @Method:updateTable
		 * @Description：TODO(更新表中某一列)
		 * @param tableName
		 * @param rowKey
		 * @param familyName
		 * @param columnName
		 * @param value
		 * @throws IOException
		 * void
		 */
		public static void updateTable(String tableName, String rowKey,  
	            String familyName, String columnName, String value)  
	            throws IOException {  
			HTable table = new HTable(config, Bytes.toBytes(tableName));  
			Put put = new Put(Bytes.toBytes(rowKey));  
			put.add(Bytes.toBytes(familyName), Bytes.toBytes(columnName),  
				Bytes.toBytes(value));  
			table.put(put);  
			System.out.println("update table Success!");  
	    	}
		
		/**
		 * 
		 * @Method:deleteColumn
		 * @Description：TODO(删除指定参数的列)
		 * @param tableName
		 * @param rowKey
		 * @param falilyName
		 * @param columnName
		 * @throws IOException
		 * void
		 */
		public static void deleteColumn(String tableName, String rowKey,  
	            String falilyName, String columnName) throws IOException {  
			HTable table = new HTable(config, Bytes.toBytes(tableName));  
			Delete deleteColumn = new Delete(Bytes.toBytes(rowKey));  
			deleteColumn.deleteColumns(Bytes.toBytes(falilyName),  
				Bytes.toBytes(columnName));  
			table.delete(deleteColumn);  
			System.out.println(falilyName + ":" + columnName + "is deleted!");  
	    	} 
		
		/**
		 * 
		 * @Method:deleteAllColumn
		 * @Description：TODO(删除指定rowkey的列 )
		 * @param tableName
		 * @param rowKey
		 * @throws IOException
		 * void
		 */
		public static void deleteAllColumn(String tableName, String rowKey)  
	            throws IOException {  
			HTable table = new HTable(config, Bytes.toBytes(tableName));  
			Delete deleteAll = new Delete(Bytes.toBytes(rowKey));  
			table.delete(deleteAll);  
			System.out.println("all columns are deleted!");  
	   	}
		
		/**
		 * 
		 * @Method:deleteTable
		 * @Description：TODO(删除表)
		 * @param tableName
		 * @throws IOException
		 * void
		 */
		public static void deleteTable(String tableName) throws IOException {  
			HBaseAdmin admin = new HBaseAdmin(config);  
			admin.disableTable(tableName);  
			admin.deleteTable(tableName);  
			System.out.println(tableName + "is deleted!");  
	    	}
		
		/**
		 * 
		 * @Method:put
		 * @Description：TODO(添加数据)
		 * @param tableName
		 * @param rowKey
		 * @param family
		 * @param column
		 * @param value
		 * void
		 */
		public static void put(String tableName,String rowKey,String family,String column,String value)  
	    	{    
			try {  
			    HTable table=new HTable(config,tableName);  
			    HBaseAdmin admin=new HBaseAdmin(config);  
			    //判断表是否存在，如果不存在进行创建  
			    if(!admin.tableExists(Bytes.toBytes(tableName)))  
			    {  
				HTableDescriptor tableDescriptor=new HTableDescriptor(Bytes.toBytes(tableName));  
				HColumnDescriptor columnDescriptor=new HColumnDescriptor(Bytes.toBytes(family));  
				tableDescriptor.addFamily(columnDescriptor);  
				admin.createTable(tableDescriptor);  
			    }  
			    table.setAutoFlush(true);  
			    //进行数据插入  
			    Put put=new Put(Bytes.toBytes(rowKey));  
			    put.add(Bytes.toBytes(family),Bytes.toBytes(column),Bytes.toBytes(value));  
			    table.put(put);  
			    table.close();  
			} catch (IOException e) {  
			    // TODO Auto-generated catch block  
			    e.printStackTrace();  
			}  
	    	}  
		
		public static void main(String[] args) throws Exception {
			System.out.println("===================");
			//getResult("member","xt");
			//put("member","xt","info","add","添加");
			//getResultScann("member");
			//String[] strs = {"111","222"};
			//creatTable("member2",strs);sssssssss
			//put("member2","222","111","add2","添加2");
			//deleteColumn("member2", "222", "222", "");
			//deleteAllColumn("member2","222");
			//deleteTable("member2");
		}
	}


 
