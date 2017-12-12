# Sqlalchemy-migrate使用简介

## 模块功能简介

## 创建数据库版本控制仓库

## 使用流程

1. 创建数据库版本仓库：

  `$ migrate create my_repos_dir "my_repos_name"  `

  `my_repos_name`为仓库的ID，创建后成功会在`my_repos_dir`下生成相应目录和文件：`versions/`用于存放数据库模式的版本信息；`migrate.cfg`是仓库的配置文件；`migrate.py`是一个命令行工具。

2. 对数据库进行版本控制

  数据库的版本控制信息存放在该数据库中，表名为`migrate_version`，使用`version_control`命令工具可将数据库关联到版本控制仓库中：

  `python my_repos_dir/manage.py version_control mysql+mysqlconnector://root:ycj@localhost:3306/migrate my_repos_dir`

  获取版本号，使用`db_version`命令工具，返回的结果是数字，0表示当前数据库为空：

  `python my_repos_dir/manage.py db_version mysql+mysqlconnector://root:ycj@localhost:3306/migrate my_repos_dir`
