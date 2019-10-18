---
title: Freemarker的基本语法及入门基础
date: 2019-10-18 17:14:09
tags:
---
### 一、freemarker模板文件(*.ftl)的基本组成部分
1. 文本：直接输出的内容部分
2. 注释：不会输出的内容，格式为<#--  注释内容  -->
3. 取值(插值)：代替输出数据模型的部分，格式为${数据模型}或#{数据模型}
4. ftl指令：Freemarker指令，类似于HTML标记。
           内建指令：开始标签:<#directivename parameter>     
                    结束标签:</#directivename>      
                    空标签:<#directivename parameter/>   
           自定义指令：开始标签:<@directivename parameter>       
                      结束标签:</@directivename>      
                      空标签:<@directivename parameter/>
           至于什么是内建指令，什么是自定义指令   我会在下面叙述到。

### 二、Freemarker语法及使用方法
1. 取值(插值)指令及适用类型：

        (1) ${var}
                适用类型：java中常用的八大基本类型以及我们的String引用类型，但是，freemarker中boolean类型显示时true==yes  false==no
                示例：
                        在后台文件中定义变量
                                String strVar = "世界你好";
                                int intVar = 10;
                                boolean booVar = true;
                        在页面中获取变量：
                                String获取：<font color="red"> ${strVar} </font><br>
                                int获取：<font color="red"> ${intVar} </font><br>
                                boolean获取：<font color="red"> ${booVar?string("yes","no")} </font>
                        展示结果：
                                String获取：世界你好
                                int获取：10
                                boolean获取：yes
        (2)${var!}
                适用类型：对 null 或者不存在的对象进行取值，可以设置默认值，例：${var!'我是默认值'}    即，有值时显示正常值，无值时显示默认值
                示例：
                        在后台文件中定义变量
                                String strVar = "世界你好";
                                String str = null;
                        在页面中获取变量：
                                String获取：<font color="red"> ${strVar!"我是空"} </font><br>
                                str获取：<font color="red"> ${str!} </font><br>
                                str获取：<font color="red"> ${str!"默认"} </font><br>
                        展示结果：
                                String获取：世界你好
                                str获取：
                                str获取：默认
        (3)${封装对象.属性}
                适用类型：对封装对象进行取值，例：${User.name}
                示例：
                        在后台文件中封装对象User[ name,  age ]
                                String name = "姓名";
                                int age = 18;
                        在页面中获取变量：
                                name获取：<font color="red"> ${User.name} </font><br>
                                age获取：<font color="red"> ${User.age} </font><br>
                        展示结果：
                                name获取：姓名
                                age获取：18
        (4)${date?String('yyyy-MM-dd')}
                适用类型：对日期格式进行取值，在这里我要强调的是，定义Date类型的变量时，java.util.Date无法输出日期，须使用java.sql.Date
                示例：
                        在后台文件中定义变量
                                java.sql.Date date = new Date().getTime();
                                java.sql.Date time = new Date().getTime();
                                java.sql.Date datetime = new Date().getTime();
                        在页面中获取变量：
                                date获取：<font color="red"> ${date?string('yyyy-MM-dd')} </font><br>
                                time获取：<font color="red"> ${date?string('HH:mm:ss')} </font><br>
                                datetime获取：<font color="red"> ${date?string('yyyy-MM-dd HH:mm:ss')} </font><br>
                        展示结果：
                                name获取：姓名
                                age获取：18
        (5)${var?html}
                适用类型：转义HTML内容
                示例：
                        在后台文件中封装变量Menu[ name, model ]
                                Menu m = new Menu();
                                m.setName(" freemarker ");
                                m.setModel("<font color = 'red'>我只是个菜单</font>");
                        在页面中获取变量：
                                非转义获取：<font color="red"> ${m.model} </font><br>
                                转义获取： ${m.model?html} </font><br>
                        展示结果：
                                非转义获取：我只是个菜单
                                转义获取：<font color = 'red'>我只是个菜单</font>
        (6)<#assign num = 100 />
                适用类型：定义变量，支持计算和赋值
                示例：
                        在页面中定义变量：
                                <#assign num = 100 />
                                num获取：<font color="red"> ${num)} </font><br>
                                计算结果：<font color="red"> ${num * 10} </font><br>
                        展示结果：
                                num获取：100
                                计算结果：1000
        (7)对List集合进行取值
                <#list  list集合  as  item>
                       ${item}    --取值
                </#list>
                示例：
                        在后台文件中定义变量
                                List<String> strList = new ArrayList<String>();
                                strList.add("第一个值");
                                strList.add("第二个值");
                                strList.add("第三个值");
                        在页面中获取变量：
                                <#list  strList  as  item>
                                       ${item!}<br/>    --取值
                                </#list>
                        展示结果：
                                第一个值
                                第二个值
                                第三个值
        (8)对Map集合进行取值
                <#list map?keys as key>
                       ${key}:${map[key]}
                </#list>
                示例：
                        在后台文件中定义变量
                                Map<String, Object> m = new HashMap<String, Object>();
                                m.put("name","姓名");
                                m.put("age",18);
                                m.put("sex","男");
                        在页面中获取变量：
                                 <#list m?keys as key>
                                        ${key}:${m[key]}
                                 </#list>
                        展示结果：
                                name：姓名
                                age：18
                                sex：男
2. 条件判断指令：
 
        (1) if
                格式：<#if 条件>
                                输出
                             </#if>
                示例：
                        在页面中定义变量并判断条件：
                                <#assign age = 18 /><br>
                                <#if age == 18>
                                        <font color="red"> age = 18</font>
                                </#if>
                        展示结果：
                                age = 18
        (2) if - else
                格式：<#if 条件>
                                输出
                            <#else>
                                输出
                            </#if>
                示例：
                        在页面中定义变量并判断条件：
                                <#assign age = 20 /><br>
                                <#if age == 18>
                                        <font color="red"> age = 18</font>
                                <#else>
                                        <font color="red"> age != 18</font>
                                </#if>
                        展示结果：
                                age != 18
        (3) if - elseif - else
                格式：<#if 条件1>
                                输出
                            <#elseif 条件2>
                                输出
                            <#else>
                                输出
                            </#if>
                示例：
                        在页面中定义变量并判断条件：
                                <#assign age = 20 /><br>
                                <#if age &gt; 18>
                                        <font color="red">青年</font>
                                <#elseif age == 18>
                                        <font color="red"> 成年</font>
                                <#else>
                                        <font color="red"> 少年</font>
                                </#if>
                        展示结果：
                                成年
        (4) switch  --常与case break default一起使用  参数可为字符串
                格式：<#switch var>
                            <#case 条件1>
                                  输出
                            <#break>
                            <#case 条件2>
                                  输出
                            <#break>
                            <#default>
                                  输出
                            </#switch>
                示例：
                        在页面中定义变量并判断：
                               <#switch var="星期一">
                               <#case "星期一">
                                       油焖大虾
                               <#break>
                               <#case "星期二">
                                       炸酱面
                               <#break>
                               <#default>
                                       肯德基
                               </#switch>
                        展示结果：
                                油焖大虾
3. 自定义函数、自定义指令：
 
        (1) 自定义函数
                实现TemplateMthodModelEx
        (2) 自定义指令
                实现TemplateDirectiveModel
                示例：
                        <@自定义指令名称 入参(key-value格式) ; 出参(list格式)>
                                运行条件
                        </@自定义指令名称>
    
                        PS:不同的返回值用逗号( , )间隔开
                        

4.常用内建函数、macro(宏指令)、function(函数指令)：

        (1) 常用内建函数
                处理字符串：
                     substring                                          截取字符串，包头不包尾（下标）
                     cap_first                                          第一个字母大写
                     end_with                                           以什么字母结尾    
                     contains                                            是否包含目标字符串
                     date  datetime  time                       转换成日期格式
                     starts_with                                      以什么字母开头
                     index_of                                          返回某个指定的字符串值在字符串中首次出现的位置（下标）
                     last_index_of                                  获取指定字符出现的最后位置（下标）
                     split                                                  分隔
                     trim                                                  去两端空格
                处理数字：
                     string                                              
                     x?string("0.##")                          变成小数点后几位
                     round                                              四舍五入
                     floor                                               去掉小数点
                     ceiling                                             近1   变成整数
                处理list:
                      first:                                              取List值第一个值
                      last:                                                取List值最后一个值
                      seq_contains:                                是否包含指定字符
                      seq_index_of:                               指定字符所在位置
                      size:                                                集合大小
                       reverse:                                          集合倒序排列
                      sort:                                                对集合进行排序
                      sort_by:                                         根据某一个属性排序
                      chunk:                                            分块处理
                其他:
                      is_string:                                      是否为字符类型
                      is_number:                                    是否为整数类型
                      is_method:                                   是否为方法
                      ():                                                  判断整个变量
                      has_content:                                判断对象是否为空或不存在
                      eval：                                           求值
        (2) macro(宏指令)
                调用：<@macro_name param />
                语法：<#macro  变量名  参数>  
                                    <#nested/>  
                            </#macro>
        (3) function(函数指令)
                调用：${function_name(param)}
                语法：<#function  变量名  参数>  
                            <#return>  
                            </#function>
                     