## List中add和set的用法

#### 1.作用

add()是添加元素的作用，set()是更改元素的作用

#### 2.用法

void add（int index，e element）：将指定的元素（element）插入此列表中的指定位置（index）。将当前处于该位置的元素（如果有）和所有后续元素移动到右侧（将其索引添加到1），没有返回值。

set（int index，e element）：用指定的元素替换列表中指定位置的元素。返回值是通用的（由e决定）。

#### 3.不同点

当 index位置上没有元素时，set方法会报错，add不会

#### 4.注意

ArrayList集合里存的是一个对象的引用**。**当我们改变obj时，因为ArrayList.add的是 obj的引用，之前的元素都指向了同一个对象 obj，所以在改变obj时，之前添加的也会随之改变。