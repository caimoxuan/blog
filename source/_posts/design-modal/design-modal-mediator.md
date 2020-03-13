---
title: 行为型-中介者模式
date: 2019-10-25 18:10:55
thumbnail: /gallery/thumbnail/design_modal.jpg
categories:
    - 架构之路
tags:
    - 设计模式
---

## 棋盘裁判中介者

&emsp;&emsp;为什么中间穿插了一篇mongodb，因为小时后断网的时候玩的翻转棋，实现起来的算法还有点难度，应该是对我来说吧，花了两天时间终于把基本的功能实现了。为什么翻转棋会出现在片中介者模式中呢?个人觉得中介者模式中的最关键的中介者，就像是棋盘中的裁判，狼人杀中的上帝，而中介者模式又被称为调停模式。为了保证游戏的公平性和玩家之间的操作合理性，这样的角色就不可少了。
&emsp;&emsp;中介者模式可以可以将多个对象之间的直接影响转变成间接影响，从而使对象之间的耦合度降低。在对于两个对象相互影响的情况下，这样的关系处理也许还能接受。但是多个对象互相影响，对象之间的关系就会变成的网状，这样的设计是高度耦合的。接下来就用一个翻转棋的游戏来将这个模式表现出来。

<!-- more -->

首先介绍出现的对象：首先我们需要一个棋盘，棋盘上的棋子，接下来是一个裁判和两名玩家。
棋盘我们这样来设计，其实棋盘也可以是抽象的，可以为多种棋做扩展，先简单来：
``` java
public class Board {

    //可定制棋盘的大小
    public Board(Integer length) {
        this.borderLength = length;
        this.table = new Chess[borderLength][borderLength];
    }

    //棋盘的格数
    private Integer borderLength;
    //棋盘上的所有位置
    private Chess[][] table;

    //省略get、set

    //初始化棋盘，翻转棋开局就会有4枚棋子
    public void initBoard() {
        for(int x = 0; x < borderLength; x++) {
            for(int y = 0; y < borderLength; y++) {
                table[x][y] = new Chess(x, y);
            }
        }

        //黑白棋的中间开始就是有4颗棋子
        table[3][3].setCode(Chess.ChessCode.WHITE);
        table[4][3].setCode(Chess.ChessCode.BLACK);
        table[3][4].setCode(Chess.ChessCode.BLACK);
        table[4][4].setCode(Chess.ChessCode.WHITE);
        //初始化的时候黑棋能下的位置
        table[2][3].setCode(Chess.ChessCode.ALLOW);
        table[3][2].setCode(Chess.ChessCode.ALLOW);
        table[4][5].setCode(Chess.ChessCode.ALLOW);
        table[5][4].setCode(Chess.ChessCode.ALLOW);

        printBoard();
    }

    public void printBoard() {
        for(int x = 0; x < borderLength; x++) {
            for(int y = 0; y < borderLength; y++) {
                System.out.print("[" + table[x][y].getCode().getValue() + "]");
            }
            System.out.println();
        }
    }

}
```
接下来定义上面出现的棋子：
``` java
public class Chess {

    private Integer xPoint;

    private Integer yPoint;
    //棋子类型
    private ChessCode code;

    public enum ChessCode {

        //无棋子
        NULL (0, " "),
        //黑棋
        BLACK(1, "●"),
        //白旗
        WHITE(2, "○"),
        //可走位置
        ALLOW(9, "·");

        private int code;
        
        private String value;

        //省略get set

        ChessCode(int code, String value) {
            this.code = code;
            this.value = value;
        }
    }

    public Chess(Integer x, Integer y) {
        this.code = ChessCode.NULL;
        this.xPoint = x;
        this.yPoint = y;
    }

    public Chess(Integer x, Integer y, ChessCode chessCode) {
        this.code = chessCode;
        this.xPoint = x;
        this.yPoint = y;
    }
}
```
接下来是抽象的裁判和抽象的玩家
``` java
public abstract class Abstract AbstractPlayer {

    //玩家手执上面类型棋子；
    protected Chess.ChessCode chessCode;

    public AbstractPlayer(Chess.ChessCode code) {
        this.chessCode = code;
    }

    //玩家下棋， 然后告诉裁判
    public moveChess (Chess chess, AbstractJudger judger) {
        chess.setCode(this.chessCode);
        //裁判做操作
        judger.judge(chess);
    }
}

```

``` java
public abstract class AbstractJudger {

    //当前轮到哪个玩家
    protected AbstractPlayer currentPlayer;

    //棋盘
    protected Board board;

    //裁判需要知道所有的玩家
    protected AbstractPlayer player1;

    protected AbstractPlayer player2;

    protected Integer totalChess;

    public AbstractJudger(Board board) {
        this.board = board;
        this.totalChess = board.getBorderLength() * board.getBorderLength();
    }

    public void ready(AbstractPlayer player1, AbstractPlayer player2) {
        this.player1 = player1;
        this.player2 = player2;
        this.currenPlayer = player1;
    }

    //切换玩家
    private void toggleCurrentPlayer() {
        this.currentPlayer = currentPlayer.equals(player1) ? player2 : player2;
    }

    private boolean checkBeforePut(Chess c){
        //判断是否轮到该用户
        if(c.getCode() != this.currentCode) {
            System.out.println("It is not your turn !");
            return true;
        }
        //是否下到棋盘外
        if(c.getXPoint() < 0 || c.getXPoint() > maxLength || c.getYPoint() < 0 || c.getYPoint() > maxLength) {
            System.out.println("you can't put chess here! out of range!");
            return true;
        }
        Chess[][] table = this.board.getTable();
        //判断当前位置能否落子
        if(table[c.getXPoint()][c.getYPoint()].getCode() != Chess.ChessCode.ALLOW) {
            System.out.println("you can't put chess here!");
            return true;
        }
        return false;
    }

    //放置棋子
    void putChess(Chess chess) {
        this.board.getTable()[chess.getXPoint()][chess.getYPoint()] = chess;
        this.toggleCurrentPlayer();
        this.totalChess --;
    }

    void judge(Chess c) {
        //初步判断
        if(checkBeforePut(c)) {
            return;
        }
        //判断反转和游戏是否结束、放置棋子、结算放置棋子
        if(!this.judgeChessMove(c)) {
            judgeGameOver();
            return;
        }
        //重新打印棋盘
        this.refreshBoard();
    }

    //判断棋子移动
    abstract boolean judgeChessMove(Chess chess);
    //判断游戏结束
    abstract void judgeGameOver();
}
```
只要实现了翻转棋的裁判和玩家就能开始了：
``` java
public class ReverseJudger extends AbstractJudger {

    
    public boolean judgeChessMove(Chess chess){
        //实现棋子的翻转和落子提示（翻转棋只能下在提示的位置）
        //当落子提示中没有可落位置，返回false
    }
    
    public void judgeGameOver() {
        //当棋盘里面没有可以落子的时候判断游戏结束
    }
}
```
``` java 
public class ReversePlayer extends AbstractPlayer {
    public ReversePlayer(Chess.ChessCode code) {
        super(code);
    }
}
```
&emsp;&emsp;这样就可以开始了，可以看到白棋和黑棋之间的相互影响是通过裁判来进行的，毕竟钥匙白棋直接自己动手翻动黑棋的棋子，而黑棋也这样做的话，这样的话游戏的体验就非常的差了，最后放上一张最后的对弈情况;

![step1](mediator_1.png)

![step1](mediator_2.png)

![step1](mediator_3.png)

ps: 游戏的最终效果可以通过博客菜单中的游戏链接进入查看（尝试对弈可以开启2个窗口访问）