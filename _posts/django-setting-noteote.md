##基础##
setting.py 是一个保存了模块级全局变量的文件(也是模块)
因为setting.py是一个python模块,所以它的内容要符合以下条件:
- 符合python语法
- 允许变量动态赋值
- 允许从其他setting文件引入变量

你要告诉Django你想使用哪一个setting.py, 所以需要设置环境变量
DJANGO_SETTINGS_MODULE.
DJANGO_SETTINGS_MODULE的值必须满足python的路径语法,比如 mysite.settings 

##默认设置##

Django setting 的默认值 由django/conf/global_settings.py规定
Django 编译setting的顺序如下:
- 从global_settings.py里面加载变量
- 再从DJANGO_SETTINGS_MODULE指定的配置文件中加载变量,如果有冲突则覆盖global_settings 中的设置.setting 文件不可以从global_settings 中加载变量,因为Django会自
动完成这个事情.

你还可以用 manage.py diffsettings 命令来查看当前设置与默认设置有什么不同.

##在代码中使用##

在Django app中，可以导入djang.conf.setting对象来访问setting,注意这不是一个模块而是一个对象.不可以从global_setting或者其他setting文件直接导入参数,因为django.conf.setting 对象为所有的参数提供了统一的访问接口,还将参数与配置文件路径解耦(不需要知道配置文件的路径).

##请不要在应用运行时修改setting !##

##创建自己的setting##

你当然可以创建自己的setting,要注意两个规范:
- 配置名全部大写
- 不要重写已有的配置
如果配置是一个序列,Django的规范是用元组而非列表,但并不强制

##在没有DJANGO_SETTINGS_MODULE的情况下使用setting##
有时候你想绕过DJANGO_SETTINGS_MODULE的设置, 对个别参数做另外的设置,可以调用
这个函数django.conf.settings.configure(default_settings, **settings)
configure函数的关键字参数都被作为参数注入Django,其他没有指定的参数将使用global_settings中的默认设置.
###自定义默认设置###

如果不想使用django.conf.global_settings,你可以传递一个提供了自定义默认参数的模块或者类给 configure函数的default_settings 参数.


##`DJANGO_SETTINGS_MODULE`和`configure()`必须使用其中的一个##
如果两个都不用,Django会抛出`ImportError`异常
如果两个都用, Django会抛出`RuntimeError`异常. 






























