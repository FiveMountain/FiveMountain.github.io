---
title: Spring Data 入门
date: 2021-03-10 18:07:34
categories:
- Learning Notes
- Spring
---
## 安装MongoDB(CentOS7)

### 安装Docker
在root账户下执行以下命令

    yum -y update
    yum -y install epel-release
    yum -y install docker-io

### 安装MongoDB


    # 启动Docker
    systemctl start docker
    # 检查docker版本号
    docker version

    # 下载镜像(27017是MongoDB服务的端口号)
    docker pull mongo:latest
    # 查看MongoDB的镜像
    docker images

    # 启动MongoDB
    docker run -itd --name mongo -p 27017:27017 mongo --auth
    # 检查MongoDB是否启动成功(输出mongo及其端口号27017信息，就表示成功了)
    docker ps


### 登录数据库


    docker exec -it mongo mongo admin


### 创建admin账户

创建一个用户admin(管理员账户)，密码123456(仅演示，可自行设置复杂密码)

    db.createUser({user:'admin', pwd:'123456', roles:[{role:'root', db:'admin'}]});


> 这一步执行成功后，以后重新登录云服务器，就不需要创建管理员账户了

认证管理员账户

    db.auth('admin', '123456')

系统返回结果`1`，则表示admin账户创建并验证成功

## 创建数据库实例

创建一个`practice`数据库，然后再创建一个可读写操作的用户`uuu`，密码自行设置

> MongoDB是数据库软件，`practice`是具体的数据库实例。一个MongoDB数据库软件中可以包含多个数据库实例

**注意**：数据库对应的用户，是用来读写本数据库的。不同于上一步创建的admin管理员账户，admin不能用来读写数据库

数据库名称(`practice`)，自己设置的用户名和密码很重要，请记牢

1. 切换数据库

`use`命令的作用是创建并切换到指定的数据库：

    use practice

看到系统输出`switched to db practice`就表示成功了

2. 创建读写用户

`db.createUser`用于创建可读写操作的用户

    db.createUser({user:'uuu', pwd:'wuyue98983.+uuu', roles:[{role:'root', db:'admin'}, {role:'dbAdmin', db:'practice'}]});

> 此处`practice`为上方创建的数据库名称


> 创建读写用户前必须先认证admin账户
>> 再次登录云服务器后，创建读写用户前仍需要再次认证admin账户(db.auth('admin', 'password'))


3. 认证读写用户

创建读写用户后执行一次db.auth()，验证设置的用户和密码是否正确

    db.auth('user', 'password')

再次登录数据库后，只需执行切换数据库(user db)和认证用户(db.auth())命令即可，无需再次创建用户

4. 退出登录

在MongoDB登录状态下执行`exit`可以退出数据库，再次执行可从云服务器退出登录

5. 结尾提醒

安装都是一次性的，但是要记住云主机公网IP地址、MongoDB服务的端口号、以及数据库的用户名和密码

## Spring Data MongoDB 配置

### pom.xml依赖

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>

### 配置

`src/main/resources/application.properties`中增加配置项

    # 购买的云服务器的公网IP
    spring.data.mongodb.host=xxx.xxx.xxx.xxx
    # MongoDB服务的端口号
    spring.data.mongodb.port=27017
    # 创建的数据库及用户名和密码
    spring.data.mongodb.database=practice
    spring.data.mongodb.username=user
    spring.data.mongodb.password=password


> 其中，用户名和密码是读写用户的


### 云服务器安全设置

检查云服务器是否开放27017端口

## Spring Data CRUD

对数据库的操作一定要放在`@Service`类中，而不是放在`@Controller`类中；且`@Controller`类可以调用`@Service`类的方法，反之则不行。这是SpringMVC的经典架构设计理念

- `@Service`类主要用于不易变的核心业务逻辑
- `@Controller`类与前端页面紧密配合，调用`@Service`服务读写数据，从而响应前端请求


### 新增数据

新增数据就是向数据库中插入一条数据。在Java世界中万物皆对象，所以，所谓数据就是实例对象

    import org.springframework.data.mongodb.core.MongoTemplate;
    
    @Autowired
    private MongoTemplate mongoTemplate;

    @Override
    public Song add(Song song) {
        // 作为服务，要对入参进行判断，不能假设被调用时，传入的一定是真正的对象
        if (song == null) {
        LOG.error("input song data is null.");
        return null;
        }

        return mongoTemplate.insert(song);
    }


### 查询数据


    @Override
    public Song get(String songId) {
        // 输入的主键 id 必须有文本，不能为空或全空格
        if (!StringUtils.hasText(songId)) {
            LOG.error("input songId is blank.");
            return null;
        }

        Song song = mongoTemplate.findById(songId, Song.class);
        return song;
    }

`findById()`方法第1个参数就是主键id，第2个参数是具体的类，写法是`类名.class`

### 修改数据

修改的操作包括两个部分：
1. 修改哪条数据？
2. 哪个字段修改成什么值？


    @Override
    public boolean modify(Song song) {
        // 作为服务，要对入参进行判断，不能假设被调用时，入参一定正确
        if (song == null || !StringUtils.hasText(song.getId())) {
            LOG.error("input song data is not correct.");
            return false;
        }

        // 主键不能修改
        Query query = new Query(Criteria.where("id").is(song.getId()));

        Update updateData = new Update();
        // 值为 null 表示不修改。值为长度为 0 的字符串 "" 表示清空此字段
        if (song.getName() != null) {
            updateData.set("name", song.getName());
        }

        if (song.getLyrics() != null) {
            updateData.set("lyrics", song.getLyrics());
        }

        if (song.getSubjectId() != null) {
            updateData.set("subjectId", song.getSubjectId());
        }

        UpdateResult result = mongoTemplate.updateFirst(query, updateData, Song.class);
        return result != null && result.getModifiedCount() > 0;
    }


先试用条件对象`Criteria`构建条件对象`Query`实例
然后再调用修改对象`Update`的方法`.set()`设置需要修改的字段
最后调用`mongoTemplate.updateFirst(query, updateData, Song.class)`方法完成修改，第三个参数是具体的类

### 删除数据

调用`mongoTemplate.remove()`方法即可删除数据，参数是对象，表示需要删除哪些数据

    @Override
    public boolean delete(String songId) {
        // 输入的主键 id 必须有文本，不能为空或全空格
        if (!StringUtils.hasText(songId)) {
            LOG.error("input songId is blank.");
            return false;
        }

        Song song = new Song();
        song.setId(songId);

        DeleteResult result = mongoTemplate.remove(song);
        return result != null && result.getDeletedCount() > 0;
    }

创建一个对象并设置好属性值，作为删除的条件，符合条件的数据都将被删除。可以设置更多的属性值来提高精确性，但通过主键来删除数据，是保证不误删的一个较好的方法

## Spring Data Query

条件查询操作的核心方法

    List<Song> songs = mongoTemplate.find(query, Song.class);

第一个参数是查询对象`Query`实例，第二个参数表示查询什么样的对象，写法是`类名.class`
查询操作的复杂性在于条件，需要用构建好的`Criteria`条件对象的实例，构建`Query`实例

    Query query = new Query(criteria);

构建`Criteria`条件对象，一般有两种情况：
- 单一条件，用：
`Criteria criteria = Criteria.where("条件字段名").is("条件值")`即可返回一个条件对象的实例
- 组合条件，根据or、and的关系进行组合，多个子条件对象组合成一个总条件对象：
    - or
    
        Criteria criteria = new Criteria();
        criteria.orOperator(criteria1, criteria2);

    - and

        Criteria criteria = new Criteria();
        criteria.andOperator(criteria1, criteria2);
    
    - `orOperator()`和`andOperator()`的参数，都可以输入多个子条件，也可以输入子条件数组

例如，根据歌曲的专辑查询，并限定最多返回10条数据

    import org.springframework.data.mongodb.core.query.Query;
    import org.springframework.data.mongodb.core.query.Criteria;

    public List<Song> list(Song songParam) {
        // 总条件
        Criteria criteria = new Criteria();
        // 可能有多个子条件
        List<Criteria> subCris = new ArrayList<>();
        if (StringUtils.hasText(songParam.getName())) {
            subCris.add(Criteria.where("name").is(songParam.getName()));
        }

        if(StringUtils.hasText(songParam.getLyrics())) {
            subCris.add(Criteria.where("lyrics").is(songParam.getLyrics()));
        }

        if(StringUtils.hasText(songParam.getSubjectId())) {
            subCris.add(Criteria.where("subjectId").is(songParam.getSubjectId()));
        }

        // 必须至少有一个查询条件
        if (subCris.isEmpty()) {
            LOG.error("input song query is not correct.");
            return null;
        }

        // 三个子条件以and关键词连接成总条件对象
        criteria.andOperator(subCris.toArray(new Criteria[]{}));

        // 条件对象构建查询对象
        Query query = new Query(criteria);
        query.limit(10);
        List<Song> songs = mongoTemplate.find(query, Song.class);

        return songs;
    }

> 这里使用Song对象来表示查询条件。作为服务接口，使用对象做参数，具备比较好的扩展性和兼容性。例如，如果增加一个查询条件，就不需要增加方法参数，只需要为参数对象增加属性即可，否则所有的调用查询接口方法的代码都需要做修改，影响面可能很大，扩展性和兼容性都不好。


## Spring Data 分页

分页是查询中最常用的功能，同时也为了防止一次查询的数据量太大而影响性能

查询支持分页也比较简单，只需要调用`PageRequest.of()`方法构建一个分页对象，然后注入到查询对象即可

`PageRequest.of()`方法第一个参数是页码，从**0**开始计数；第二个参数是每页的数量

    import org.springframework.data.domain.PageRequest;
    import org.springframework.data.domain.Pageable;

    Pageable pageable = PageRequest.of(0, 20);
    query.with(pageable);

对于分页来说，出了要查询结果以外，还需要查询总数，才能进一步计算出总共多少页，实现完整的分页功能。所以，还需要两个步骤
- 调用`count(query, XXX.class)`方法查询总数。第一个参数是查询条件，第二个参数表示查询什么样的对象
- 根据结果、分页条件、总数三个数据，构建分页器对象


    import org.springframework.data.domain.Page;
    import org.springframework.data.repository.support.PageableExecutionUtils;

    // 总数
    long count = mongoTemplate.count(query, Song.class);
    // 构建分页器
    Page<Song> pageResult = PageableExecutionUtils.getPage(songs, pageable, new LongSupplier(){
        @Override
        public long getAsLong() {
            return count;
        }
    });

`PageableExecutionUtils.getPage()`方法第一个参数是查询结果；第二个参数是分页条件对象；第三个参数实现一个`LongSupplier`借口的匿名类，在匿名类的`getAsLong()`方法中返回结果总数。方法返回值是一个`Page`分页器对象

control中得到分页结果对象后，可以调用`Page`的各个方法取得前端分页器的各种数据值

