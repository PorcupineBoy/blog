# Pandas 技巧

## 处理列数据
### 单列lambda
``` python
data["运营商"] = data["运营商"].apply(lambda x: "其它" if x == "-" else x) 
```
### 单列自定义函数
``` python
def function(x):
	return x+"hello"
data["运营商"] = data["运营商"].apply(function)  // 对运营商的值没行数据加上"hello"
```
## pands drop
```python
init_data_df.drop("phone_pri", inplace=True, axis=1) # pandas 默认删除行数据
```

## 处理  多列
``` python
# demo1
pd_merge_data["流失率"] = pd_merge_data.apply(lambda row: row["数量_y"] / row["数量_x"], axis=1)
# deno2
pd_feature_["last_type"] = pd_feature_.apply(lambda row: rfm_value(row["r_value"],row["f_value"],row["m_value"] ), axis=1)
```

## 排序
```python
loss_data = data[data["流失状态"] == "loss"]
loss_data.sort_values(by='数量',axis = 0,ascending = False, inplace=True)
loss_data[["省份", "数量"]].head()
```

## groupby
``` python
new_data = data[["省份", "数量"]].groupby(by="省份").sum().reset_index()
new_data.sort_values(by = '数量',axis = 0,ascending = False, inplace=True)
new_data.head()
# 支持sum()、mean() 、count()等函数 
.reset_index() # 维持原列数据，可以拿掉看下效果
```
## merge
```python
pd_merge_data =  pd.merge(new_data, loss_data, on="省份")
pd_merge_data.head()   

# 多表关联
pd_feature_ = pd.merge(pd.merge(pd_user_data_m, pd_user_data_f), pd_user_data_r)   # 默认on 是第一个字段，可以指定
```
[merge 函数的文档](https://blog.csdn.net/zutsoft/article/details/51498026)

## 条件过滤
```python
loss_data = data[data["流失状态"] == "loss"]

# 正则过滤
df.filter(regex=("d.*"))

# 多条件过滤
print df[(df['PCTL']<0.95) & (df['PCTL']>0.05)]
```

## pandas画图
```python
data_train.Survived.value_counts().plot(kind='bar') # 画条形图
```

## 查看存在的数据列
```python
data_train.columns   # 不需要加()
```

## 查看字段信息
```python
data.info()   
```

## 数据统计描述
```python
data.describe()  # 只会对数值型的数据进行描述
```

## 正则过滤
```python
import re
import pandas as pd
pattern = "^(?:\+?86)?1(?:3\d{3}|5[^4\D]\d{2}|8\d{3}|7(?:[35678]\d{2}|4(?:0\d|1[0-2]|9\d))|9[189]\d{2}|66\d{2})\d{6}"
data = ["18046716787@qq.com", "18946716787@qq.com",
        "289466716787@qq.com", ""]
dataDf = pd.DataFrame(data, columns=["a"])
newDataDf = dataDf[dataDf["a"].str.contains(pattern, na=True)]
print(re.findall(pattern, "18046716787@qq.com"))
print(newDataDf)
```

## 统计缺失值
```python
data = dadaDf.isna().sum()  # 统计缺失值
```

## pandas isin()
**效率会高于apply**

- 产生新的列
```python
import pandas as pd
pd_data = pd.DataFrame([[3, "a"], [4, "b"], [4, "a"]], columns=["n", "s"])
pd_high = pd.DataFrame([["a"], ["h"]], columns=["high"])
pd_data.loc[pd_data["s"].isin(pd_high["high"]), "s1"] = "高"
print(pd_data.head())
```



## pd.isna()

- 判断是否是缺失值

```python
def data_into_str(tmp_s):
    if tmp_s == "" or pd.isna(tmp_s):
        return "0-0"
    else:
        return str(int(tmp_s))
```

## pandas 构建聚合特征 udf agg
```python
import pandas as pd

df = pd.DataFrame({'code': ['上传', '上传', '下载', '下载', '分享', '备份', '上传', '下载'],
                   'n': ["yz", "yz", "cx", "cx", "cx", "yz", "cx", "cx"],
                   'times': [5000, 4321, 1234, 4010, 250, 250, 4500, 4621]})
pd_data = df.groupby(by=["n", "code"]).sum().reset_index()
pd_data["fea"] = pd_data["code"].str.cat(pd_data["times"].astype(str), sep="-")
def udf_agg(tmp_df):
    return ";".join(tmp_df["fea"].values)
new_ = pd_data[["n", "fea"]].groupby(by=["n"]).apply(udf_agg).reset_index()
print(new_.head())

```

## pandas get_dummies构建集合特征
```python
import pandas as pd
df = pd.DataFrame({'code': ['上传', '上传', '下载', '下载', '分享', '备份', '上传', '下载'],
                   'n': ["yz", "yz", "cx", "cx", "cx", "yz", "cx", "cx"],
                   'times': [5000, 4321, 1234, 4010, 250, 250, 4500, 4621]})

dummies_Cabin = pd.get_dummies(df['code'], prefix='code')
# 特征拼接
df = pd.concat([df, dummies_Cabin], axis=1)
df = df.groupby(by=["n"]).sum().reset_index()
print(df.head(100))
```

## 缺失值预测代码
```python
def set_missing_ages(df):
    # 把已有的数值型特征取出来丢进Random Forest Regressor中
    age_df = df[['Age','Fare', 'Parch', 'SibSp', 'Pclass']]
    # 乘客分成已知年龄和未知年龄两部分
    known_age = age_df[age_df.Age.notnull()].as_matrix()
    unknown_age = age_df[age_df.Age.isnull()].as_matrix()
    # y即目标年龄
    y = known_age[:, 0]
    # X即特征属性值
    X = known_age[:, 1:]
    # fit到RandomForestRegressor之中
    rfr = RandomForestRegressor(random_state=0, n_estimators=2000, n_jobs=-1)
    rfr.fit(X, y)
    # 用得到的模型进行未知年龄结果预测
    predictedAges = rfr.predict(unknown_age[:, 1::])
    # 用得到的预测结果填补原缺失数据
    df.loc[ (df.Age.isnull()), 'Age' ] = predictedAges 
    return df, rfr
```

## loc用户2
```python
# 可以查看isin()的用法
pd_data.loc[pd_data["s"].isin(pd_high["high"]), "s1"] = "高" 
```

## 结合正则表达式过滤
```python
    data['domain_space_account'].astype(str)
    data = data[data["domain_space_account"].str.contains(pattern)]
```

## 综合技巧
```python
import pandas as pd
# 筛选出手机号
def getPhone(data):
    pattern = r'(?:\+?86)?1(?:3\d{3}|5[^4\D]\d{2}|8\d{3}|7(?:[35678]\d{2}|4(?:0\d|1[0-2]|9\d))|9[189]\d{2}|66\d{2})\d{6}'
    # 筛选非空数据
    data = data[data['domain_space_account'].notnull()]
    # 将 domain_space_account 中包含手机号码的筛选出来,并将手机号替换domain_space_account
    data['domain_space_account'].astype(str)
    data = data[data["domain_space_account"].str.contains(pattern)]
    data.loc[:, "domain_space_account"] = data['domain_space_account'].str.findall(pattern)[0]
    return data
pd_data = pd.DataFrame([["19925819400@qq.com", 373], ["", 323],
                        ["19925819408@qq.com", 53]], columns=["domain_space_account", "name"])
getPhone(pd_data)
```

## 对缺失值的数据编码map
```python
# 对client_type_id进行0，1编码
data['client_type_id'] = data['client_type_id'].isnull().map({True: 0, False: 1})  # 14

import pandas as pd
data = pd.DataFrame([["dsd", 32], ["xu", 3], ["dsd", 8]], columns=["name", "value"])
data["value"] = data.value.apply(lambda x: "1" if x > 10 else "2" if x > 5 else "3").map(
    {"1": "name", "2": "sex", "3": "male"})
print(data)
```



## pd_cut生成数据集
```python
    time_name = ['early_morning', 'morning', 'forenoon', 'noon', 'afternoon', 'night', 'late-night']
    times = [-1, 7, 9, 12, 14, 18, 22, 24]
    time_set = pd.cut(data['create_time'], times, labels=time_name)
    data['create_time'] = time_set
```



## 信誉库的暑假测试
```python
import re
date_pattern = re.compile('\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}')

print(data[~data["审核时间"].str.contains(date_pattern)][["分享时间",
                                                      "分享人账号", "文件id"]].head())
print(data["文件来源"].value_counts())

print(data[data["分享人账号"] == "15692186012@189.cn"
           ][data["文件id"] == 314591931655944][["文件id",
                                               "文件名", "分享人账号", "分享时间"]].head())
print(data[["文件id", "分享人账号", "文件名"]]
      .groupby(by=["文件id", "分享人账号"]).count().reset_index()
      .sort_values(by="文件名", ascending=False).head())
```

## 机器学习测试案例一
```python
import pandas as pd
from sklearn.cross_validation import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

pd_data = pd.read_table(open(r"C:\Users\123456\Desktop\data.txt", encoding="utf-8"),
                        delimiter=",")
pd_data["gender"] = pd_data["gender"].map({"男": 1, "女": 0}).astype(int)
y_data = pd_data["gender"]
train_x = pd_data.drop(["gender", "phone"], axis=1)
X_train, X_test, y_train, y_test = train_test_split(train_x, y_data, test_size=0.2,
                                                    random_state=42)
clf = RandomForestClassifier()
clf.fit(X_train, y_train)
y_predict = clf.predict(X_test)
acc = accuracy_score(y_predict, y_test)
print(acc)
print(list(y_predict))
```