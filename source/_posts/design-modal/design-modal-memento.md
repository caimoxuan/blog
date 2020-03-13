---
title: 行为型-备忘录模式
date: 2019-11-20 15:28:52
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---


## 后悔药备忘录模式

&emsp;&emsp;备忘录模式简单来说就是将过去的操作备份，以便能回顾。日子一天天的过，每天搞一个事项清单，然后当你某一天想知道那件事做了没有，翻一翻这个清单就知道了。它的使用场景应该是最能理解的那种了，对于编码来说 `Ctrl + z`应该没有人没有用过吧...它大量的出现在下棋的悔棋（和ai下才行，和人下谁和你悔...），网页的回退、git的版本控制等等。简单的流程就是在操作的时候，将每一步所操作的对象的当前快照按步骤保存下来，以便想要回退的时候可以按快照还原成原来的样子。

<!-- more -->
那么这样的模式理解起来应该也毫不费力，首先还是认识一下备忘录模式中的几个角色：
1. Originator 发起人，负责定义哪些属于备忘范围、创建恢复备忘数据；
2. Memento 备忘录， 负责存储发起人的状态；
3. Caretaker 管理员， 负责对备忘录进行管理，使得发起热不直接与备忘录耦合，就直接想象成版本数据库；

由于角色比较简单，接下来还是以简单的场景来理解备忘录模式，代码管理器的形式比较主动，就以它为例，git作为现在主流的代码管理软件，他的管理代码的形式就类似于备忘录模式：

首先定义一个备忘录，git管理的代码其实都是文件，每一个提交都有一个commitId 和 当前版本的快照，这个快照里面就是所有的代码文件：
``` java
//用lombok吧 实在懒得写了...想着这样引用不了包... 能看就行了
@Data
public class GitMemento {

    private String commitId;
    //我们用字符代替文件
    private List<String> codeFiles;

    public GitMemento(String commitId, List<String> initState) {
        this.codeFiles = initState;
        this.commitId = commitId;
    }

    public List<String> getMemento() {
        return this.codeFiles;
    }
} 
```
然后是管理员，因为发起人要用到管理员。管理员管理了所有历史版本；
``` java
@Data
public class CareTaker {

    private Map<String, List<String>> versionMap;

    public CareTaker() {
        //因为提交版本显示的时候是按提交顺序的 实际会记录时间戳 这里简单来
        this.versionMap = new LinkedHashMap<>();    
    }

    public GitMemento get(String commitId) {
        return versionMap.get(commitId);
    }

    public void add(Memento m) {
        versionMap.put(m.getCommitId(), m);
    }

}

```

接下来是发起人，就类似于git工具的客户端一样，当用户使用`git commit`提交代码的时候，客户端工具就会将本次提交保存，生成commitId发起备忘录的创建。当想要返回到原来的某个版本的时候 `git reset --head ${commitId}` 来返回版本;
``` java
@Data
public class GitOriginator {

    //当前的版本文件
    private List<String> codeFiles;

    //找到一个管理员
    private GitCareTaker careTaker;

    public GitOriginator(GitCareTaker ck) {
        codeFiles = new ArrayList<>();
        this.careTaker = ck;
    }

    //这里我们就实现2功能 提交， 回退
    public String commit () {
        String commitId = UUID.random().toString();
        cakeTaker.add(new GitMemento(commitId, codeFiles));  
        return commitId;
    }

    public List<String> reset(String commitId) {
        return cakeTaker.get(commitId); 
    }

}
```

这样我们就完成了一个非常简单的个人git的超级阉割版就完成了...接下来试用一下：

``` java
public static void main(String[] args) {
    //管理员先联系好
    GitCareTaker careTaker = new GitCareTaker();
    //假装写代码
    String code1 = "bug1"；
    String code2 = "bug2";
    //写完了提交
    GitOriginator og = new GitOriginator(careTaker);
    //代码动态git是能监控的，我们这个当然没有这个功能啦 只能手动处理
    og.getCodeFiles().add(code1);
    og.getCodeFiles().add(code2);
    String commitId1 = og.commit();

    //修改代码
    code1 = "bug fix1";
    code2 = "bug fix2";
    //然后重新提交， 提交前我们需要清空下发起人手里的原稿，还是因为我们没有文件监控功能
    og.getCodeFiles().clear();
    og.getCodeFiles().add(code1);
    og.getCodeFiles().add(code2);
    String commitId2 = og.commit();

    //我们现在有的是当前修改完bug的代码，现在我想看看bug...
    List<String> bugFiles = og.reset(commitId1);
    bugFiles.foreach(System.out::println);
}
```

当然这个git过于简单了，但是照着这个思路搞一个git服务端应该还是可以的。类似于这样，我们也许需要回退，回顾以往的这样的场景，还是比较适合备忘录模式，可以发现模式真的和生活密不可分，但是生活没有后悔药可以吃，如果能把人生的每一天存起来，那天日子过不下去了，就回到最快乐的那一天（也去再过一次就体会不到那样的快乐了），哪怕只是去看一看...算了，好了这样一看备忘录模式的缺点就暴露出来了。现在记录过往能够将记忆保存起来的东西可以是视频，但是如果要记录每一天，一天在活动的时候（除了睡觉）有大概12-18个小时吧，一个2小时的视频大概在3G的样子（清晰度稍高点的话），那么这样算一天大概需要20G的样子。然后按照hdfs（分布式文件系统-hadoop）的最低要求备份2份就是60G，即使压缩一下也不小，这是存储成本。然后还要有一个人或物来拍摄等等...好了总结下来就是备忘录模式虽然能够充当计算机世界的后悔药，但是他需要很多额外的成本来存储这些副本，所以在使用的时候要考虑快照的压缩，甚至内存泄漏的风险，但是真的好用啊，想想如果IED没有撤销功能...git没有回退功能...