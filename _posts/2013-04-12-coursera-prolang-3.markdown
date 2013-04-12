---
layout: post
title: "Coursera Prolang总结3(Haskell的type class)"
date: 2013-04-10 20:48
comments: true
categories: Coursera Haskell
---

Dan说如果时间允许, Haskell也将被讨论. 当然, 时间总是不允许的.

于是我看完SML之后就很不知趣地去啃Haskell. 然后我现在就是一个接触Haskell累计不超过一周的水货, 但是这并不妨碍我发表一些无关痛痒/*咦?*/的并且很有可能失偏颇的断章取义的评论. Haskell和SML有不少相似的地方, 但是我今天想讨论的是其不同的地方. 最大的不同, 在我看来, 是Haskell允许用户定义type class.

有两个基本的前置科技要点 : 模板(C++)/泛型(Java或SML)和纯虚类(C++)/接口(Java).

我某天就遇到了Haskell的type class:

``` hs
class Eq a where
    (==) :: a -> a -> Bool
    (/=) :: a -> a -> Bool
    x == y = not (x /= y)
    x /= y = not (x == y)
```

上一片代码描述了*一类类型*, 此类类型的值可以和同类比较是否相等. 代码逻辑由该类型自己实现(如同实现一个接口). 如果我们实现一个列表类型, 其相等操作是递归地比较每个相同位置的元素, 只有每个相对元素都相等时列表才相等:

``` hs
data MyList a = Cons a (MyList a) | Empty

instance (Eq k) => Eq (MyList k) where
    Cons a as == Cons b bs = (a == b) && (as == bs)
    Empty == Empty = True
    _ == _ = False
```

data定义了列表类型(风格和SML一致); 而下面的instance表示MyList对于每个可比较的类型参数k, 可以定义自己的比较函数, 从而让自身成为可比较类型. 下面的where里是一个简单的pattern matching, 和SML的case一致.

如果没法获得一个确切的直观的印象, 请看Java版:

``` java
interface Eq<T> {
    public boolean equal(T b);
}

abstract class List<T extends Eq<T>> implements Eq<List<T>> {
    abstract public boolean equal(List<T> a);
    abstract protected boolean equalEmpty();
    abstract protected boolean equalCons(Cons<T> a);
}

class Cons<T extends Eq<T>> extends List<T> {
    private T data;
    private List<T> next;

    public Cons(T data, List<T> next) {
        this.data = data;
        this.next = next;
    }

    @Override
    public boolean equal(List<T> b) {
        return b.equalCons(this);
    }

    @Override
    protected boolean equalCons(Cons<T> cons) {
        return data.equal(cons.data) && next.equal(cons.next);
    }

    @Override
    protected boolean equalEmpty() {
        return false;
    }

}

class Empty<T extends Eq<T>> extends List<T> {
    @Override
    public boolean equal(List<T> b) {
        return b.equalEmpty();
    }

    @Override
    protected boolean equalCons(Cons<T> data) {
        return false;
    }

    @Override
    protected boolean equalEmpty() {
        return true;
    }
}
```

不幸的是这个例子包含了不止一个知识 : **double dispatch**和**bounded polymorphism**. 但是反正这些东西都挺有意思的, 不妨一并学了吧:

**double dispatch** : 这个例子中, List有两种实际情况 : `Cons`和`Empty`. 于是`equal`事实上是一个多态函数 : `Cons`和`Cons`比, `Cons`和`Empty`比, `Empty`和`Cons`比, `Empty`和`Empty`比. 考虑`someEmpty.equal(b)`的状况, 我并不知道`a`是`Empty`还是`Cons`, 虽然我知道自己是`Empty`; 我们必须写4个辅助函数来实现这四位的逻辑, 然后在`equal`中只做`dispatch`. 这时候我们就要做第二次dispatch(跳`equal`虚函数是第一次), 执行`b.equalEmpty()`来告诉b, 有个`Empty`要跟你比. 和Dan的观点一致, 我觉得这丑暴了. 一个函数最好*do one thing and do it well*, 这本来连成一片的逻辑, 最好还是用一个大的`switch (type)`来解决. 这肯定会被说"不够OOP", 不过反正我不稀罕...

**bounded polymorphism** : 很简单 ------ 泛型参数不再是接受类型全集. 过去Java里有`class ClassName<T>`, 现在有这样的写法 : `class ClassName<T extends ClassName2>`. 这就提醒编译器`T`是可以被限制的. 这样T也就可以使用`ClassName2`所拥有的功能了. C++里对应的概念就叫concepts, 不过还没进入标准>.<

Java可以通过泛型接口来实现type class. 但是这不完全一样 :

1) 为了达成和double dispatch一样的功能, Haskell的类型系统有BNF风格的机制, 直接秒杀继承机制;

2) 为了达成"类型A is-a 类型B", Haskell没有像Java那样通过继承(实现接口), 而是通过泛型参数的传入(List is-a Eq). 区别在哪呢? 直观地讲, Java里总有个甩不掉的this, 而Haskell里没有. 为什么要有this呢? 没有this就是一个全局函数(Java里没有, C++里有); 不知道这个函数属于哪个类, 就算这个函数实现了某个接口里的方法, 也不知道哪个类能使用这个接口, 即没法提供"A is a B"这样的信息, 由于A的缺失; 那反正只要想个办法把A搞出来就行了. Mixin是一个不错的尝试, 它允许让用户指定一个函数属于多个类来产生类型约束, 但是这还是没有脱离让用户指定"这个函数属于某个类"来在实现接口的时候提供有用的类型约束. Haskell的提供约束的方式 ------ 现在很显然了 ------ 通过类型推导自动确定! 因为我们很容易判断一个函数原型是不是另一个函数原型的具体化. 比如lessThan(int, int)就是lessThan(T, T)的具体化(T是泛型). 这就为"类型int就是lessThan所属的那个type class的一个成员"提供了支持.

所以总结一下, type class 是这样的 :
是泛型接口, 但是不是靠继承(即把this这个参数揪出来瞎搞)来产生接口约束, 而是靠泛型参数来产生.

现在, 你看到了继承机制的生硬的设定了么?

如果我们把a.b(c)看成b(a, c), 那么类型系统会*仅*通过第一个参数的类型来产生各种约束. 为什么C++/Java要通过这种近乎hack的手段来突破类型系统呢? 啊哈, 原来是C++/Java不会做类型推导! 确实, C++的模板参数在很多情况下, 明明能推导出来, 但是就是要你手填T\_T.

在有强大的类型推导的语言里, 我们完全可以通过更灵活的策略来产生约束, 正如Haskell做的那样.
