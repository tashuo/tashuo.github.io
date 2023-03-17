---
title: Symfony EasyAdminBundle初体验
date: 2016-12-22 12:01:13
categories:
- symfony
- 笔记
tags:
- symfony
---

## 安装[symfony](http://symfony.com/doc/current/setup.html)

### Linux/OSX
*   `$ sudo curl -LsS https://symfony.com/installer -o /usr/local/bin/symfony`
*   `$ sudo chmod a+x /usr/local/bin/symfony`

### OSX
*   `$ brew install homebrew/php/symfony-installer`

## 新建symfony项目
`$ symfony new my_project_name [version]`  

symfony 3.0 开始需PHP版本不低于5.5.9, 2.x版本支持PHP 5.3.x版本, 需要根据自己的PHP版本来确定symfony项目的版本, version参数不加默认是master分支, 具体可查 [symfony的composer.json文件](https://github.com/symfony/symfony/blob/master/composer.json) 

也可用composer方式新建项目: `composer create-project symfony/framework-standard-edition my_project_name "[version]"`  

测试项目是否初始化成功

*   进入到 `my_project_name` 文件夹  
*   启动php内置web服务 `php bin/console server:run`
*  访问 `http://localhost:8000` 即可看到symfony欢迎页面

**注: symfony 3.0之前的版本console文件路径是 app/console, 之后的是 bin/console, 下文默认使用3.0后的版本, 如果你的环境是2.x版本则需改为 app/console**

## 初始化数据库
**注: 若数据库已存在则不需要这个过程, 只需将配置文件中 database_name 字段修改为所用数据库名称**
### 修改配置文件
编辑 app/config/parameters.yml, 修改 `database_` 相关字段的配置
<!-- more -->
### 生成数据库
`$ php bin/console doctrine:database:create --if-not-exists`    
此处使用 [doctrine orm](https://github.com/doctrine/doctrine2) 框架来实现

## 使用 [FOSUserBundle](https://github.com/FriendsOfSymfony/FOSUserBundle)
FOSUserBundle 是symfony框架用于用户身份验证的模块, 类似于 laravel的auth模块
### 安装
`$ composer require friendsofsymfony/user-bundle "~2.0@dev"`    
此处需根据自己php版本来选择相应的版本安装

### 注册模块
AppKernel.php 文件中增加FOSUserBundle模块  

```php
<?php
// app/AppKernel.php

public function registerBundles()
{
    $bundles = array(
        // ...
        new FOS\UserBundle\FOSUserBundle(),
        // ...
    );
}
```

### 调整User类
修改 src/AppBundle/Entity/User.php 文件，引入FOSUserBundle User类       

```php
<?php
// src/AppBundle/Entity/User.php

namespace AppBundle\Entity;

use FOS\UserBundle\Model\User as BaseUser;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 * @ORM\Table(name="fos_user")
 */
class User extends BaseUser
{
    /**
     * @ORM\Id
     * @ORM\Column(type="integer")
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    protected $id;

    public function __construct()
    {
        parent::__construct();
        // your own logic
    }
}  
```

### 调整 security.yml
security.yml文件是项目基础身份认证的配置文件, 需要在此引入FOSUserBundle       

``` yaml
# app/config/security.yml
security:
    encoders:
        FOS\UserBundle\Model\UserInterface: bcrypt

    role_hierarchy:
        ROLE_ADMIN:       ROLE_USER
        ROLE_SUPER_ADMIN: ROLE_ADMIN

    providers:
        fos_userbundle:
            id: fos_user.user_provider.username

    firewalls:
        main:
            pattern: ^/
            form_login:
                provider: fos_userbundle
                csrf_token_generator: security.csrf.token_manager
                # if you are using Symfony < 2.8, use the following config instead:
                # csrf_provider: form.csrf_provider

            logout:       true
            anonymous:    true

    access_control:
        - { path: ^/login$, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/register, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/resetting, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/admin/, role: ROLE_ADMIN }
        
```

### 调整 config.yml
上一步修改的 security.yml 文件中 `providers` 选项下 `id: fos_user.user_provider.username`, 现在是要做 `fos_user` 这个id的配置 

``` yaml
# app/config/config.yml
fos_user:
    db_driver: orm # other valid values are 'mongodb' and 'couchdb', 同之前配置
    firewall_name: main     // 同上一步中firewalls配置
    user_class: AppBundle\Entity\User   // 同前面调整的User类
```

### 调整 routing.yml
routing.yml 文件负责项目路由规则，这一步引入 FOSUserBundle 路由规则 

``` yaml
# app/config/routing.yml
fos_user:
    resource: "@FOSUserBundle/Resources/config/routing/all.xml"
```

### 更新数据库
`$ php bin/console doctrine:schema:update --force`  

这一步会在数据库中创建`fos_user`表, 表名在前面 User 类中配置 `@ORM\Table(name="fos_user")`, 可修改为你所需要的名称

### 验证
访问 `http://localhost:8000/login` 页面, 会出现登录框, 不过好丑

### 增加测试用户
`$ php bin/console fos:user:create admin admin@local 123456`    
`$ php bin/console fos:user:promote admin ROLE_ADMIN`

这步会向 fos_user 表中插入一条数据, 用户名: admin, 密码: 123456, 并且设置该用户为admin角色, 供之后登录测试用

## 使用 [EasyAdminBundle](https://github.com/javiereguiluz/EasyAdminBundle)
EasyAdminBundle包是symfony框架下一个简单易用可扩展的后台
### 安装
`$ composer require javiereguiluz/easyadmin-bundle`

### 引入
1.   引入 app/AppKernel.php
    ```php
    <?php
    // app/AppKernel.php
    
    public function registerBundles()
    {
        $bundles = array(
            // ...
            new FOS\UserBundle\FOSUserBundle(),
            new JavierEguiluz\Bundle\EasyAdminBundle\EasyAdminBundle(),
            // ...
        );
    }
    ```
2.   引入 app/config/routing.yml
    ``` yaml
    # app/config/routing.yml
    easy_admin_bundle:
        resource: "@EasyAdminBundle/Controller/"
        type:     annotation
        prefix:   /admin        // 设置admin为后台目录
    ```
3.   引入资源文件  
`$ php bin/console assets:install --symlink`

4.   配置
    编辑 app/config/config.yml, 配置EasyAdminBundle, 加入前面配置的 AppBundle\Entity\User
    ``` yaml
    # app/config/config.yml
    easy_admin:
        entities:
            - AppBundle\Entity\User
    ```

    访问 `http://localhost:8000/login`, 输入之前插入的admin用户名密码, 登录成功会跳转至 `http://localhost:8000/admin` 页面,  至此就完成了 EasyAdminBundle 的安装, 左侧栏只有上一步 `config.yaml` 中设置的 User 菜单

    如果你已经建立好项目所需的表, 需要使用 symfony 根据已存在的表生成 Entity

## 引入已存在的数据表
1.   table to xml
    `$ php bin/console doctrine:mapping:import --force AppBundle xml`   

    这一步使用Doctrine生成数据库中独立的含有表结构信息等特殊结构的xml文件, 目录在 src/AppBundle/Resources/config/doctrine/ , 这个路径需根据自己项目路径确定

2.   xml to Entity
    `$ php bin/console doctrine:generate:entities AcmeBlogBundle`   

    根据上一步生成的xml文件生成相应的 Entity 文件

    **注: 这一步会生成所有表对应的Entity文件, 之前已存在 Entity\User 文件, 默认不覆盖, 会生成一个 Entity\User.php~ 文件, 可直接删掉此文件, 如果某种原因覆盖掉了, 需根据前面步骤重新在 Entity\User 文件中引入FOSUserBundle**

    可能还会报另一个错 `[Doctrine\ORM\Mapping\MappingException] Duplicate definition of column 'username' on entity 'AppBundle\Entity\User' in a field or discriminator column mapping.`, 可能是两个原因导致:     
    *   如果`AppBundle\Entity\User.php`文件是根据xml文件生成, 则文件中会定义有`username`, `email`等字段, 而`FOS\UserBundle\Model\User`类中已定义有这些字段, 故报错
    *   如果没有使用xml生成的`AppBundle\Entity\User.php`, 则是`AppBundle/Resources/config/doctrine/User.orm.xml`文件中重复定义所导致     

    解决方法: 删除`User.orm.xml`文件中跟`FOS\UserBundle\Model\User`有冲突的字段即可

3.   配置菜单
    修改 app/config/config.yml, 引入各个 Entity   

    ``` yaml
    # app/config/config.yml
    easy_admin:
        entities:
            - AppBundle\Entity\User
            - AppBundle\Entity\Post
            - AppBundle\Entity\Comment
            ...
    ```


<br/>
刷新admin页面即可看到左侧菜单已增加, bingo!

<br/><br/><br/><br/>
参考资料: 
>- Symfony2 Admin panel in 30 seconds https://level7systems.co.uk/symfony2-admin-panel-in-30-seconds/
>- 使用FOSUserBundle https://symfony.com/doc/master/bundles/FOSUserBundle/index.html
>- 安装Symfony http://symfony.com/doc/current/setup.html
>- 使用EasyAdminBundle https://github.com/javiereguiluz/EasyAdminBundle



