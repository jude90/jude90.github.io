python 属性的查找顺序是 __getattribute__ ， __dict__ , __getattr__ .
一个接一个调用。在 __getattribute__ 中 调用对象的某个属性， 可能会引起递归。
