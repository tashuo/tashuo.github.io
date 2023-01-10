---
title: Hyperf db连接池
date: 2023-01-09 12:47:24
categories:
- php
- hyperf
tags:
- 连接池
---

### [连接池](https://hyperf.wiki/3.0/#/zh-cn/db/quick-start)

> [hyperf/database](https://github.com/hyperf/database) 组件是基于 [illuminate/database](https://github.com/illuminate/database) 衍生出来的组件，我们对它进行了一些改造，从设计上是允许用于其它 PHP-FPM 框架或基于 Swoole 的框架中的，而在 Hyperf 里就需要提一下 [hyperf/db-connection](https://github.com/hyperf/db-connection) 组件，它基于 [hyperf/pool](https://github.com/hyperf/pool) 实现了数据库连接池并对模型进行了新的抽象，以它作为桥梁，Hyperf 才能把数据库组件及事件组件接入进来。

### 实现
Hyperf的db使用有两种方式，是否走ORM，ORM会对db查询及响应做多一层封装，底层最终也是通过`connection` 发起db请求。

<!-- more -->

`Hyperf\DbConnection\Model\Model` 继承 `Hyperf\Database\Model/Model` ，所有的db请求通过 `Hyperf\Database\Model\Bulder` 发出：
```php
# vendor/hyperf/database/src/Model/Model.php   
 
/**
 * Handle dynamic method calls into the model.
 *
 * @param string $method
 * @param array $parameters
 */
public function __call($method, $parameters)
{
    if (in_array($method, ['increment', 'decrement'])) {
        return $this->{$method}(...$parameters);
    }

    return call([$this->newQuery(), $method], $parameters)
}

# 最终都是获取到 Hyperf\Database\Model\Bulder 的实例
/**
 * Get a new query builder for the model's table.
 *
 * @return \Hyperf\Database\Model\Builder
 */
public function newQuery()
{
    return $this->registerGlobalScopes($this->newQueryWithoutScopes());
}

/**
 * Get a new query builder that doesn't have any global scopes or eager loading.
 *
 * @return \Hyperf\Database\Model\Builder|static
 */
public function newModelQuery()
{
    return $this->newModelBuilder($this->newBaseQueryBuilder())->setModel($this);
}

/**
 * Get a new query builder instance for the connection.
 *
 * @return \Hyperf\Database\Query\Builder
 */
protected function newBaseQueryBuilder()
{
    $connection = $this->getConnection();

    return new QueryBuilder($connection, $connection->getQueryGrammar(), $connection->getPostProcessor());
}

/**
 * Get the database connection for the model.
 * You can write it by yourself.
 */
public function getConnection(): ConnectionInterface
{
    return Register::resolveConnection($this->getConnectionName());
}
```

如上代码所示，db请求都会进入到 `Hyperf\Database\Query\Builder` ，之后通过自身的`connection` 属性完成db请求，而`connection` 属性由上面最后一个方法注入`Register::*resolveConnection*($this->getConnectionName())` :
```php
# vendor/hyperf/database/src/Model/Register.php
/**
 * Resolve a connection instance.
 *
 * @param null|string $connection
 * @return ConnectionInterface
 */
public static function resolveConnection($connection = null)
{
    return static::$resolver->connection($connection);
}
```

`$resolver` 属性框架启动时注入:
```php
# vendor/hyperf/db-connection/src/Listener/RegisterConnectionResolverListener.php
public function listen(): array
{
    return [
        BootApplication::class,
    ];
}

public function process(object $event)
{
    if ($this->container->has(ConnectionResolverInterface::class)) {
        Register::setConnectionResolver(
            $this->container->get(ConnectionResolverInterface::class)
        );
    }
}
```

`ConnectionResolverInterface::class` 在 `ConfigProvider` 中注入：
```php
# vendor/hyperf/db-connection/src/ConfigProvider.php
public function __invoke(): array
{
    return [
        'dependencies' => [
            PoolFactory::class => PoolFactory::class,
            ConnectionFactory::class => ConnectionFactory::class,
            ConnectionResolverInterface::class => ConnectionResolver::class,
            'db.connector.mysql' => MySqlConnector::class,
            MigrationRepositoryInterface::class => DatabaseMigrationRepositoryFactory::class,
        ],
...
```

`connection` 属性：
```php
# vendor/hyperf/db-connection/src/ConnectionResolver.php
/**
 * Get a database connection instance.
 *
 * @param string $name
 * @return ConnectionInterface
 */
public function connection($name = null)
{
    if (is_null($name)) {
        $name = $this->getDefaultConnection();
    }

    $connection = null;
    $id = $this->getContextKey($name);
    if (Context::has($id)) {
        $connection = Context::get($id);
    }

    if (! $connection instanceof ConnectionInterface) {
        $pool = $this->factory->getPool($name);
        $connection = $pool->get();
        try {
            // PDO is initialized as an anonymous function, so there is no IO exception,
            // but if other exceptions are thrown, the connection will not return to the connection pool properly.
            $connection = $connection->getConnection();
            Context::set($id, $connection);
        } finally {
            if (Coroutine::inCoroutine()) {
                defer(function () use ($connection, $id) {
                    Context::set($id, null);
                    $connection->release();
                });
            }
        }
    }

    return $connection;
}
```
此处使用了连接池，连接池的db实现是 `Hyperf\DbConnection\Pool\DbPool` 和 `Hyperf\DbConnection\Connection` ，具体可以看源码。

上面`connection` 方法使用了`defer` 方法，会在协程结束即请求结束时将db连接放回连接池；