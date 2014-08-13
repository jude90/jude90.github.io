Django lazy
Django的惰性计算,使函数返回的不是的计算结果,而是一个__proxy__对象.
当这个对象的某个属性或者方法被访问时,函数才会被计算. 函数计算需要的上下文和参数
由__proxy__对象来维护.
比如 ```func_lazy= lazy(func, resultclasses)```, resultclasses 是func函数期望的返回值类型, func_lazy 的返回值就是一个惰性对象,它的属性和方法是由resultclasses对应的属性和方法加上func函数的参数得到了的.
foo = func_lazy(*args,**kw),

func 的计算结果只有在 foo 某个属性或者方法被访问时才会计算出来.
