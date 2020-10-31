【题目描述】

    某单位要对辖内管辖的客户业务数据进行采集，采集人及其上级可对自己采集的数据表数据进行访问，其他人员只有在授权的情况下才可对非本人采集的数据表数据进行访问。
【试题要求】

    有两个用户user01和user02，其上级用户为user03，其中用户user01采集的数据表为T_TABLE_1和T_TABLE_2，用户user02采集的数据表为T_TABLE_3。
	请设计一种方案（包括数据库表和JAVA授权及校验程序），实现用户对数据表数据访问权限的动态设置和权限校验。
    注：假定一张表只由一个用户进行采集。

【输入】
    
    数据库初始化脚本INIT_GEEK_DATA_AUTH.sql、INIT_GEEK_DATA_AUTH.bat。
	执行INIT_GEEK_DATA_AUTH.bat进行初始化（需要根据mysql数据库实际情况，修改bat脚本中的数据库用户名、密码、端口、数据库名称等信息）。
	输入样例：无。
【输出】

    1）提交数据库脚本（如有）：UPT_GEEK_DATA_AUTH.sql
    2）提交可执行jar包：geek-data-auth.jar，要求：
    a)将依赖的jar包全部打到geek-data-auth.jar中，能够直接执行；
    b)geek-data-auth.jar中包含3个可执行类DataAuthChk、DataAuthRevoke、DataAuthSet，分别实现权限验证、权限授权和权限取消，其中DataAuthChk为主类。

执行时所需参数及说明如下：

|类名|main 输入参数|	输入参数说明|	System.exit值说明|
|---|---|---|---|
|DataAuthChk.java|args[0]|用户ID|	-1：缺少输入参数  0：用户无权执行SQL  1：用户有权执行SQL|
|DataAuthChk.java|args[1]|	待执行SQL语句	|	-1：缺少输入参数  0：用户无权执行SQL  1：用户有权执行SQL|
|DataAuthSet.java|	args[0]|	待授权用户ID|	-1：缺少输入参数 0：授权成功|
|DataAuthSet.java|	args[1]|	待授权数据采集表表名|	-1：缺少输入参数 0：授权成功|
|DataAuthRevoke.java|	args[0]	|待取消授权用户ID|	-1：缺少输入参数  0：取消授权成功|
|DataAuthRevoke.java|	args[1]	|待取消授权数据采集表表名|	-1：缺少输入参数  0：取消授权成功|

三个可执行类所在包名为com.yusys.geek，伪代码分别如下：
```
package com.yusys.geek;
//权限校验
public class DataAuthChk {

/**
 * 判断指定的用户是否可执行SQL语句
 * @param userId 执行人ID
 * @param sql 待执行SQL语句
 */
public static final Boolean canVisit(String userId, String sql) {
    
处理逻辑……
if(用户userId可执行sql) {
    return true;
}else{
    return false;
}
}
public static void main(String[] args) {
   if (args.length != 2) {
      System.out.println("缺少参数：第1个参数为用户名，第2个参数为待执行的SQL");
      System.exit(-1);
   }

      if(DataAuthChk.canVisit(args[0], args[1])){
          System.out.print("用户["+args[0]+"]可执行SQL语句["+args[1]+"]");
            System.exit(1);
        }else{
            System.out.print("用户["+args[0]+"]无权执行SQL语句["+args[1]+"]");
            System.exit(0);
        }
   }

}


package com.yusys.geek;
//权限设置
public class DataAuthSet {
public static void main(String[] args) {
   if (args.length != 2) {
      System.out.println("缺少参数：第1个参数为待授权用户名，第2个参数为待授权表名");
      System.exit(-1);
   }
   
   处理逻辑……
   
System.out.println("用户["+args[0]+"]访问数据表["+args[1]+"]授权成功");
   System.exit(0);
   }

}

package com.yusys.geek;
//权限设置
public class DataAuthRevoke {
public static void main(String[] args) {
   if (args.length != 2) {
      System.out.println("缺少参数：第1个参数为待取消授权的用户名，第2个参数为待取消授权的表名");
      System.exit(-1);
   }
   
   处理逻辑……
   
System.out.println("用户["+args[0]+"]访问数据表["+args[1]+"]取消授权成功");
   System.exit(0);
   }

}
```



输出样例：（包括验证方法）

    按照初始化脚本数据，用户user01、user02、user03对表T_TABLE_1和T_TABLE_3的访问权限如下：
	T_TABLE_1	T_TABLE_2	T_TABLE_3
    user01	是	是	否
    user02	否	否	是
    user03	是	是	是
    其他用户	否	否	否

    验证方法如下：
    1）用户user02作为访问用户，无权执行指定SQL，执行以下命令：
    java -jar geek-data-auth.jar user02 "select t1.* from T_TABLE_1 t1 join T_TABLE_3 t3 on t1.DATA_DT=t2. DATA_DT"
    	输出结果：
    用户[user02]无权执行SQL语句[select t1.* from T_TABLE_1 t1 join T_TABLE_3 t3 on t1.DATA_DT=t2. DATA_DT]
    System.exit值为：0
    2）给用户user02授权访问表T_TABLE_1，执行以下命令：
    java -classpath geek-data-auth.jar com.yusys.geek.DataAuthSet user02 T_TABLE_1
    输出结果：
    用户[user02]访问数据表[T_TABLE_1]授权成功
    System.exit值为：0
    
    验证用户user02对表T_TABLE_1的访问权限，重复执行第1）步中的命令，输出结果：
    用户[user02]可执行SQL语句[select t1.* from T_TABLE_1 t1 join T_TABLE_3 t3 on t1.DATA_DT=t2. DATA_DT]
    System.exit值为：1
    
    3）取消用户user02对表T_TABLE_1的访问权限，执行以下命令：
    java -classpath geek-data-auth.jar com.yusys.geek.DataAuthRevoke user02 T_TABLE_1
    输出结果：
    用户[user02]访问数据表[T_TABLE_1]取消授权成功
    System.exit值为：0
    
    验证用户user02对表T_TABLE_1的访问权限，重复执行第1）步中的命令，输出结果：
    用户[user02]无权执行SQL语句[select t1.* from T_TABLE_1 t1 join T_TABLE_3 t3 on t1.DATA_DT=t2. DATA_DT]
    System.exit值为：0

【基线时间】
	
	3小时
【评分标准】
	
	正确性100%，得100分。
    满足以下验证结果（0-无权执行，1-有权执行），则认为正确率为100%：
    1）使用指定用户执行待验证的SQL进行验证，结果正确；
    2）对于无权执行的SQL，通过授权之后，有权执行，结果正确；
    3）取消授权后，无权执行，结果正确。