# ThinkPHP V5.0 学习笔记

[TOC]

## 基础

### 安装

### 编码规范

#### 目录和文件

- 目录使用小写+下划线；
- 类库、函数文件统一以`.php`为后缀；
- 类的文件名均以命名空间定义，并且命名空间的路径和类库文件所在路径一致；
- 类文件采用驼峰法命名（首字母大写），其它文件采用小写+下划线命名；
- 类名和类文件名保持一致，统一采用驼峰法命名（首字母大写）；

#### 函数和类、属性命名

- 类的命名采用驼峰法（首字母大写），例如 `User`、`UserType`，默认不需要添加后缀，例如`UserController`应该直接命名为`User`；
- 函数的命名使用小写字母和下划线（小写字母开头）的方式，例如 `get_client_ip`；
- 方法的命名使用驼峰法（首字母小写），例如 `getUserName`；
- 属性的命名使用驼峰法（首字母小写），例如 `tableName`、`instance`；
- 以双下划线“__”打头的函数或方法作为魔术方法，例如 `__call` 和 `__autoload`；

#### 常量和配置

- 常量以大写字母和下划线命名，例如 `APP_PATH`和 `THINK_PATH`；
- 配置参数以小写字母和下划线命名，例如 `url_route_on` 和`url_convert`；

#### 数据表和字段

- 数据表和字段采用小写加下划线方式命名，并注意字段名不要以下划线开头，例如 `think_user` 表和 `user_name`字段，不建议使用驼峰和中文作为数据表字段命名。

#### 应用类库命名空间规范

应用类库的根命名空间统一为app（不建议更改，可以设置`app_namespace`配置参数更改，`V5.0.8`版本开始使用`APP_NAMESPACE`常量定义）；
例如：`app\index\controller\Index`和`app\index\model\User`。

### 目录结构

```
project  应用部署目录
├─application           应用目录（可设置）
│  ├─common             公共模块目录（可更改）
│  ├─index              模块目录(可更改)
│  │  ├─config.php      模块配置文件
│  │  ├─common.php      模块函数文件
│  │  ├─controller      控制器目录
│  │  ├─model           模型目录
│  │  ├─view            视图目录
│  │  └─ ...            更多类库目录
│  ├─command.php        命令行工具配置文件
│  ├─common.php         应用公共（函数）文件
│  ├─config.php         应用（公共）配置文件
│  ├─database.php       数据库配置文件
│  ├─tags.php           应用行为扩展定义文件
│  └─route.php          路由配置文件
├─extend                扩展类库目录（可定义）
├─public                WEB 部署目录（对外访问目录）
│  ├─static             静态资源存放目录(css,js,image)
│  ├─index.php          应用入口文件
│  ├─router.php         快速测试文件
│  └─.htaccess          用于 apache 的重写
├─runtime               应用的运行时目录（可写，可设置）
├─vendor                第三方类库目录（Composer）
├─thinkphp              框架系统目录
│  ├─lang               语言包目录
│  ├─library            框架核心类库目录
│  │  ├─think           Think 类库包目录
│  │  └─traits          系统 Traits 目录
│  ├─tpl                系统模板目录
│  ├─.htaccess          用于 apache 的重写
│  ├─.travis.yml        CI 定义文件
│  ├─base.php           基础定义文件
│  ├─composer.json      composer 定义文件
│  ├─console.php        控制台入口文件
│  ├─convention.php     惯例配置文件
│  ├─helper.php         助手函数文件（可选）
│  ├─LICENSE.txt        授权说明文件
│  ├─phpunit.xml        单元测试配置文件
│  ├─README.md          README 文件
│  └─start.php          框架引导文件
├─build.php             自动生成定义文件（参考）
├─composer.json         composer 定义文件
├─LICENSE.txt           授权说明文件
├─README.md             README 文件
├─think                 命令行入口文件
```

## 架构

### 架构总览

### 生命周期

#### 1. 入口文件

#### 2. 引导文件

`start.php`文件就是系统默认的一个引导文件。在引导文件中，会依次执行下面操作：

- 加载系统常量定义；
- 加载环境变量定义文件；
- 注册自动加载机制；
- 注册错误和异常处理机制；
- 加载惯例配置文件；
- 执行应用；

#### 3. 注册自动加载

系统的自动加载由下面主要部分组成：

1. 注册系统的自动加载方法 **\think\Loader::autoload**
2. 注册系统命名空间定义
3. 加载类库映射文件（如果存在）
4. 如果存在`Composer`安装，则注册**`Composer`**自动加载
5. 注册`extend`扩展目录

一个类库的自动加载检测顺序为：

1. 是否定义类库映射；
2. `PSR-4`自动加载检测；
3. `PSR-0`自动加载检测；

#### 4. 注册错误和异常机制

执行`Error::register()`注册错误和异常处理机制。

由三部分组成：

- 应用关闭方法：`think\Error::appShutdown`
- 错误处理方法：`think\Error::appError`
- 异常处理方法：`think\Error::appException`

#### 5. 应用初始化

执行应用的第一步操作就是对应用进行初始化，包括：

- 加载应用（公共）配置；
- 加载扩展配置文件（由`extra_config_list`定义）；
- 加载应用状态配置；
- 加载别名定义；
- 加载行为定义；
- 加载公共（函数）文件；
- 注册应用命名空间；
- 加载扩展函数文件（由`extra_file_list`定义）；
- 设置默认时区；
- 加载系统语言包；

#### 6. URL访问检测

#### 7. 路由检测

#### 8. 分发请求



### 入口文件

### URL访问

### 模块设计

### 命名空间

### 自动加载

### Traits引入

### API友好

## 配置

## 路由

## 控制器

### 控制器定义

### 控制器初始化

### 前置操作

### 跳转和重定向

### 空操作

### 空控制器

### 多级控制器

### 分层控制器

### Rest控制器

### 自动定位控制器

### 资源控制器

