---
title: 行为型-解释器模式
date: 2019-11-19 14:46:08
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 规则解析解释器模式

&emsp;&emsp;解释器模式据说是使用场景最少的模式了，但是却在许多的框架中都发现了他的影子。例如el表达式的解析、mybatis解析xml文件、和一些json工具中。但是毕竟大多时候使用的频次会比较的低，所以掌握的并不会很多，而且教程里面的解释器模式实在是太抽象了，看了简直一脸懵逼...在介绍设计模式的时候就说过，如果能找到比较合适的场景运用设计模式（先不说好坏），写出能够工作的demo，就算是入门这个模式了，说实在这个模式也只是了解到了皮毛，因为他的缺点十分的明显，维护困难，还可能引起效率问题，一般来说很少使用。但是了解一下还是可以的。还是先从一个简单的demo开始也许例子会比较勉强...

<!-- more -->

&emsp;&emsp;在使用数据库的时候，常常会有一个需求，就是分页查询，这个太多了，做过erp系统的肯定不会不知道。但是，每种数据库都拥有自己的语法，oracle的语法和mysql的语法就不太一样，在编码中数据库语法的差异被称之为数据库的方言。在分页组装sql的时候，需要根据当前数据库的类型来选择方言。介绍一下解释器中的成员：
1. 环境角色Context
2. 抽象解释器
3. 终结符解释器
4. 非终结符解释器

<1> 还是比较好理解的，就是存储一些环境变量，当前环境的一些参数等，一般来说维护一个map就行了。
<2> 就是解释器的父类，定义了一个抽象方法让解释器去实现，也比较好理解。
<3> <4> 就比较难理解了，他们都是抽象解释器的实现,但是非终结符解释器实际上需要依赖终结符解释器来完成解释,类似这样一个公式R = R1 + R2;那么解释R1 和 R2的表达式的解释器就是终结符解释器，而解释 + 的解释器就是非终结符解释器。
再说说终结符和非终结符的概念，他们的说法因该是来自编译原理`编译原理`。简单说终结符就是不能单独出现在表达式左边的元素，对其不可继续拆分。而非终结符是一个可拆分的元素，并且可在分析。在本例子中假设我们要分页的sql是：
``` sql
select * from t_user where user_state = 1 order by user_id desc
```
可以理解为其中的关键字 `where` 、`order by`都是终结符，他们有明确的语义，不能再推导了。而另一些元素如 `*` 它是可拆分，`t_user`它是可推导（可将其他参数带入）。那么我们就形成了一个公式：
``` bash
SELECT select_expr [,select_expr...]
[
FROM table_references
[WHERE where_condition]
[GROUP BY {col_name | postion}  [ASC|DESC],...]
[HAVING where_condition]
[ORDER BY {col_name | expr | position} [ASC | DESC],...]
[LIMIT { [offset,] row_count | row_count OFFSET offset } ]
]
整个select公式 = 终结符 + 非终结符 + ...
```
在这里我们使用终结符解释器先解析出sql中的终结符，将非终结符作为之后的非终结符解释器的入参再解析；

第一步准备好环境
``` java
public class Context {
    private Map<String, String> env;

    public Context() {
        this.env = new HashMap<>(); 
    }

    public Map<String, String> getEnv () {
        return this.env;
    }
}
```
其次是抽象解释器
``` java
public abstract class AbstractExpression {
    public abstract String interpret(Context ctx);
}
```
接下来是终结符解释器, 解析一些通用的东西，如果是表达式的话，可能解析的是符号。这里我们解析sql中的元素
``` java
public class TerminalExpression extends AbstractExpression {

    @Override
    public String interpret (Context ctx) {
        Map<String, String> env = ctx.getEnv();
        //获取当前sql
        String sql = env.get("sqlSource");
        //看看是 CURD中的哪一种。。算了这里尽量极简模式就不考虑分页
        env.put("sqlType", "select");
        //解析select后的column
        env.put("column", "select之后的逗号分隔字段");
        //解析from ，将表名放入
        env.put("tableName", "FROM 后的第一个字段")
        //看看有没有where order by啥的
        if((int i = sql.indexof("where")) != -1) {
            env.put("where", "where后的子句");
        }
        ....
    }
}
```

然后是mysql 和 oracle的解释器
``` java
public class MysqlExpression extends AbstractExpression {

    private AbstractExpression expression;

    public MysqlExperssion(AbstractExpression ep) {
        this.expression = ep;
    }

    @Override
    public String interpret(Context context) {
        //先使用终结符表达式完成对sql的初步解析
        ep.interpret(context);
        //然后对于mysql我们要得到这样的语句
        //select * from t_user where user_state = 1 order by user_id desc limit ${start}, ${rowCount}
        //虽然这里看起来就是再最后品格limit 因为这里的sql过于简单，一个sql解析器可不是这样就能完成的，我们通过上一步的解析构造mysql的分页sql NPE异常为了简化就不处理了, 假设下面的拼接是StringBuilder
        Map<String, String> sqlMap = context.getEnv();
        return String.valueOf(sqlMap.get("sqlType")).contact(sqlMap.get("column").contact("LIMIT").contact(sqlMap.get("startPage")).contact(",").contact(sqlMap.get("pageSize"));
    }
}

public class OracleExpression extends AbstractExpression {
    
    private AbstractExpression expression;

    public MysqlExperssion(AbstractExpression ep) {
        this.expression = ep;
    }

    @Override
    public String interpret(Context context) {
        e.interpret(context);
        //对于oracle来说，我们需要这样的sql
        //select *
        //    from (select rownum rw,a.*
        //            from (
        //            select * from t_user where user_state = 1 order by user_id desc
        //            ) a
        //            where rownum < ${start+rowCount}) b
        //     where b.rw > ${start}
        String sql = "select * from (select rownum rw, a.* from "+ ") a ..."；
        return sql; 
    }
}
```
在实际的sql解析中，情况比这要复杂太多了，涉及的解释器也不会是这么简单，并且sql中的终结符都需要单独解析等等。这里就简单演示逻辑, 最后调用它,
``` java
public static void main(String[] args) {

    //初始化环境
    Context context = new Context();
    //设置方言
    context.getEnv().put("dialect", "mysql");
    //分页参数
    context.getEnv().put("startPage", "0");
    context.getEnv().put("pageSize", "10");
    //当前sql
    context.getEnv().put("sqlSource", "select * from t_user where user_state = 1 order by user_id desc");

    AbstractExpression te = new TerminalExpression();
    //这里可以根据设置的方言来获取合适的解释器，类似策略模式这里就简单来
    String mysqlSql = new MysqlExpression(te).interpret(context);

    String oracleSql = new OracleExpression(te).interpret(context);
}
```

一个超级简易的sql解析器（目前完全不能用！）就完成了...对于解释器模式，可能会引起类膨胀，对于非终结符解释器需要维护类中逻辑，还要维护和终结符解释器之间的关系（不过也可以没有非终结符解释器），其实对于终结符非终结符的也不用那么拘泥，毕竟设计模式的形成，一定是现有代码的基础，逐渐形成的一定套路，而适合场景的模式，可能往往需要一些变形或者是模式互相组合。固定使用模式，可能会显得生硬而且可能会有不太舒服的感觉（肯定是有体会的啦）...