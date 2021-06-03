# Protocol Buffers



### 定义A消息类型

```
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

指定使用proto3，如果不指定的话，编译器会使用proto2编译。

变量类型、变量名、变量Tag



### 定义变量类型



### 分配Tag