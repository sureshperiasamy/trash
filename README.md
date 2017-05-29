sqoop import --connect jdbc:mysql://localhost:3306/sqoop --username root -P --hive-import --table sqoop1 --create-hive-table --hive-table db.sqtest --m 1 --driver com.mysql.jdbc.Driver

-

a = load 'hdfs://localhost:9000/project/flume_data/StatewiseDistrictwisePhysicalProgress.xml' using org.apache.pig.piggybank.storage.XMLLoader('row') as (x:chararray);


B = foreach a generate REPLACE(x,'[\\n]','') as x;

C = foreach B generate REGEX_EXTRACT_ALL(x,'.*(?:<State_Name>)([^<]*).*(?:<District_Name>)([^<]*).*(?:<Project_Objectives_IHHL_BPL>)([^<]*).*(?:<Project_Objectives_IHHL_APL>)([^<]*).*(?:<Project_Objectives_IHHL_TOTAL>)([^<]*).*(?:<Project_Objectives_SCW>)([^<]*).*(?:<Project_Objectives_School_Toilets>)([^<]*).*(?:<Project_Objectives_Anganwadi_Toilets>)([^<]*).*(?:<Project_Objectives_RSM>)([^<]*).*(?:<Project_Objectives_PC>)([^<]*).*(?:<Project_Performance-IHHL_BPL>)([^<]*).*(?:<Project_Performance-IHHL_APL>)([^<]*).*(?:<Project_Performance-IHHL_TOTAL>)([^<]*).*(?:<Project_Performance-SCW>)([^<]*).*(?:<Project_Performance-School_Toilets>)([^<]*).*(?:<Project_Performance-Anganwadi_Toilets>)([^<]*).*(?:<Project_Performance-RSM>)([^<]*).*(?:<Project_Performance-PC>)([^<]*).*');


D = FOREACH C GENERATE FLATTEN ($0);

STORE D INTO ' hdfs://localhost:9000/xmlfile/' USING PigStorage (',');


-



loadJson = LOAD '/olympic.json' USING JsonLoader('athelete:chararray,age:INT,country:chararray,year:chararray,closing:chararray,sport:chararray,gold:INT,silver:INT,bronze:INT,total:INT');

-

hive> set hive.auto.convert.join=true;
set hive.auto.convert.join.noconditionaltask=true;



-

import org.apache.hadoop.hive.ql.exec.UDAF;
import org.apache.hadoop.hive.ql.exec.UDAFEvaluator;

public class hiveudf extends UDAF {
  public static class udfeval implements UDAFEvaluator
  {
	  
  public static class column
  {
	  int sum = 0;
  }
private column col = null;

public udfeval()
{
	super();
	init();
}
	@Override
	public void init() {
		// TODO Auto-generated method stub
		col = new column(); 
	}

	
public boolean iterate(int value)
{
	if(col==null)
	{
		return true;
		
	}
	else
	{
		col.sum = col.sum+value;
		return true;
	}
}
public boolean merge(column other)
{
	if (other == null)
	{
		return true;
		
	}
	else
	{
		col.sum = col.sum + other.sum;
		return true;
	}
	

  }

public column terminatePartial()
{
	return col;
}
public  int terminate()
{
	return col.sum;
	
}

  
  } 
}
	  
  	
---




import java.util.ArrayList;

import org.apache.hadoop.hive.ql.exec.UDF;
import org.apache.hadoop.io.Text;

public class hiveudf2 extends UDF{
	public Text evaluate(String str ,ArrayList<String> str1)
	{
		StringBuffer sb = new StringBuffer();
		for (int i = 0; i < str1.size(); i++) {
			sb.append(str1.get(i));
			sb.append(str);
			
		}
		return new Text(sb.toString());
		
	}
	

}


--

hive> set hive.auto.convert.join=true;

hive> set hive.auto.convert.join.noconditionaltask=true;

--
public class ToolMapReduce extends Configured implements Tool {
 
    public static void main(String[] args) throws Exception {
        int res = ToolRunner.run(new Configuration(), new ToolMapReduce(), args);
        System.exit(res);
    }

 public int run(String[] args) throws Exception {
 
        // When implementing tool
        Configuration conf = this.getConf();
 
        // Create job
        Job job = new Job(conf, "Tool Job");
        job.setJarByClass(ToolMapReduce.class);
 
        // Setup MapReduce job
        // Do not specify the number of Reducer
        job.setMapperClass(Mapper.class);
        job.setReducerClass(Reducer.class);
 
        // Specify key / value
        job.setOutputKeyClass(LongWritable.class);
        job.setOutputValueClass(Text.class);
 
        // Input
        FileInputFormat.addInputPath(job, new Path(args[0]));
        job.setInputFormatClass(TextInputFormat.class);
 
        // Output
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        job.setOutputFormatClass(TextOutputFormat.class);
 
        // Execute job and return status
        return job.waitForCompletion(true) ? 0 : 1;
    }
}
