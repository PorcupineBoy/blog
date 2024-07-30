#### Hbase-java-Api

```
新增 ，根据tableName获取mutator,然后 新增put 每个put 添加family,columns 。
mutator 可以为list 。list.add(put)
motator.mulate(motator)
```





#### DefaultMapper

```
package com.project.mapper;
/**
*   @author:Porcupine
*   @date:2020/7/6
*   
*/
import java.io.IOException;
import java.lang.reflect.Field;
import java.lang.reflect.Modifier;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;


import org.apache.hadoop.hbase.client.Mutation;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.util.Bytes;

import info.data.common.json.JsonWrap;


public class DefaultMapper<T>  {
    private String columnFamily;
    private Map<String, Field> colums = new HashMap<>();
    private Class<T> clazz;

    public DefaultMapper(Class<T> clazz) {
        super();
        this.clazz = clazz;
        map();
    }

    @Override
    public T result2object(Result result) throws IllegalAccessException, InstantiationException, IOException {
        if (result.size() == 0) {
            return null;
        }
        T obj = clazz.newInstance();
        for (Entry<String, Field> entry : colums.entrySet()) {
            Field f = entry.getValue();
            byte[] val = result.getValue(Bytes.toBytes(columnFamily), Bytes.toBytes(entry.getKey()));
            if (val == null) {
                continue;
            }
            switch (f.getName()) {
                case "Integer":
                    f.setInt(obj, Bytes.toInt(val));
                    break;
                case "Long":
                    f.setLong(obj, Bytes.toLong(val));
                    break;
                case "Boolean":
                    f.setBoolean(obj, Bytes.toBoolean(val));
                    break;
                case "String":
                    f.set(obj, Bytes.toString(val));
                    break;
                default:
                    f.set(obj, JsonWrap.readAsObject(Bytes.toString(val), f.getType()));
                    break;
            }
        }
        return obj;
    }

    @Override
    public List<Mutation> object2mutation(T t) throws IllegalAccessException, IOException {
        List<Mutation> list = new ArrayList<>();
        Field rowkeyField = colums.get("rowkey");
        String rowkey = rowkeyField.get(t).toString();
        for (Entry<String, Field> entry : colums.entrySet()) {
            Put put = new Put(Bytes.toBytes(rowkey));
            String conlumn = entry.getKey();
            Object val = entry.getValue().get(t);
            if (val != null) {
                put.addColumn(Bytes.toBytes(columnFamily), Bytes.toBytes(conlumn),
                        Bytes.toBytes(JsonWrap.writeAsString(val)));
                list.add(put);
            }
        }
        return list;
    }

    /**
     * 获取所有字段 字段名称与hbase一一对应，存在一个final+static字段为列簇字段名称
     *
     * @throws IllegalAccessException
     * @throws IllegalArgumentException
     */
    private void map() {
        Field[] fs = clazz.getDeclaredFields();// 得到类中的所有属性集合
        for (int i = 0; i < fs.length; i++) {
            Field f = fs[i];
            f.setAccessible(true); // 设置些属性是可以访问的
            int m = f.getModifiers();
            if (!Modifier.isFinal(m) && !Modifier.isStatic(m)) {
                colums.put(f.getName(), f);
            } else if ("FAMILY".equals(f.getName())) {
                try {
                    columnFamily = f.get(clazz).toString();
                } catch (IllegalArgumentException | IllegalAccessException e) {
                    columnFamily = "f0";
                }
            }
        }
    }

}

```

