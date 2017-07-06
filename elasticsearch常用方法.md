## elasticsearch常用方法

* 使用**HEAD**检查是否存在，返回响应头信息
* 使用**DELETE**删除文档
* 使用**PUT**添加文档（在这个URL下存储文档）
* 使用**POST**添加文档（在类下面，也就是在_type下面，主要用于自动生成_id）
* 使用**GET**检索文档（利用`-i`参数可以显示响应头）


### 文档元数据

节点  	| 		说明
-----  |-------
_index |	索引，文档存储的地方
_type	| 文档的类型，代表对象的类
_id    | 文档唯一的标识

```
_index/_type/_type
```


###增加

新增加一个文件的索引可以使用两种方法，一种是`PUT`方法，该方法却要确定的`index,type,id`；另外一个是`POST`方法，该方法只需要确定的`index,type`，系统会为文件自动分配一个不重复的自增长的uuid。



###删除

删除一个已有的检索，使用`DELETE`方法，`DELETE /index/type/id`，如果正常删除，则返回一个`200 OK`状态码，如果未正常删除，则返回`404 Not Found`状态码




###检索

* 查看所有的索引     
`curl 'http://localhost:9200/_cat/indices?v'`    

* 查看所有的节点     
`curl 'http://localhost:9200/_cat/nodes?v'`

* id检索    
`curl XGET 'http://localhost:9200/wbesite/blog/1?pretty'`

* 字段检索    
`curl XGET 'http://localhost:9200/wbesite/blog/123?_source=title,test?pretty'`




###修改
文档在elasticsearch是不可变的，我们只能重建索引的方法去修改一个文件。
    
如何确定创建的索引文档是新建还是覆盖一个已经存在的，最好的方法是使用`POST /index/type`在类下自动生成一个唯一的id来生成一个索引。    

如果需要自定义`id`，且不能确定`index、type、id`三者唯一的情况下，可以使用`PUT /index/type/id?op_type=create`或者`PUT /index/type/id/_create`确定，如果是新建的，则会返回`201 Created`状态码，如果存在，则会返回`409 Confilt`状态码，文件已经存在。

修改文档的一般流程：

* 查找索引
* 删除索引
* 重建索引

elacsticsearch提供了一个`updata`方法来进行文档的局部更新。`POST '/index/type/id/_update'  -d  '{"doc":{具体内容}}'`进行更新。


###批量检索

elasticsearch系统提供了一个批量检索的方法`mget`,它所需要的参数为一个`docs`数组，该数组中包括具体的`_index`,`_type`,`id`;如果需要在特定的`index`下进行查找，可以在url中指定`/index/_mget`，同理特定的`type`

###批量操作

elasticsearch提供了一个批量操作`bulk`来实现批量的`create`,`index`,`update`或者`delete`。具体的实现格式如下：

```
POST /_bulk
{
    {action:{metadata}}\n
    {request:{data}}\n
    {action:{metadata}}\n
    {request:{data}}\n
}
```

需要注意几点：

* 每一行操作都是以回车的转义字符为结尾,包括最后一行
* `delete`操作不需要`request`行
* `metedata`指的是`_index`,`_type`,`id`等项，如果需要在特定的`index`中进行批量操作，可以在url中指定，同理`type`
