$ npm install nodal -g
$ nodal new [project-name]
$ cd [project-name]

```
nodal g:model Tweet user_id:int body:string #定义model结构
nodal db:create #创建数据库
nodal db:prepare # 生成schema_migrations
nodal db:migrate # 根据model生成表
nodal db:rollback # 回滚
nodal g:controller v1 --for tweet #生成controller
nodal s #启动服务
visit: 
    get 
        http://127.0.0.1:3000/v1/tweets
    post 
        Body(x-www-form-urlencoded): body -> Hey
        http://127.0.0.1:3000/v1/tweets
    put
        Body(x-www-form-urlencoded): body -> Hello.No1
        http://127.0.0.1:3000/v1/tweets/1
    delete
        http://127.0.0.1:3000/v1/tweets
    other query:
        http://127.0.0.1:3000/v1/tweets?body__not_null
        http://127.0.0.1:3000/v1/tweets?body__startswith=a
        http://127.0.0.1:3000/v1/tweets?body__endswith=b
        http://127.0.0.1:3000/v1/tweets?body__iendswith=b
        http://127.0.0.1:3000/v1/tweets?id__gte=3&__offset=2&__count=2
        
    

```

```
nodal db:create
nodal db:prepare
nodal g:model --user
nodal g:controller v1 --for:user
nodal db:migrate
```

```
$ nodal poly:login
$ nodal poly:new [project-name]
//Error: Command "poly:new" does not exist.
$ nodal poly:create [project-name]
$ 
```
