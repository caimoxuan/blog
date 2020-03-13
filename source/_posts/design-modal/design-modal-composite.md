---
title: 结构型-组合模式
date: 2019-11-22 09:18:09
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 树形控件组合模式

&emsp;&emsp;组合模式的结构可以说是有非常明显的特征，结构非常鲜明。做过用户中心类似这样的需求的人来说，这样的机构一定不会陌生。在开发一个ERP系统的时候，第一步要做的就是登录，其次是角色控制、权限管理、再者是组织架构、数据隔离。一般一个用户属于一个组织，组织具有上下层级关系，并且层级的深度又不确定，就好似一颗树一般。

<!-- more -->
&emsp;&emsp;首先想象二叉树，是不是有一个根节点，由根节点向下衍生，每个节点最多有2个子节点。组织机构就和这类似，不同的是，每个组织的子组织可以有任意多个子节点，像文件目录一样。接下来就是具体实现，首先看一张图，是ant design的树形控件。组合模式可以说是与之完美结合。

![tree](tree.png)

首先我们定义好一个组织，需要注意的是组织模式中的元素是具体的，很少由抽象，因为每一节点的元素都是一样的，只不过是层级不同。
``` java
@Data
public class Organization {

    //节点的id
    private Integer key;
    //节点的名称
    private String name;
    //父节点的id
    private Integer parentKey;

    public Organization(Integer key, Integer parentKey, String name) {
        this.key = key;
        this.parentKey = parentKey;
        this.name = name;
    }

    //节点的子节点 (这就是组合模式的主体，而这个列表中的对象也可能是其他类型，这里是同一类型)
    private List<Organization> children;
}
```

说起来我们的组合模式就已经完成了，但是组合模式难点不是在实现，而是在使用。一般来说，组合模式都是和递归一起出现，而这难点主要来源于递归的使用。下面模拟一下数据实现上面的树形控件。
``` java
public static void main(String[] args) {
    //我们的数据库中存储的数据是不会包含子节点的，现在我们从数据库中查询出所有的机构（实际的实现都是异步逐级加载的不然数据太大）
    Organization root = new Organization(1, 0, "root");
    Organization level1 = new Organization(2, 1, "level1");
    Organization level11 = new Organization(3, 1, "level11");
    Organization level2 = new Organization(4, 2, "level2");
    Organization level22 = new Organization(5, 2, "level22");
    List<Organization> orgs = Arrays.asList(root, level1, level11, level2, level22);

    //假设上面是数据库中查询的数据，一共有5个节点，界限来需要转换成一个树形控件,我们还差每个节点的子节点的组装,我们需要一个递归方法;
    parseOrg(orgs, 0);
}

public static List<Organization> parseOrg(List<Organization> orgs, Integer parentKey) {
    //新列表报错结果
    List<Organization> results = new ArrayList<>();

    for(Organization o : orgs) {
        if(parentKey.equals(o.getParentKey()) {
            //因为在组装根节点的子节点的时候先要组装子节点的子节点(以当前节点的key寻找子节点)
            List<Organization> children = parseOrg(orgs, o.getKey());
            o.setChildren(children);
            results.add(o);
        }
    }

    return results;
}
```

然后我们就得到了这样的数据(json格式)：
``` json
[
    {
        "key":1,
        "name":"root",
        "parentKey":0,
        "children":[
            {
                "key":2,
                "name":"level1",
                "parentKey":1,
                "children":[
                    {
                        "key":4,
                        "name":"level2",
                        "parentKey":2,
                        "children":[

                        ]
                    },
                    {
                        "key":5,
                        "name":"level22",
                        "parentKey":2,
                        "children":[

                        ]
                    }
                ]
            },
            {
                "key":3,
                "name":"level11",
                "parentKey":1,
                "children":[

                ]
            }
        ]
    }
]
```
我们把这样的数据作为树形控件的数据源，就能在前端展示一个机构的树形结构了。组合模式的精髓其实就是一个类中持有一个类的列表，一般都用在文件目录、组织机构等场景，具有明显的层级，使用这个模式，可以方便的实现一个ERP系统的组织架构展示和管理。
不过话说类似这样的结构，即使设计者本身不知道组合模式，在开发的时候也是会这样的使用的，毕竟这样的场景使用这个结构是顺其自然的吧。这时候就能体会到，设计模式一定也是来源于编码的经验，绝不是纸上谈兵。如果需要开发这样的权限管理，文件管理等工具，可以在使用的同时，看看组合模式是如何工作的。


