---
layout: post
title: "fastjson 反序列化时对kotlin 字段默认值的处理"
description: 
image: 
category: 'blog'
tags:
- kotlin
- fastjson
- byteCode
---

#### fastjson 如何进行反序列化
- 为每一个反序列化类，动态生成一个反序列化器
    - ASM
    - defineClass
    - 源码如下：
        - ![](../assets/img/dynamic-class.png)

#### 反序列化过程
jsonString ——> FastjsonASMDeserializer_x_xxx.class ——> JavaBean

FastjsonASMDeserializer_x_xxx.class 为fastjson生成的字节码

#### 如何去探究其反序列化过程
- 调试源码？
    - 动态生成的类，IDEA没有符号。不可调  
    - 反编译 FastjsonASMDeserializer_x_xxx.class，查看不同kotlin类对应的 FastjsonASMDeserializer_x_xxx.class 的反序列化逻辑 
    查看deserialze 方法的逻辑，发现没有针对JavaBean的特定逻辑差异 （此路不通）
- arthas
    - sc,sm,watch,trace,stack,tt  等命令查找class文件，根据方法执行路径  
    [arthas@1]$ trace com.alibaba.fastjson.parser.deserializer.FastjsonASMDeserializer_12_QuestionDetailInfo deserialze -n 2  
Press Q or Ctrl+C to abort.  
Affect(class count: 1 , method count: 1) cost in 577 ms, listenerId: 8  
`---ts=2020-06-11 17:27:12;thread_name=DubboServerHandler-172.30.105.162:20880-thread-192;id=16b;is_daemon=true;priority=5;TCCL=org.springframework.boot.loader.LaunchedURLClassLoader@a7e666        
    `---[0.97367ms] com.alibaba.fastjson.parser.deserializer.FastjsonASMDeserializer_12_QuestionDetailInfo:deserialze()    
        +---[0.036864ms] com.alibaba.fastjson.parser.JSONLexerBase:token() #-1  
        +---[min=0.003817ms,max=0.007004ms,total=0.010821ms,count=2] com.alibaba.fastjson.parser.JSONLexerBase:isEnabled() #-1  
        +---[0.0119ms] com.alibaba.fastjson.parser.JSONLexerBase:scanType() #-1  
        +---[min=0.007255ms,max=0.037986ms,total=0.045241ms,count=2] com.alibaba.fastjson.parser.DefaultJSONParser:getContext() #-1  
        **+---[0.018674ms] com.seewo.classroom.core.entity.course.QuestionDetailInfo:\<init\>() #-1**    
        +---[min=0.007506ms,max=0.033038ms,total=0.040544ms,count=2] com.alibaba.fastjson.parser.DefaultJSONParser:setContext() #-1  
        +---[min=0.004043ms,max=0.022455ms,total=0.026498ms,count=2] com.alibaba.fastjson.parser.JSONLexerBase:scanFieldString() #-1  
        +---[min=0.003605ms,max=0.00713ms,total=0.026378ms,count=6] com.alibaba.fastjson.parser.JSONLexerBase:scanFieldInt() #-1  
        +---[min=0.004841ms,max=0.023299ms,total=0.02814ms,count=2] com.alibaba.fastjson.parser.JSONLexerBase:scanFieldStringArray() #-1  
        +---[0.01117ms] com.seewo.classroom.core.entity.course.QuestionDetailInfo:setAnswerBoardUid() #-1  
        +---[0.006022ms] com.seewo.classroom.core.entity.course.QuestionDetailInfo:setCorrect() #-1  
        +---[0.005935ms] com.seewo.classroom.core.entity.course.QuestionDetailInfo:setCorrectNumber() #-1  
        +---[0.032129ms] com.seewo.classroom.core.entity.course.QuestionDetailInfo:setCorrectOptions() #-1  
        +---[0.005944ms] com.seewo.classroom.core.entity.course.QuestionDetailInfo:setCorrectRate() #-1  
        +---[0.006962ms] com.seewo.classroom.core.entity.course.QuestionDetailInfo:setOptions() #-1  
        +---[0.005242ms] com.seewo.classroom.core.entity.course.QuestionDetailInfo:setQuestionNumber() #-1  
        +---[0.007674ms] com.seewo.classroom.core.entity.course.QuestionDetailInfo:setSlideHash() #-1  
        `---[0.005561ms] com.seewo.classroom.core.entity.course.QuestionDetailInfo:setTotalStudentNumber() #-1   
    - 对比不同kotlin类的构造函数的区别
    ```
    data class QuestionDetailInfo(
        var answerBoardUid:String,//答题板id
        var type:AnswerBoardType=AnswerBoardType.OBJECTIVE, //答题板类型
        var correctOptions:List<String> = emptyList(),//正确选项
        var options:List<String>?=null,//所选选项
        var correct:Int=0,//是否回答正确 0:没答  1:正确  2:错误
        var questionNumber:Int=0,//题号
        var correctRate:Int=0,
        var correctNumber:Int=0,
        var totalStudentNumber:Int=0,
        var answerStudentNumber:Int=0,
        var slideHash:String?=null) 
    ```    
    与
    ```
    data class SeewoWxUserBindTopicDTO(
        var userId: String = "",
        var openId: String = "",
        var extendParam: String = "")
    ```


- 查看class文件
    - javap -verbose class文件
    - 通过class 文件 查看 QuestionDetailInfo 与 SeewoWxUserBindTopicDTO 类的 构造函数  

    ```
    public com.seewo.classroom.core.dto.weixin.SeewoWxUserBindTopicDTO();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=6, locals=1, args_size=1
         0: aload_0
         1: aconst_null
         2: aconst_null
         3: aconst_null
         4: bipush        7
         6: aconst_null
         7: invokespecial #48                 // Method "<init>":(Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;ILkotlin/jvm/internal/DefaultConstructorMarker;)V
        10: return

    ``` 
    

    ```   
    public com.seewo.classroom.core.entity.course.QuestionDetailInfo();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #84                 // Method java/lang/Object."<init>":()V
         4: return
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lkotlin/Unit;
    ```  

    **发现这两个类的 默认构造函数实现不一样** 这最终导致了反序列化时，对默认值的处理不同
    



-----
