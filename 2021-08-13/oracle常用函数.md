## oracle常用函数

##### 1.decode

decode(X，A，B，C，D，E）

这个函数运行的结果是，当X = A，函数返回B；当X != A 且 X = C，函数返回D；当X != A 且 X != C，函数返回E。 其中，X、A、B、C、D、E都可以是表达式，这个函数使得某些sql语句简单了许多。

##### 2.instr

instr( string1, string2, start_position,nth_appearance )

●string1：源字符串，要在此字符串中查找。

●string2：要在string1中查找的字符串 。

●start_position：代表string1 的哪个位置开始查找。此参数可选，如果省略默认为1. 字符串索引从1开始。如果此参数为正，从左到右开始检索，如果此参数为负，从右到左检索，返回要查找的字符串在源字符串中的开始索引。

●nth_appearance：代表要查找第几次出现的string2. 此参数可选，如果省略，默认为 1.如果为负数系统会报错。

