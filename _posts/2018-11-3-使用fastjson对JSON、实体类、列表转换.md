---
layout:     post
title:      使用fastjson对JSON、实体类、列表转换
subtitle:   使用fastjson对JSON、实体类、列表转换
date:       2018-11-3
author:     LY
header-img: img/post-bg-debug.png
catalog: true
tags:
    - java
    - ajax
    - JSON
---

> 在这里小小推荐下我的个人博客
>
> csdn：[雷园的csdn博客](https://blog.csdn.net/leiyuan2580)
>
> 个人博客：[雷园的个人博客](https://imlcl.store)
>
> 简书：[雷园的简书](https://www.jianshu.com/u/016322e40e1f)
>

# 使用fastjson对JSON、实体类、列表转换

#### 导入jar包或者使用maven添加依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.49</version>
</dependency>
```



#### 首先我们来写一个带有列表的实体类：

```java
package com.entity;

import java.util.List;

public class Teacher {
    private String id;
    private String name;
    private String info;
    private List<Student> studentList;

    public static class Student {
        private String id;
        private String name;
        private String student_num;

        @Override
        public String toString() {
            return "Student{" +
                    "id='" + id + '\'' +
                    ", name='" + name + '\'' +
                    ", student_num='" + student_num + '\'' +
                    '}';
        }

        public String getId() {
            return id;
        }

        public void setId(String id) {
            this.id = id;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public String getStudent_num() {
            return student_num;
        }

        public void setStudent_num(String student_num) {
            this.student_num = student_num;
        }

        public Student(String id, String name, String student_num) {

            this.id = id;
            this.name = name;
            this.student_num = student_num;
        }
    }

    @Override
    public String toString() {
        return "Teacher{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", info='" + info + '\'' +
                ", studentList=" + studentList +
                '}';
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getInfo() {
        return info;
    }

    public void setInfo(String info) {
        this.info = info;
    }

    public List<Student> getStudentList() {
        return studentList;
    }

    public void setStudentList(List<Student> studentList) {
        this.studentList = studentList;
    }

    public Teacher(String id, String name, String info, List<Student> studentList) {

        this.id = id;
        this.name = name;
        this.info = info;
        this.studentList = studentList;
    }
}

```

#### 你收到的JSON数据格式如下

```json
{
    "id":"1",
    "name":"your_name",
    "info":"你是一个好老师",
    "studentList":[
        {
            "id":"1",
            "name":"student_name1",
            "student_num":"134233721"
        },
        {
            "id":"2",
            "name":"student_name2",
            "student_num":"134233722"
        },
        {
            "id":"3",
            "name":"student_name3",
            "student_num":"134233723"
        },
        {
            "id":"4",
            "name":"student_name4",
            "student_num":"134233724"
        }
    ]
}
```

#### 接下来进行格式转换

```java
package com.controller;

import com.alibaba.fastjson.JSON;
import com.leiyuan.entity.Teacher;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller
@RequestMapping("/test")
public class TestController {

    /**
     * 在这里进行转换
     *
     * @param json 假设json为请求参数
     */
    @RequestMapping(value = "/transJson", method = RequestMethod.POST)
    public void transJson(@RequestBody String json) {
        System.out.println("原始数据==========" + json);
        // 转化为teacher实体类
        Teacher teacher = JSON.parseObject(json, Teacher.class);
        System.out.println("转换为teacher实体类==========" + teacher.toString());
        // 对teacher实体类中的studentList进行转换
    }
}

```

#### 输出结果如下

```java
原始数据=========={
	"id":"1",
	"name":"your_name",
	"info":"你是一个好老师",
	"studentList":[
		{
			"id":"1",
			"name":"your_name",
			"student_num":"134233721"
		},	
		{
			"id":"2",
			"name":"your_name",
			"student_num":"134233722"
		},
				{
			"id":"3",
			"name":"your_name",
			"student_num":"134233723"
		},
				{
			"id":"4",
			"name":"your_name",
			"student_num":"134233724"
		}
	]
}
转换为teacher实体类==========Teacher{id='1', name='your_name', info='你是一个好老师', studentList=[Student{id='1', name='your_name', student_num='134233721'}, Student{id='2', name='your_name', student_num='134233722'}, Student{id='3', name='your_name', student_num='134233723'}, Student{id='4', name='your_name', student_num='134233724'}]}
```

#### 如何将实体类转换为JSON数据？

```java
// 直接调用如下方法
JSON.toJSONString(your_class);
```

