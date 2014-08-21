##Django 文件上传##

当Django 处理文件上传时,文件是通过request.FILES访问的,这是一个字典,保存了通过表单上传的所有文件.

当且仅当表单使用了POST方法,并且设置了属性enctype="multipart/form-data"时,request.FILES才包含了文件数据,否则为空.
request.FILES的值都属于UploadedFile对象,它们是文件对象的简单包装,在文件的基本方法上增加了,`multiple_chuncks()` , `chunks()`等方法
文件小于2.5M时,数据会被直接载入内存,当大于2.5M,文件将保存在一个临时文件夹内,通过硬盘访问.
setting 文件里有三个参数与文件上传有关:

- 默认最大内存占用大小通过 `FILE_UPLOAD_MAX_MEMORY_SIZE`设置,默认2.5M
- 文件暂时存放的路径通过`FILE_UPLOAD_TEMP_DIR`设置
- `FILE_UPLOAD_PERMISSIONS` 规定了文件上传的权限,如果为空,则权限取决于服务器的系统权限.

##Upload Handlers##
用户上传文件时, Django会调用一个upload handler类进行处理.处理类在
`FILE_UPLOAD_HANDLERS`设置,默认为:
```
("django.core.files.uploadhandler.MemoryFileUploadHandler",
 "django.core.files.uploadhandler.TemporaryFileUploadHandler",)
```
这两个处理类分别提供了上传文件到内存,和上传文件到临时文件夹的功能.我们也可以自定义upload handler

###动态修改upload handler###
你可以通过修改request.upload_handlers 来添加自己的upload handler,但是只能在 访问 request.POST, 和request.FILES 之前修改.

###自定义 upload handler###
自定义的upload handle 必须是FileUploadHandler的子类.子类**必须**实现下列的方法:

- FileUploadHandler.receive_data_chunk(raw_data, start)
  从上传的文件中收到一块数据.`raw_data`是上传数据的比特流,`start`是`raw_data`
  在源文件中的位置. 这个函数的返回值会作为一下 upload handler的receive_data_chunk的参数, 众多upload handler像管道一样连接起来.如果返回None, 后面upload handler将不会得到这个数据块.如果你抛出StopUpload 或者SkipFILE异常,上传过程会终止,整个文件被丢弃.

- FileUploadHandler.file_complete(file_size)
 在文件上传完成以后被调用. 这个函数应该返回一个UploadFile对象,它将被存储在request.FILES里面.如果函数返回None,则说明UploadFile对象将由后续的upload handler 提供.

###可选方法###
还有一些可以自定义的属性和方法
- FileUploadHandler.chunk_size upload handler 收到的数据块的大小, 也就是FileUploadHandler.receive_data_chunk 函数收到的数据块的大小.
数据块大小应该是4的整数倍,而且不能超过2G,这样性能最优,当不同的upload handler
设置了多个块大小时,Django会以最小的值为准. 默认块大小为64kb.


















