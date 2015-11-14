##Django 中间件##

中间件就是负责加工Django受到的请求和响应的模块,它是一个轻量的,插件式的底层模块,用于改变Django的输入输出.

每个中间件都有一个特定的功能,比如Django的中间件`AuthenticationMiddleware`,通过session 在一个请求和用户账号联系起来.

###激活一个中间件###

把你自己的中间件添加到Django setting的 MIDDLEWARE_CLASSES 元组中,就可以激
活它.在MIDDLEWARE_CLASSES 中,中间件用字符串来表示,而且必须是中间件类的完整路径.
中间件在MIDDLEWARE_CLASSES的排列顺序很重要,因为中间件之间有依赖关系.

###钩子函数和应用顺序####
在处理请求阶段,调用View之前,Django会按照 MIDDLEWARE_CLASSES中间的顺序自顶
向下依次调用中间件,用到两个钩子函数
- `process_request()`
- `process_view()`

在处理响应阶段,也就是View返回了一个响应之后,中间件会以与刚才相反的顺序被调
用,从下往上,调用三个钩子函数.
- `process_exception()`(当且仅当抛出异常时)
- `proces_template_response()` (当且仅当用模板响应)
- `process_response()`

看起来就像一个洋葱:每个中间件都是View的一层包装.














