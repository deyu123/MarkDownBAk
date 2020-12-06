# Hive自定义函数

***



## 一、概述

```sql
--1. 在hive中有三种自定义函数：
1. UDF  ：一进一出  --一行变一行
2. UDTF : 一进多出  -- 一行变多行
3. UDAF ：多进一出  -- 多行变一行

-- 2. 实现步骤：
   a、进入函数的是什么参数
   b、希望得到什么结果
   c、考虑通用性
```

## 二、UDTF函数

### 2.1 UDTF解析

```sql
-- 1. 说明
A custom UDTF can be created by extending the GenericUDTF abstract class and then implementing the initialize, process, and possibly close methods. The initialize method is called by Hive to notify the UDTF the argument types to expect. The UDTF must then return an object inspector corresponding to the row objects that the UDTF will generate. Once initialize() has been called, Hive will give rows to the UDTF using the process() method. While in process(), the UDTF can produce and forward rows to other operators by calling forward(). Lastly, Hive will call the close() method when all the rows have passed to the UDTF.

    -- 解析如上内容：
    1. 自定义UDTF函数，继承于抽象类GenericUDTF;
    2. 实现initialize, process,close 的三个方法；
    3. 三个方法的作用说明：
       1. 'initialize()'：
          a、规定形参的参数类型：Hive调用initialize()方法，来告诉UDTF函数接收参数的类型，然后在这个方法中就可以对参数的
          类型进行校验，参数类型包括个数、数据类型等等
          b、规定函数返回结果数据类型：返回initialize()方法必须返回一个与UDTF将生成的行对象对应的对象检查器   
       2. 'process()'：
          a、调用时机：initialize()被调用以后，即对输入的数据进行了检查满足条件以后，执行这个方法
          b、作用：一行数据调用一次process方法，process方法的内部，会遍历这一行数据，如果是数组，那就遍历数组，遍历以后的
          一个元素，调用一次forward()方法，对数据进行输出。
       3. 'close()'：
          a、调用时机：当hive中的所有行都通过了UDTF函数以后，则Hive会调用这个close方法
          b、作用：关闭资源。      
```

### 2.2 代码实现

```java
package com.atguigu.hive;

import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDTF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.StructField;
import org.apache.hadoop.hive.serde2.objectinspector.StructObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;
import org.json.JSONArray;

import java.util.ArrayList;
import java.util.List;

/**
 * @author lianzhipeng
 * @Description
 * @create 2020-06-30 18:25:10
 */

public class ExplodeJSONArray extends GenericUDTF {
    /*
    StructObjectInspector:结构体,输出数据类型和返回数据类型均是结构体
    UDTF：返回的数据可以是多列，多列可以封装到struct类型中
    [agr1:type1,agr2:type2,agr3:type3,...]
     */
    @Override
    public StructObjectInspector initialize(StructObjectInspector argOIs) throws UDFArgumentException {

        // 1. 获取输入的参数，并对输入的参数进行检测
        /*
         1.1 获取形参，获取的数据为一个数组。
             List(agr1:type1,agr2:type2,agr3:type3,...)
          */
        List<? extends StructField> fieldRefs = argOIs.getAllStructFieldRefs();

        /*
         1.2 检验一：检验数据的个数是否为1个，在本案例中，传进来一个json数组
          */
        if (fieldRefs.size() != 1) {
            throw new UDFArgumentException("传递参数的个数超过1个");
        }

        /*
        1.3 校验2 ：判断形参的数据类型是否为string类型
         */
        // 返回一个参数，因为我们规定只能是一个参数，返回参数的检查器
        ObjectInspector fieldObjectInspector = fieldRefs.get(0).getFieldObjectInspector();

        // 通过参数的检查器返回参数的数据类型判定需要是字符串类型
        if (!"string".equals(fieldObjectInspector.getTypeName())) {
            throw new UDFArgumentException("传入的参数类型错误，需要是string类型");
        }

        // 2. 返回函数结果的检查器
        // 创建两个数组，一个用来包装列名，另外一个用来包装列名的数据类型
        // 数组1：包装列名
        List<String> filenames = new ArrayList<String>();
        // 数组2：包装列名的数据类型
        ArrayList<ObjectInspector> fileOIS = new ArrayList<ObjectInspector>();

        // 添加元素
        filenames.add("actions");
        fileOIS.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);

        return ObjectInspectorFactory.getStandardStructObjectInspector(filenames, fileOIS);
    }

    /**
     * 对数据进行处理，执行函数逻辑的方法，每一行数据会调用一次process方法
     * 1.为什么是object类型？
     * 表示任意参数
     * 2.为什么是一个数组？
     * 表示可以传多个
     *
     * @param args 每一行数据会调用一次process方法
     * @throws HiveException
     */
    @Override
    public void process(Object[] args) throws HiveException {
        /*
         1.在initialize方法中，已经对输入的数据进行校验，此时的形参中只能是一个
         参数，参数为一个json数组。
       [{"action_id":"favor_add","item":"7","item_type":"sku_id","ts":1592668888084}]
           */
        String json = args[0].toString();

        /*
        2. 对上面的json进行解析
        jsonArray ： List({"action_id":"favor_add","item":"7","item_type":"sku_id","ts":1592668888084},....)
         */
        JSONArray jsonArray = new JSONArray(json);

        // 3. 遍历上面的json数组，行中的每一个数据都需要调用一次forward方法，将结果进行写出
        for(int i = 0 ; i < jsonArray.length() ; i++){
            // 创建一个数组，数组的长度为1
            String[] result = new String[1];
            // 获取jsonArray指定位置的json数据，并存放到数组中
            result[0] = jsonArray.getString(i);
            forward(result);

        }

    }

    @Override
    public void close() throws HiveException {

    }
}

```

## 三、UDF函数

### 3.1 代码实现

```java
package com.atguigu.hive.udf;

import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDF;
import org.apache.hadoop.hive.serde2.objectinspector.ConstantObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;
import org.json.JSONArray;
import org.json.JSONObject;

import java.util.ArrayList;
import java.util.List;

/**
 * @author lianzhipeng
 * @Description
 * @create 2020-07-01 2:03:24
 */

public class JsonArrayToStructArray extends GenericUDF {

    /*
    本案例中实现的目的：
    原始数据：
   [{"action_id":"favor_add","item":"5","item_type":"sku_id","ts":1592668884186},{...}]
     转换后数据：
     array<struct("action_id":"favor_add","item":"5","item_type":"sku_id","ts":1592668884186),struct(...)>
     */

    /**
     * 用来校验输入的参数类型和个数，同时返回函数返回结果的检查器
     *
     * @param arguments 输入的参数
     * @return
     * @throws UDFArgumentException
     */
    @Override
    public ObjectInspector initialize(ObjectInspector[] arguments) throws UDFArgumentException {

    /*
    1. 为了实现代码的通用性，由传入的形参的数据列类型和列名来确定返回值的检查器。
    2. 则在实际的调用的过程中，使用：如下案例所示
       a、"id","name","age"，表示从json对象获取的字段
       b、“id:string”,"name:string","age:int"：表示每个字段的数据类型
       如上两个参数完全由用户自定义，作为参数传入到udf的方法中。
    JsonArrayToStructArray(jsonarray,"id","name","age",“id:string”,"name:string","age:int")

     3. 实现步骤：
        a、获取形参中的第一个参数，返回一个字符串类型的json对象
        array<Struct<k1:v1,k2:v2,k3:v3,....>
        b、创建两个集合，一个集合用来接收k1~kn列名，另外一个集合用来接收v1~vn的数据类型
     */
        // 1. 对输入的参数进行校验
        // 判断参数的个数
        if (arguments.length < 3) {
            throw new UDFArgumentException("JsonArrayToStructArray至少需要3个参数");
        }
        // 判断参数的数据类型
        for (ObjectInspector argument : arguments) {
            if (!"string".equals(argument.getTypeName())) {
                throw new UDFArgumentException("JsonArrayToStructArray输入的参数类型不对，只能是string类型");
            }
        }

        // 2. 对返回结果的数据类型和名字进行封装
        // array<Struct<k1:v1,k2:v2,k3:v3,....>,...>
        List<String> FieldNames = new ArrayList<>();
        List<ObjectInspector> fieldOIS = new ArrayList<>();

        // 2.1 获取传递参数的“id:string”,"name:string","age:string"
        //     然后将id、name，age加入到FieldNames中
        //     将string、string、int加入到fieldOIS中
        // JsonArrayToStructArray(jsonarray,"id","name","age",“id:string”,"name:string","age:int")
        for (int i = (arguments.length + 1) / 2; i < arguments.length; i++) {
            if (!(arguments[i] instanceof ConstantObjectInspector)) {
                throw new UDFArgumentException("JsonArrayToStructArray输入的参数类型不对，只能是string常量");
            }
            // 此时获取：field=“id:string”
            String field = ((ConstantObjectInspector) arguments[i]).getWritableConstantValue().toString();
            // 对上诉数据进行切分
            String[] split = field.split(":");
            // 添加数据
            FieldNames.add(split[0]);
            switch (split[1]) {
                case "int":
                    fieldOIS.add(PrimitiveObjectInspectorFactory.javaIntObjectInspector);
                    break;
                case "string":
                    fieldOIS.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);
                    break;
                case "bigint":
                    fieldOIS.add(PrimitiveObjectInspectorFactory.javaLongObjectInspector);
                    break;
                default:
                    throw new UDFArgumentException("json_array_to_struct_array 不支持" + split[1] + "类型");
            }
        }
        return ObjectInspectorFactory
               .getStandardListObjectInspector(ObjectInspectorFactory.getStandardStructObjectInspector(FieldNames, fieldOIS));
    }
    /**
     * 处理的逻辑
     * array类型：传入一个数组或集合
     * map类型：Map
     * struct：数组或集合
     *
     * @param arguments
     * @return
     * @throws HiveException
     */
    @Override
    public Object evaluate(DeferredObject[] arguments) throws HiveException {
        // JsonArrayToStructArray(jsonarray,"id","name","age",“id:string”,"name:string","age:int")
        // arguments = 'jsonarray',"id","name","age",“id:string”,"name:string","age:int"

        // 获取第一个数据jsonarray，判断是否为空
        if (arguments[0].get() == null) {
            return null;
        };
        // 获取第一个数据jsonarray:"[{"name":"scala"},{"name":"python"}]"
        String strArray = arguments[0].get().toString();
        // 解析json数据:jsonArray = {"name":"scala"},{"name":"python"}
        JSONArray jsonArray = new JSONArray(strArray);
        // 创建一个集合，最外层的集合
        //array<struct("action_id":"favor_add","item":"5","item_type":"sku_id","ts":1592668884186),struct(...)>
        ArrayList<List<Object>> array = new ArrayList<>();
        // 接下来就是遍历这个json对象，每一个对象封装成struct类型的对象
        for (int i =0 ; i < jsonArray.length() ; i ++){
            // 创建一个集合，用来接收value值
            ArrayList<Object> struct = new ArrayList<>();
            // 获取一个json对象:{"name":"scala"}
            JSONObject jsonObject = jsonArray.getJSONObject(i);
            // 遍历"id","name","age"
            for (int j = 1 ; j < (arguments.length + 1 ) / 2 ;j++){
             // 轮询的方式获取"id","name","age"
                String key = arguments[j].get().toString();
               // 判断当前的json对象中是否有key
                if (jsonObject.has(key)){
                    // 获取key的value值
                    Object value = jsonObject.get(key);
                    // 添加元素
                    struct.add(value);
                }else {
                    struct.add(null);
                }
            }
            // 将获取json的指定key【"id","name","age"】的value值放进数组中
            array.add(struct);
        }
        return array;
    }
    /**
     *   这个方法也可以不实现。
     * @param children
     * @return
     */
    @Override
    public String getDisplayString(String[] children) {
        // json_array_struct_array:指函数的名字
        return getStandardDisplayString("json_array_struct_array",children);
    }
}


```

