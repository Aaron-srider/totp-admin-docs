# 前后端参数

## 参数格式

**前端->后端**

变量命名规则：驼峰命名

示例：
* tableName
* fileId

**后端->前端**

变量规则命名：单词之间用下划线连接

示例：
* table_name
* file_id

# 后端返回值


|  返回值类型   | 返回值  |说明  |
|  ----  | ----  | ----  |
| 全局返回值  | GENERAL_ERROR |后端代码出错，属于bug |
| 全局返回值  | FRONT_END_PARAMS_ERROR |前端参数没有按照API要求传递 |
