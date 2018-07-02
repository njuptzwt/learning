java中自动装箱，拆箱遇到的神坑......



项目中使用到了，update某个对象（itemUpdateVO)，某个字段设置为Integer类型，可以传入为空。。。。。传入空的时候，一直报空指针异常。。。。。最终的原因是拆箱的操作，导致了空对象调用了intValue方法（）；

integer price;

然后get和set方法为：

public <u>***int***</u> getPrice(){

return price;

}

public void setPrice(Integer price){

this.price=price;

}





1、

什么叫装箱：

jdk自动调用：Integer.ValueOf(int value);

jdk自动调用对象的转换方法，将一般的数据类型直接转换，一般不会有什么问题



什么叫拆箱：

jdk自动调用对象的转换方法Object.intValue(),将对象转换为一般的数据类型，这个地方有坑，如果对象是一个null，调用方法会出现空指针的异常。。第一个坑。。



2、等号==在比价对象和普通数据的时候的对比



==是判断两个对象的地址是否相等，或者两个基本的数据类型是否相等

如果一个是对象和一个基本的数据类型的话就是需要装箱和拆箱来解决问题



Object==Primitive,对象和原始数据类型比较，对象进行拆箱。进行比较

Integer a=3;

int b=3;那么  a==b值为true

陷阱2：

int类型的自动装箱



integer a=100;

integer b=100;

integer c=300;

Integer d=400;

那么：

a==b,值为true

c==d,值为false

<u>**为什么：源码里面显示的是，当调用自动装箱的时候int类型：****</u>

<u>***-128~127范围内***</u>

这个范围内直接返回的是同一个Integer对象的引用，所以不管建立几个都是一样的！