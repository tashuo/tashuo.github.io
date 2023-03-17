---
title: session的开启与关闭
date: 2017-02-08 16:52:50
categories:
- 笔记
- PHP
tags:
- session
---

### session start
手工开启之前需先判断是否已经启动, 两种判断:

##### PHP版本 < 5.4.0
```php
<?php
    // 通过session_id亦可适用于高版本
    if (session_id() == '') { 
        session_start(); 
    }
?>
```

##### PHP版本 >= 5.4.0
```php
<?php
    // session_status最低只支持5.4.0的PHP版本
    if (session_status() == PHP_SESSION_NONE) { 
        session_start(); 
    }
?>
```

即:
```php
<?php
    // 检测session是否开启
    function hasSessionStarted() {
        if (version_compare(phpversion(), '5.4.0', '<')) {
            $session_id = session_id();
            return !empty($session_id);
        } else {
            return session_status() != PHP_SESSION_NONE;
        }
    }
```

<!-- more -->

### session destrory(完整的logout姿势)

```php
<?php
    // 重置会话中的所有变量
    $_SESSION = array();
    
    // 如果要清理的更彻底，那么同时删除会话 cookie
    // 注意：这样不但销毁了会话中的数据，还同时销毁了会话本身
    if (ini_get("session.use_cookies")) {
        $params = session_get_cookie_params();
        setcookie(session_name(), '', time() - 42000,
            $params["path"], $params["domain"],
            $params["secure"], $params["httponly"]
        );
    }
    
    // 最后，销毁会话
    session_destroy();
?>
```


