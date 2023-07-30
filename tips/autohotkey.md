---
title: AutoHotKey使用技巧
date: 2022-08-14
tags: [tips,AutoHotKey]
---

## AutoHotKey

强烈推荐 AutoHotKey 这个小东西，可以快速在 typora 中输入 html 代码，因为在 typora 中是支持 html 的，比如高亮，在 typora 中使用 `==` 可显示出来；

```
==高亮内容==
```

使用 AutoHotKey 脚本的话，设置好了快捷键之后选中需要高亮的内容可以直接生成相应的 html 代码，还可以进一步设置背景颜色，字号大小等等；

```
<font size=4 style="font-weight:bold;background:yellow;">高亮内容</font>
```

<!--more-->

最重要的是在 hexo 中不支持 `==` 这样的高亮样式显示，这样在本地 typora 写完之后上传到网页上高亮显示不出来，AutoHotKey 完美解决！

- `==高亮内容==`：==高亮内容==
- 使用 html：<font size=4 style="font-weight:bold;background:yellow;">高亮内容</font>

**AutoHotKey 不只能做这些，定义好脚本之后，基本上任何的样式只要想得到，都能通过快捷键设置出来：**

<font color='red' style="font-weight:bold;">test</font>

<font color='orange' style="font-weight:bold;">test</font>

<font color='yellow' style="font-weight:bold;">test</font>

<font color='green' style="font-weight:bold;">test</font>

<font color='cyan' style="font-weight:bold;">test</font>

<font color='cornflowerblue' style="font-weight:bold;">test</font>

<font color='purple' style="font-weight:bold;">test</font>

```
; 分号以及分号后的内容代表注释，以下为代码解释
#IfWinActive ahk_exe Typora.exe
{
	
    ; alt+1 红色
    !1::addFontColor("red")
	
    ; alt+2 深蓝色
    !2::addFontColor("#0000ff")
	
    ; alt+3 深绿色
    !3::addFontColor("#198437")
	
    ; alt+4 深紫色
    !4::addFontColor("#924193") 
	
	; ctrl+q 三级标题
	^q::addBackgroundColor()
	
	; alt+d 获取当前时间
	!d::GetCurrentTimeStamp()
	
	::/head::---`ntitle: `ndate: `ntags: []`ntoc: true
	
	::/code::{{}% spoiler 展开查看折叠代码 %{}}`n/java`n{{}% endspoiler %{}}
	::/cs::{{}% spoiler 展开查看折叠代码 %{}}
	::/ce::{{}% endspoiler %{}}

	::/java::``````java
	::/sql::``````sql
	::/bash::``````bash
	
	

}

; 快捷增加字体颜色
addFontColor(color){
    clipboard := "" 					; 清空剪切板
    Send {ctrl down}c{ctrl up} 			; 复制
    ; SendInput {Text} 					; 解决中文输入法问题
    SendInput {TEXT}<font color='%color%' style="font-weight:bold;">
    SendInput {ctrl down}v{ctrl up}		; 粘贴
    If(clipboard = ""){
        SendInput {TEXT}</font> 		; Typora 在这不会自动补充
    }else{
        SendInput {TEXT}</ 				; Typora中自动补全标签
    }
}
; 增加字体颜色，增加字体背景颜色
addBackgroundColor(){
    clipboard := "" 					; 清空剪切板
    Send {ctrl down}c{ctrl up} 			; 复制
    ; SendInput {Text} 					; 解决中文输入法问题
    SendInput {TEXT}<font size=4 style="font-weight:bold;background:yellow;">
    SendInput {ctrl down}v{ctrl up}		; 粘贴
    If(clipboard = ""){
        SendInput {TEXT}</font> 		; Typora在这不会自动补充
    }else{
        SendInput {TEXT}</ 				; Typora中自动补全标签
    }
}
; 获取当前的时间戳精确到秒
GetCurrentTimeStamp(){
    send %A_YYYY%-%A_MM%-%A_DD%%A_SPACE%
	ClipTemp = %Clipboard%
	FormatTime , Clipboard , , HH':'mm
	Send ^v
	Sleep 100
	Clipboard = %ClipTemp%
	Return
}
```

除了在笔记中设置样式之外，AutoHotKey 还有很多其他的骚操作，以后有用到的会同步更新在这里~



**8.15 更新：添加折叠代码块，效果：**

<details><summary><font color='blue' style="font-weight:bold;">展开查看折叠代码块</font></summary><pre><code>public class Hello{
    public static void main(String[] args) {
        System.out.println("Hello World! ");
    }
}</code></pre></details>







代码折叠测试



{% spoiler mybatisplus代码测试 %}

```java
package com.haining820;

import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.haining820.entity.Student;
import com.haining820.mapper.StudentMapper;
import com.haining820.utils.TimeUtil;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@SpringBootTest
class MybatisplusStudyApplicationTests {

    @Autowired
    private StudentMapper studentMapper;

    private static Logger LOGGER = LoggerFactory.getLogger(MybatisplusStudyApplicationTests.class);

    @Test
    public void testInert() {
        Student student = new Student("灰太狼", 37, "testInert-> " + TimeUtil.getCurTime());
        int result = studentMapper.insert(student);
        LOGGER.info("testInert,result->" + result);
    }


    @Test
    public void testUpdate() {
        Student student = new Student();
        student.setId(5);
        student.setAge(32);
        student.setName("张三");
        int result = studentMapper.updateById(student);
        LOGGER.info("testUpdate,result->" + result);
    }

    @Test
    public void testSelect() {
        // 参数是一个条件构造器Wrapper，这里先设置为null
        List<Student> studentList = studentMapper.selectList(null);
        for (Student s : studentList) {
            LOGGER.info(s.toString());
        }
    }

    @Test
    public void testOptimistic() {
        // 查询用户信息
        Student student = studentMapper.selectById(2);
        LOGGER.info(student.toString());
        // 更新用户信息
        student.setName("yhyy");
        student.setContent("test version");
        // 执行更新操作
        studentMapper.updateById(student);
    }


    @Test
    public void testOptimisticUnderMultiThread() {
        Student student = studentMapper.selectById(2);
        student.setName("yhyy11");
        student.setContent("test version11");

        Student student2 = studentMapper.selectById(2);
        student2.setName("yhyy22");
        student2.setContent("test version22");
        studentMapper.updateById(student2);

        studentMapper.updateById(student);
    }

    @Test
    public void testOptimisticUnderMultiThread2() {

        new Thread(() -> {
            Student student = studentMapper.selectById(6);
            student.setName("yhyy in AAA");
            student.setAge(student.getAge() + 10);
            student.setContent("test version in AAA");
            int i = studentMapper.updateById(student);
            LOGGER.info(Thread.currentThread().getName() + "->" + i);
        }, "A").start();

        new Thread(() -> {
            Student student = studentMapper.selectById(6);
            student.setName("yhyy in BBB");
            student.setAge(student.getAge() - 3);
            student.setContent("test version in BBB");
            int j = studentMapper.updateById(student);
            LOGGER.info(Thread.currentThread().getName() + "->" + j);

        }, "B").start();

        Student student = studentMapper.selectById(6);
        student.setName("yhyy in main");
        student.setAge(student.getAge() + 80);
        student.setContent("test version in main");
        int k = studentMapper.updateById(student);
        LOGGER.info(Thread.currentThread().getName() + "->" + k);
    }

    @Test
    public void testPage() {
        // 设置当前页和页面大小
        Page<Student> page = new Page<>(1, 3);
        // queryWrapper参数与高级查询有关
        studentMapper.selectPage(page, null);
        for (Student s : page.getRecords()) {
            LOGGER.info(s.toString());
        }
        LOGGER.info("page.getTotal()->" + page.getTotal());
    }

}
```

{% endspoiler %}