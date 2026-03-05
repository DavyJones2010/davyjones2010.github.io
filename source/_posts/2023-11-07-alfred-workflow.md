---
title: Alfred Workflow 使用指南
date: 2023-11-07 00:00:00
tags: ['alfred', 'macos', 'productivity']
category: tech-notes

---

# Components

---

介绍主要有哪些基础组件及其用法

## Arg/Var utility

1. 把 输入值(param) 保存到 自定义的参数(例如 my_task) 中. 
2. 后续可以使用 {var:my_task} 来引用该值.
- 如下, 把 {query} 保存到 task 参数中

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/2023-11-07-alfred-workflow/Untitled.png)

- 如下, 使用参数 {var:task}

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/2023-11-07-alfred-workflow/Untitled-1.png)

## ListFilter

# Practices

## 如何对参数进行dict映射?

如下需求: 

1. 提前定义好了字典内容: cn-beijing=cn-beijing-btc-a01; cn-hangzhou=cn-hangzhou-dg-a01; 
2. 输入了参数 cn-beijing, 需要能映射为对应的value, 即 cn-beijing-btc-a01 
3. 然后把该value赋值到一个新的变量 `region` 上

这里选用自己熟悉的bash脚本,  分为几部分: 

### 1. 如何对输入参数进行映射?

由于bash里定义dict的数据格式, 需要较高的版本, 兼容性不好. 因此这里推荐使用 json 格式 + jq 命令 来完成. kv的查表&映射. 如下脚本:

```bash
#!/bin/bash

# 预定义JSON数据
json_data='{
    "cn-beijing": "cn-beijing-btc-a01",
    "cn-hangzhou": "cn-hangzhou-dg-a01"
}'

# 获取输入参数并进行翻译
translate() {
  word=$1
  translated_word=$(echo "$json_data" | jq -r --arg word "$word" '.[$word]')
  if [ "$translated_word" != "null" ]; then
    echo "$translated_word"
  else
    echo "无法翻译：$word"
  fi
}

# 测试脚本
translate "$1"
```

但注意, 上边脚本只是把对应的value echo出来, 并没有赋值到workflow变量里. 如果需要放到变量里, 需要继续看第二步. 

### 2. 如何将value保存在新的workflow的变量中?

要注意的是, echo出来的一定要是JSON类型, 且kv需要符合alfred的规范. 参见文章: [Workflow/environment variables in Alfred — deanishe.net](https://www.deanishe.net/post/2018/10/workflow/environment-variables-in-alfred/)

详细规范&格式定义如下: 

1. 如果是 **From Run Script actions 类型, 则脚本里输出格式需要为: (通过echo/puts来输出)**

```json
{
    "alfredworkflow":
    {
        // "arg": "https://www.google.com", // 注意输出的args, 后边是通过 {query} 来进行引用. 个人建议不要使用这种hidden的方式. 建议都显式通过{var:xxx}来引用.
        "variables":
        {
            "region": "$translated_word" // 后续可以使用
        }
    }
}
```

1. 如果是 From Script Filters 类型, 则脚本里输出格式需要为: **(通过echo/puts来输出)**

```json
{
    "items":
    [
        {
            "title": "$translated_word",
            // "arg": "$translated_word",
            "variables":
            {
                "region": "$translated_word"
            }
        }
    ]
}
```

### 3. 完整样例

1. **From Run Script actions 样例**

```bash
#!/bin/bash

# 预定义JSON数据
json_data='{
    "cn-beijing": "cn-beijing-btc-a01",
    "cn-hangzhou": "cn-hangzhou-dg-a01"
}'

# 获取输入参数并进行翻译
translate() {
  regex=$1
  translated_words=$(echo "$json_data" | jq -r "to_entries[] | select(.key | test(\"$regex\")) | {\"arg\": .value, \"variables\": {\"region\": .value}}")
  if [ -n "$translated_words" ]; then
    echo "{ \"alfredworkflow\": $translated_words }"
  else
    echo '{ "alfredworkflow": { "arg": "无法翻译", "title": "无法翻译", "subtitle": "无法翻译" } }'
  fi
}

# 测试脚本
translate "$1"
```

1. From Script Filters 样例

```bash
#!/bin/bash

# 预定义JSON数据
json_data='{
    "cn-beijing": "cn-beijing-btc-a01",
    "cn-hangzhou": "cn-hangzhou-dg-a01"
}'

# 获取输入参数并进行翻译
translate() {
  word=$1
  translated_word=$(echo "$json_data" | jq -r --arg word "$word" '[{ "arg": $word, "title": .[$word], "subtitle": .[$word], "variables": {"region": .[$word]}}]')
  if [ "$translated_word" != '[{"arg":null,"title":null,"subtitle":null}]' ]; then
    echo "{ \"items\": $translated_word }"
  else
    echo '{ "items": [{ "arg": "无法翻译", "title": "无法翻译", "subtitle": "无法翻译" }] }'
  fi
}

# 测试脚本
translate "$1"
```

## 如何添加自定义日期参数?

例如 最终需要的url 是 [`https://www.xxx.com/query={query}&startTime=xxx&endTime=xxx`](https://www.xxx.com/query={query}&startTime=xxx&endTime=xxx) 格式

需要默认 

1. startTime是 now() - 3天;
2. endTime是now()
3. 且格式是 `2023-01-01 12:20:20`

### 1. 添加自定义参数

如下图,  增加 `Arg and Vars` 组件: 

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/2023-11-07-alfred-workflow/Untitled-2.png)

### 2. 设置时间格式:

如上图, `{isodate +1d:yyyy-MM-dd hh:mm:ss}` 代表1天以后, 格式是 2023-01-01 12:12:12 的格式

完整的时间格式, 参见: [Dynamic Placeholders - Alfred Help and Support](https://www.alfredapp.com/help/workflows/advanced/placeholders/#date-time)

### 3. 使用自定义参数:

如下图, 使用 `{var:starTime}` 的方式引用变量

![Untitled](https://davywalker-bucket.oss-cn-shanghai.aliyuncs.com/source/assets/2023-11-07-alfred-workflow/Untitled-3.png)