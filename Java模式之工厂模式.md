Java模式之工厂模式：

1、简单工厂模式

通过一个唯一的大工厂来生产所有的产品。



抽象产品--->具体产品

factory实现类，方法getProduct(String name)，通过传入的name来获取对应的具体类生产。



2、工厂模式

抽象工厂---->具体工厂（每个具体产品对应一个具体工厂，生产对应的具体类）

抽象产品---->具体产品 （产品继承同一个接口，工厂类实现由具体子类实现，实现对应的产品工厂）



3、抽象工厂模式

原来的抽象工厂中返回的是一个产品的工厂，抽象工厂的接口中可以返回多个产品的抽象工厂！

http://www.hollischuang.com/archives/1401

https://juejin.im/entry/58f5e080b123db2fa2b3c4c6

http://www.hollischuang.com/archives/1408





使用场景和示例：colllection和数据库的连接（不同数据库的连接，改驱动的名字）