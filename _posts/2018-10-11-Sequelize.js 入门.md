---
layout: post
title: 'Sequelize.js 入门'
date: 2018-10-11 20:35:55 +0800
categories: ['编程']
tags: ['数据库', 'sequelize']
author: Alex
permalink: /get-started-with-sequelize
---

`Sequelize`是一款基于 Nodejs 功能强大的异步 ORM 框架，同时支持 PostgreSQL, MySQL, SQLite and MSSQL 多种数据库，很适合作为 Nodejs 后端数据库的存储接口

# 0 说明

- 学习`sequelize.js`
- 官方文档: [http://docs.sequelizejs.com/manual/installation/getting-started.html](http://docs.sequelizejs.com/manual/installation/getting-started.html)
- 本文档对应的`github`路径: [https://github.com/alex-my/javascript-learn/tree/master/sequelize](https://github.com/alex-my/javascript-learn/tree/master/sequelize)
- 本文需要一点点`ES6`的知识，如果你不懂，可以看看阮一峰写的[ECMAScript 6 入门](http://es6.ruanyifeng.com/)
- 本文关于`事务`和`SQL语句`章节因为报错，因此没有写，以后补上
- 小项目不是很喜欢`外键`、`触发器`之类的东西，因此也没有写表关联相关的章节
- 博客: [alex-my.xyz](http://alex-my.xyz), [keylala 在线工具](https://www.keylala.cn)

# 1 环境搭建

- 创建文件夹`sequelize`，然后在文件夹中执行`npm init`，这样就得到一个基础的工程
- 进入文件夹`sequelize`，执行命令`yarn add sequelize mysql2`，也可以使用`npm`命令
- 创建数据库`test_sequelize`

  ```sql
  CREATE DATABASE test_sequelize DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
  ```

- 创建以下文件夹

  - sequelize/src
  - sequelize/src/app
  - sequelize/src/db
  - sequelize/src/utils

- 新增文件`sequelize/src/db/index.js`

  ```javascript
  const Sequelize = require('sequelize');

  const dbConfig = {
    database: 'test_sequelize',
    username: 'root',
    password: '123456',
    host: '127.0.0.1',
    dialect: 'mysql', // 'mysql'|'sqlite'|'postgres'|'mssql'
  };

  const sequelize = new Sequelize(
    dbConfig.database,
    dbConfig.username,
    dbConfig.password,
    {
      host: dbConfig.host,
      dialect: dbConfig.dialect,
      operatorsAliases: false,
      // 设置时区
      timezone: '+08:00',
      pool: {
        max: 5,
        min: 0,
        acquire: 30000,
        idle: 10000,
      },

      // define: {
      // 全局设置引擎, 默认是 InnoDB
      //     engine: 'MYISAM',
      // SQLite only
      // storage: 'path/to/database.sqlite'
    }
  );

  module.exports = sequelize;
  ```

- 新增文件`sequelize/src/app/1-getstarted/index.js`

  ```javascript
  const db = require('../../db');

  db.authenticate()
    .then(() => {
      console.log('Connection has been established successfully.');
    })
    .catch(err => {
      console.error('Unable to connect to the database:', err);
    });
  ```

- 在`sequelize/src/app/1-getstarted`目录下执行`node index.js`，看是否能够连接成功

  ```text
  Executing (default): SELECT 1+1 AS result
  Connection has been established successfully.
  ```

- 新增文件`sequelize/src/utils/random.js`，用于生成随机数

  ```javascript
  const crypto = require('crypto');

  const getRandomString = (len = 16) => {
    const buf = crypto.randomBytes(len);
    return buf.toString('hex');
  };

  module.exports = {
    getRandomString,
  };
  ```

- 至此环境搭建完毕

# 2 第一个示例

- 新增文件`sequelize/src/app/2-firstdemo/index.js`

  ```text
  const sequelize = require('sequelize');
  const db = require('../../db');
  const random = require('../../utils/random');

  // 定义一个 user model
  const {
      STRING
  } = sequelize;

  const User = db.define('user', {
      user_name: {
          type: STRING(32),
      },
      password: {
          type: STRING(32),
      }
  });

  const createTable = async () => {
      // 可以选择在 应用/服务器 启动的时候，把 User 映射到数据库中，比如这里会在数据库中自动创建一张表: users
      // 如果 force 为 true, 则每次都会重建表 users
      await User.sync({
          force: true,
      });
  }

  const createUser = async () => {
      // 创建几个用户
      for (let i = 0; i < 5; i++) {
          const userName = random.getRandomString(8);
          const password = random.getRandomString(16);

          const user = await User.create({
              user_name: userName,
              password
          });
          // sequelize 会默认加上 id, createdAt, updatedAt 字段
          console.log(user.id, user.createdAt, user.updatedAt);
      }
  }

  const queryAll = async () => {
      // 查询所有的结果
      const users = await User.findAll();
      users.forEach((user) => console.log('findAll', user.id, user.user_name));
  }

  (async () => {
      console.log('------------- createTable');
      await createTable();
      console.log('------------- createUser');
      await createUser();
      console.log('------------- queryAll');
      await queryAll();
  })();
  ```

- 在`sequelize/src/app/2-firstdemo`下执行`node index.js`, 可以发现创建了表，创建了角色，并且全部搜索出来了
- 需要注意的是, `Sequelize`使用的是异步，返回值是`Promises`, 比如`const user = User.findOne(); console.log(user.id)`输出的是`undefined`，看日志可以发现，在输出之后才执行的**SELECT `id`, `user_name`, `password`, `createdAt`, `updatedAt` FROM `users` AS `user` LIMIT 1**
- 你需要使用如下方式来获取数据

  ```javascript
  User.findOne().then(user => console.log(user.id));
  ```

- 也可以使用本文的方式，使用`async/await`

  ```javascript
  const user = User.findOne();
  console.log(user.id);
  ```

# 3 模型定义

- 新增文件`sequelize/src/app/3-model-define/index.js`

  ```javascript
  const sequelize = require('sequelize');
  const db = require('../../db');

  const { INTEGER, STRING, DATE, NOW, JSON } = sequelize;

  // 定义一个账号

  const Account = db.define(
    'account',
    {
      // id （实际上 Sequelize 默认会自动生成一个自增 id）
      id: {
        type: INTEGER,
        autoIncrement: true,
        // 做为主键
        primaryKey: true,
      },
      // 账号ID为一个整型，且不能为空
      accountId: {
        type: INTEGER.UNSIGNED,
        allowNull: false,
        // 账号id需要唯一性，这个是永远不能改变的值，特别是在分表的情况下，用自增ID不小心就会重复了
        unique: true,
      },
      // 账号名称，字符串，范围是 4~32
      accountName: {
        // 如果不指定 STRING 长度，则默认是 255
        type: STRING(32),
        validate: {
          // 限制长度范围
          min: 4,
          max: 32,
        },
        // 账号需要唯一性，登录时候使用
        unique: true,
      },
      // 密码
      password: {
        type: STRING(128),
        allowNull: false,
      },
      // 昵称
      nickName: {
        type: STRING(32),
        validate: {
          min: 4,
          max: 32,
        },
        // 假设昵称后要加上 id 值
        get() {
          const id = this.getDataValue('id');
          return this.getDataValue('nickName') + '-' + id;
        },
      },
      // 邮箱
      email: {
        type: STRING(64),
        allowNull: false,
        validate: {
          // 格式必须为 邮箱格式
          isEmail: true,
        },
        // set 假设数据库中存储的邮箱都要是大写的，可以在此处改写
        set(val) {
          this.setDataValue('email', val.toUpperCase());
        },
      },
      // 注册日期 (实际上 Sequelize 默认会自动生成一个 createdAt 字段)
      registerAt: {
        type: DATE,
        defaultValue: NOW,
      },
      // 属性，这里用 JSON 格式写入
      profile: {
        type: JSON,
      },

      // 还支持外键功能, 不过我不喜欢使用, 项目换人了，触发器, 存储过程, 外键 这些东西就比较麻烦了
    },
    {
      getterMethods: {
        // 自定义函数, brief 返回该账号的简要信息
        brief() {
          return `${this.accountId} ${this.accountName} ${this.nickName}`;
        },
      },
      setterMethods: {},

      // classMethods 和 instanceMethods 在 版本4 被移除了
      // 详见: http://docs.sequelizejs.com/manual/tutorial/upgrade-to-v4.html#config-options
      // 定义 类 级别的函数，可以用 Account 调用
      classMethods: {
        getCMethod() {
          return 'classMethods';
        },
      },
      // 定义 实例 级别的函数，用 account 调用
      instanceMethods: {
        getIMethod() {
          return 'instanceMethods';
        },
      },
      // 也可以在 src/db/index 中定义全局的函数

      // 设置为 false 之后，将不会自动加上 createdAt, updatedAt 这两个字段
      timestamps: true,
      // 假设我们需要 创建和更新 这两个字段，但不喜欢驼峰命名法
      // 设置为 true 之后，自动增加的字段就会用下划线命名: created_at, updated_at
      underscored: true,
      // 也可以分别设置 createdAt, updatedAt 是否需要

      // 假设我们喜欢 date 这个名字表示创建时间
      createdAt: 'date',
      // 不想要 updatedAt 这个字段
      updatedAt: false,

      // 设置为 true 之后，则不会真正的删除数据，而是设置 deletedAt
      paranoid: true,
      // 也可以重命名 deletedAt
      deletedAt: 'deleteTime',

      // 定义表名, 默认会生成一个 accounts 的表
      tableName: 'account',

      // 设置引擎格式，默认是 InnoDB 的，这是对单个表有效的
      // engine: 'MYISAM',

      // 写表注释
      comment: '账号表',

      // 可以在定义 字段的时候添加索引，不过在这个地方加索引看的会更清晰
      indexes: [
        // email 需要唯一
        {
          unique: true,
          fields: ['email'],
        },
        // 创建一个 符合索引
        {
          // 默认名字 [table]_[fields]，巨丑
          name: 'select_index',
          fields: ['accountName', 'email', 'password'],
        },
      ],
    }
  );

  const createTable = async () => {
    await Account.sync({
      force: true,
    });
  };

  (async () => {
    console.log('------------- createTable');
    await createTable();
  })();
  ```

- 字段的类型有很多种，比如常用的`STRING`、`TEXT`、`INTEGER`、`FLOAT`、`DOUBLE`、`DATE`，更多的类型请见: [DataTypes](http://docs.sequelizejs.com/variable/index.html#static-variable-DataTypes)
- 在定义模型的时候，可以直接在参数中直接写`get`，`set`函数，这样可以在获取/设置 之前进行一些处理，例如`nickName`中，实际获取的`nickName`后面还加上了`id`值，在`email`中，邮箱全被设置为大写了
- 我们也可以在`新建`，`更新`，`保存`的时候进行一些验证,在字段中的`validate`属性中指定，比如在`email`中会验证是否是`isEmail`，如果在这三个操作之前没有检测到，在操作时候会抛出异常。`Sequelize`使用了[validator.js](https://github.com/chriso/validator.js)进行验证。需要注意的是，如果一个字段允许为`NULL`，即`allowNull:true`，当值为`NULL`的时候，`validate`不会进行验证

# 4 模型使用

- 新增文件`sequelize/src/app/4-model-usage/index.js`

  ```javascript
  const sequelize = require('sequelize');
  const db = require('../../db');

  const { INTEGER, STRING, Op } = sequelize;

  const Role = db.define(
    'role',
    {
      role_id: {
        type: INTEGER.UNSIGNED,
        allowNull: false,
      },
      role_name: {
        type: STRING(32),
        allowNull: false,
        unique: true,
        validate: {
          min: 4,
          max: 32,
        },
      },
      level: {
        type: INTEGER,
        defaultValue: 1,
      },
    },
    {
      underscored: true,
      paranoid: true,
    }
  );

  const createTable = async () => {
    await Role.sync({
      force: true,
    });
  };

  const createRoles = async () => {
    for (let i = 1; i <= 5; i++) {
      await Role.create({
        role_id: i,
        role_name: `name-${i}`,
      });
    }
  };

  const findUsage = async () => {
    // 根据 id 查找
    const role1 = await Role.findById(1);
    console.log(`findUsage/findById id: ${role1.id}, name: ${role1.role_name}`);

    // 根据条件查找一条数据, level=1 的角色有多个
    const role2 = await Role.findOne({
      where: {
        level: 1,
      },
    });
    console.log(`findUsage/findOne id: ${role2.id}, name: ${role2.role_name}`);

    const role3 = await Role.findOne({
      where: {
        level: 1,
      },
      attributes: ['id', 'role_id'],
    });
    // 结果中并不包含 level
    console.log(
      `findUsage/findOne id: ${role3.id}, role_id: ${role3.role_id}, level: ${
        role3.level
      }`
    );

    // 如果数据库中没有，则会按照 defaults 生成一条数据, 并且 created 字段为 true
    // 如果数据库中有该数据，则 created 字段为 false
    // 返回的结果是一个数组, 第一个元素为搜索结果，第二个元素为 boolean，表示是否是新建的数据
    const [role4, created] = await Role.findOrCreate({
      where: {
        role_name: 'alex',
      },
      defaults: {
        role_id: 5,
        role_name: 'alex',
        level: 15,
      },
    });
    console.log(
      `findUsage/findOrCreate created: ${created}, role4: ${JSON.stringify(
        role4
      )}`
    );

    // 查找全部数据，并获得数量
    // 返回值 count: 数据个数
    // 返回值 rows: 包含数据的集合
    const result5 = await Role.findAndCountAll({
      where: {
        level: 1,
      },
    });
    console.log(
      `findUsage/findAndCountAll count: ${
        result5.count
      }, rows: ${JSON.stringify(result5.rows)}`
    );
  };

  const findAllUsage = async () => {
    // 查找全部, 可以在其中放入查询条件和限制条件
    console.log('\r\n');
    await Role.findAll({
      where: {
        id: [1, 2, 3, 4, 5],
      },
      limit: 3,
    });

    // all() 是 findAll 的别名

    console.log('\r\n');
    // where 中可以灵活的设置各种条件
    await Role.all({
      where: {
        level: {
          [Op.gt]: 1,

          // [Op.and]: {a: 5},           // AND (a = 5)
          // [Op.or]: [{a: 5}, {a: 6}],  // (a = 5 OR a = 6)
          // [Op.gt]: 6,                // id > 6
          // [Op.gte]: 6,               // id >= 6
          // [Op.lt]: 10,               // id < 10
          // [Op.lte]: 10,              // id <= 10
          // [Op.ne]: 20,               // id != 20
          // [Op.between]: [6, 10],     // BETWEEN 6 AND 10
          // [Op.notBetween]: [11, 15], // NOT BETWEEN 11 AND 15
          // [Op.in]: [1, 2],           // IN [1, 2]
          // [Op.notIn]: [1, 2],        // NOT IN [1, 2]
          // [Op.like]: '%hat',         // LIKE '%hat'
          // [Op.notLike]: '%hat',      // NOT LIKE '%hat'
        },
      },
      limit: 3,
    });

    // 也可以设置为 OR/NOT 等条件
    console.log('\r\n');
    // ((`role`.`id` IN (1, 2, 3) OR `role`.`id` > 10) AND `role`.`level` = 1)
    await Role.findAll({
      where: {
        level: 1,
        [Op.or]: [
          {
            id: [1, 2, 3],
          },
          {
            id: {
              [Op.gt]: 10,
            },
          },
        ],
      },
    });
    console.log('\r\n');

    // (`role`.`id` IN (1, 2, 3) OR `role`.`id` > 12))
    await Role.findAll({
      where: {
        level: 1,
        id: {
          [Op.or]: [
            [1, 2, 3],
            {
              [Op.gt]: 12,
            },
          ],
        },
      },
    });

    console.log('\r\n');
    // order 排序
    await Role.findAll({
      limit: 2,
      order: [
        ['id'],
        ['role_id', 'ASC'],
        // [sequelize.fn('max', sequelize.col('level')), 'DESC'],
      ],
      // 注意 raw, 默认为 false, 这时候 Sequelize 会为搜索出的每一条数据生成一个 Role 实例，用于更新，删除等操作
      // 但当我们只想搜索出数据用于显示，并不想操作它，这个时候设置 raw: true 就会直接返回数据，而不会生成实例
      raw: true,
    });
  };

  const someUsage = async () => {
    console.log('\r\n');

    // count
    const c1 = await Role.count();
    console.log(`someUsage/count c1: ${c1}`);

    const c2 = await Role.count({
      where: {
        level: {
          [Op.gt]: 1,
        },
      },
    });
    console.log(`someUsage/count c2: ${c2}`);

    // max, min
    const m1 = await Role.max('level', {
      where: {
        id: {
          [Op.gt]: 5,
        },
      },
    });
    console.log(`someUsage/max m1: ${m1}`);

    // sum
    const s1 = await Role.sum('level');
    console.log(`someUsage/sum s1: ${s1}`);
  };

  (async () => {
    console.log('------------- createTable');
    await createTable();
    console.log('------------- createRoles');
    await createRoles();
    console.log('------------- findUsage');
    await findUsage();
    console.log('------------- findAllUsage');
    await findAllUsage();
    console.log('------------- someUsage');
    await someUsage();
  })();
  ```

# 5 查询相关

- SELECT
  - 通过`attributes`可以指定搜索的字段
  ```javascript
  Role.findAll({
    attributes: ['role_id', 'role_name'],
  });
  // SELECT `role_id`, `role_name`
  ```
- AS
  - 也可以在``
  ```javascript
  Role.findAll({
    attributes: ['role_id', ['role_name', 'name']],
  });
  // SELECT `role_id`, `role_name` AS `name` ...
  ```
- COUNT,AVG,MAX,MIN,SUM 等函数

  - 可以使用`sequelize.fn`来执行这些函数

  ```javascript
  Role.findAll({
    attributes: [[sequelize.fn('COUNT', sequelize.col('*')), 'total_count']],
  });
  // SELECT COUNT(*) AS `total_count` ...

  Role.findAll({
    attributes: [[sequelize.fn('COUNT', 1), 'total_count']],
  });
  // SELECT COUNT(1) AS `total_count` ...
  ```

- WHERE

  - 默认是 AND

  ```javascript
  Role.findAll({
    where: {
      id: 5,
      level: 1,
    },
  });
  // ... where `role`.`id` = 5 AND `role`.`level` = 1
  ```

  - OR 的写法

  ```javascript
  Role.findAll({
    where: {
      [Op.or]: [{ id: 2 }, { id: 3 }],
    },
  });
  // ... where `role`.`id` = 2 OR `role`.`id` = 3

  // 另一种写法
  Role.findAll({
    where: {
      id: {
        [Op.or]: [2, 3],
      },
    },
  });
  // ... where `role`.`id` = 2 OR `role`.`id` = 3

  // 不同字段的写法
  Role.findAll({
    where: {
      [Op.or]: [{ id: 2 }, { level: 3 }],
    },
  });
  // ... where `role`.`id` = 2 OR `role`.`level` = 3
  ```

- 复杂点的写法
  ```javascript
  Role.findAll({
    where: {
      [Op.or]: [{ id: 2 }, { level: 3 }],
      [Op.and]: [{ role_id: { [Op.ne]: 10001 } }],
    },
  });
  // ... where (`role`.`id` = 2 OR `role`.`level` = 3) AND (`role`.`role_id` != 10001)
  ```

# 6 实例

- 可以创建一个没有写入数据库的实例，当然，创建之后也可以用`save`命令存入数据库中

  ```javascript
  const role = Role.build({
    role_id: 1,
    role_name: 'name-1',
  });
  console.log(JSON.stringify(role)); // 可以发现, role.id 的值是 NULL
  await role.save(); // 在需要的时候调用 save 命令，可以将数据存入数据库中
  console.log(JSON.stringify(role)); // 这时候，role.id 有值了
  ```

- 也可以直接使用`create`命令直接创建一条写入数据库的数据，这在前文中有多处这样的写法

  ```javascript
  const role = await Role.create({
    role_id: 2,
    role_name: 'name-2',
  });
  console.log(JSON.stringify(role));
  ```

- 批量创建实例

  - 使用`bulkCreate`来批量创建

    ```javascript
    const l = [];
    for (let i = 0; i < 5; i++) {
      l.push({
        role_id: 1000 + i,
        role_name: `bulkname-${i}`,
      });
    }

    const result = await Role.bulkCreate(l);
    console.log('result:', JSON.stringify(result));
    // 输出
    // Executing (default): INSERT INTO `roles` (`id`,`role_id`,`role_name`,`level`,`created_at`,`updated_at`) VALUES (NULL,1000,'bulkname-0',1,'2018-10-15 15:38:29','2018-10-15 15:38:29'),(NULL,1001,'bulkname-1',1,'2018-10-15 15:38:29','2018-10-15 15:38:29'),(NULL,1002,'bulkname-2',1,'2018-10-15 15:38:29','2018-10-15 15:38:29'),(NULL,1003,'bulkname-3',1,'2018-10-15 15:38:29','2018-10-15 15:38:29'),(NULL,1004,'bulkname-4',1,'2018-10-15 15:38:29','2018-10-15 15:38:29');
    // result: [{"id":1,"role_id":1000,"role_name":"bulkname-0","level":1,"created_at":"2018-10-15T07:38:29.902Z","updated_at":"2018-10-15T07:38:29.902Z"},{"id":2,"role_id":1001,"role_name":"bulkname-1","level":1,"created_at":"2018-10-15T07:38:29.902Z","updated_at":"2018-10-15T07:38:29.902Z"},{"id":3,"role_id":1002,"role_name":"bulkname-2","level":1,"created_at":"2018-10-15T07:38:29.902Z","updated_at":"2018-10-15T07:38:29.902Z"},{"id":4,"role_id":1003,"role_name":"bulkname-3","level":1,"created_at":"2018-10-15T07:38:29.902Z","updated_at":"2018-10-15T07:38:29.902Z"},{"id":5,"role_id":1004,"role_name":"bulkname-4","level":1,"created_at":"2018-10-15T07:38:29.902Z","updated_at":"2018-10-15T07:38:29.902Z"}]
    ```

  - 限制字段

    ```javascript
    const l = [];
    for (let i = 0; i < 5; i++) {
      l.push({
        role_id: 1000 + i,
        role_name: `bulkname-${i}`,
        level: i + 5,
      });
    }

    const result = await Role.bulkCreate(l, {
      // 这样创建语句中只有 role_id 和 role_name，会忽略 level
      fields: ['role_id', 'role_name'],
    });
    ```

- 更新

  - 更新表中符合条件的数据(批量更新)

    ```javascript
    const result = await Role.update(
      {
        level: 4,
      },
      {
        where: {},
      }
    );
    console.log('result: ', JSON.stringify(result));
    // 输出
    // Executing (default): UPDATE `roles` SET `level`=4,`updated_at`='2018-10-15 06:51:07' WHERE (`deleted_at` > '2018-10-15 06:51:07' OR `deleted_at` IS NULL)
    // result:  [2]
    ```

  - 更新具体的实例

    ```javascript
    const role = await Role.findOne({
      where: {
        id: 1,
      },
    });
    const result2 = await role.update({
      level: 5,
    });
    // 输出
    // Executing (default): SELECT `id`, `role_id`, `role_name`, `level`, `created_at`, `updated_at`, `deleted_at` FROM `roles` AS `role` WHERE ((`role`.`deleted_at` > '2018-10-15 06:51:07' OR `role`.`deleted_at` IS NULL) AND `role`.`id` = 1);
    // Executing (default): UPDATE `roles` SET `level`=5,`updated_at`='2018-10-15 06:51:07' WHERE `id` = 1
    // result2:  {"id":1,"role_id":1,"role_name":"name-1","level":5,"created_at":"2018-10-15T06:36:26.000Z","updated_at":"2018-10-15T06:51:07.935Z","deleted_at":null}
    ```

  - 更新未写入数据库的数据，相当于创建了一条新数据

    ```javascript
    const role3 = Role.build({
      role_id: 3,
      role_name: 'name-3',
    });
    const result3 = await role3.update({
      role_id: 4,
      role_name: 'name-4',
      level: 4,
    });
    console.log('result3: ', JSON.stringify(result3));
    // 实际上执行的是: INSERT INTO `roles` (`role_id`,`role_name`,`level`,`updated_at`,`created_at`) VALUES (4,'name-4',4,'2018-10-15 06:57:28','2018-10-15 06:57:28')
    ```

- 删除

  - 如果我们启用了软删除，即设置了`paranoid:true`(需要`timestamps:true`)，则不会真正的将数据从数据库中删除，而是设置了`deleted_at`
  - 将表中符合条件的数据删除(批量删除)

    ```javascript
    const result1 = await Role.destroy({
      where: {
        id: 1,
      },
    });
    console.log('result1:', result1);
    // 输出
    // Executing (default): UPDATE `roles` SET `deleted_at`='2018-10-15 15:02:50' WHERE `deleted_at` IS NULL AND `id` = 1
    // result1: 1
    ```

  - 将具体的实例进行删除，这里设置了`force:true`，进行硬删除，把数据从数据库中删除

    ```javascript
    const role2 = await Role.findOne({
      where: {
        id: 2,
      },
    });
    // 可以设置 force: true, 使得数据真正从数据库中删除
    const result2 = await role2.destroy({
      force: true,
    });
    console.log('result2:', result2);
    // 输出
    // DELETE FROM `roles` WHERE `id` = 2 LIMIT 1
    ```

  - 删除未写入数据库的实例进行删除

    ```javascript
    // 软删除
    Role.build({
      role_id: 5,
      role_name: 'name-5',
    }).destroy();
    // 输出
    // UPDATE `roles` SET `deleted_at`='2018-10-15 15:14:28' WHERE `id` IS NULL AND `deleted_at` IS NULL

    // 硬删除
    Role.build({
      role_id: 5,
      role_name: 'name-5',
    }).destroy({
      force: true,
    });
    // 输出
    // DELETE FROM `roles` WHERE `id` IS NULL LIMIT 1
    ```

- 增减

  - 增加值使用`increment`, 减少值使用`decrement`，用法相同
  - `level`加 5，可以看到`result`中的`level`还是原来的数据

    ```javascript
    const role = await Role.findById(1);
    const result = await role.increment('level', {
      by: 5,
    });
    console.log('result:', JSON.stringify(result));
    // 输出
    // Executing (default): UPDATE `roles` SET `level`=`level`+ 5,`updated_at`='2018-10-15 16:04:35' WHERE `id` = 1
    // result: {"id":1,"role_id":1000,"role_name":"bulkname-0","level":1,"created_at":"2018-10-15T07:52:06.000Z","updated_at":"2018-10-15T07:52:06.000Z","deleted_at":null}
    ```

# 7 事务

- 事务分为两种:
  - 一种需要我们手动执行`commit`和`rollback`命令进行提交和回滚
  - 另一种是自动管理的，只要返回值都是成功`resolved`，则会一个个执行下去，一旦出现一个`rejected`，就会自行回滚

# 8 钩子

- 钩子列表

  ```text
  (1)
    beforeBulkCreate(instances, options)
    beforeBulkDestroy(options)
    beforeBulkUpdate(options)
  (2)
    beforeValidate(instance, options)
  (-)
    validate
  (3)
    afterValidate(instance, options)
    - or -
    validationFailed(instance, options, error)
  (4)
    beforeCreate(instance, options)
    beforeDestroy(instance, options)
    beforeUpdate(instance, options)
    beforeSave(instance, options)
    beforeUpsert(values, options)
  (-)
    create
    destroy
    update
  (5)
    afterCreate(instance, options)
    afterDestroy(instance, options)
    afterUpdate(instance, options)
    afterSave(instance, options)
    afterUpsert(created, options)
  (6)
    afterBulkCreate(instances, options)
    afterBulkDestroy(options)
    afterBulkUpdate(options)
  ```

- 完整列表见: [Hooks file](https://github.com/sequelize/sequelize/blob/master/lib/hooks.js#L7)
- 新增文件`sequelize/src/app/7-hooks/index.js`

  ```javascript
  const sequelize = require('sequelize');
  const db = require('../../db');

  const { INTEGER, STRING } = sequelize;

  const Role = db.define(
    'role',
    {
      role_id: {
        type: INTEGER.UNSIGNED,
        allowNull: false,
      },
      role_name: {
        type: STRING(32),
        allowNull: false,
        unique: true,
        validate: {
          min: 4,
          max: 32,
        },
      },
      level: {
        type: INTEGER,
        defaultValue: 1,
      },
    },
    {
      underscored: true,
      paranoid: true,
      // 使用钩子方法1: 在定义的时候新增钩子
      hooks: {
        beforeCreate: (role, options) => {
          console.log('beforeCreate ...');
          role.level = 100;
        },
        // 可以写多个钩子
      },
    }
  );

  // 使用钩子方法2
  Role.hook('beforeValidate', (role, options) => {
    console.log('beforeValidate ...');
  });

  // 使用钩子方法3
  Role.afterCreate((role, options) => {
    console.log('afterCreate ...');
    console.log('role created success,', JSON.stringify(role));
  });

  const createTable = async () => {
    await Role.sync({
      force: true,
    });
  };

  const createRole = async () => {
    await Role.create({
      role_id: 1,
      role_name: 'name-1',
    });

    // 输出
    // beforeValidate ...
    // beforeCreate ...
    // Executing (default): INSERT INTO `roles` (`id`,`role_id`,`role_name`,`level`,`created_at`,`updated_at`) VALUES (DEFAULT,1,'name-1',100,'2018-10-15 17:08:52','2018-10-15 17:08:52');
    // afterCreate ...
    // role created success, {"level":100,"id":1,"role_id":1,"role_name":"name-1","updated_at":"2018-10-15T09:08:52.923Z","created_at":"2018-10-15T09:08:52.923Z"}
  };

  (async () => {
    console.log('------------- createTable');
    await createTable();
    console.log('------------- createRole');
    await createRole();
  })();
  ```

# 9 SQL 语句

- 在`sequelize`中还可以直接使用`sql`语句，但在测试报错: `TypeError: sequelize.query is not a function`，因此后续还有个功能: 替换,`用 ? 或者 :key 替换值` 就没有继续研究了
- 新增文件`sequelize/src/app/8-sql/index.js`

  ```javascript
  const sequelize = require('sequelize');
  const db = require('../../db');

  const { INTEGER, STRING } = sequelize;

  const Role = db.define(
    'role',
    {
      role_id: {
        type: INTEGER.UNSIGNED,
        allowNull: false,
      },
      role_name: {
        type: STRING(32),
        allowNull: false,
        unique: true,
        validate: {
          min: 4,
          max: 32,
        },
      },
      level: {
        type: INTEGER,
        defaultValue: 1,
      },
    },
    {
      underscored: true,
      paranoid: true,
    }
  );

  const createTable = async () => {
    await Role.sync({
      force: true,
    });
  };

  const createRoles = async () => {
    const l = [];
    for (let i = 1; i <= 5; i++) {
      l.push({
        role_id: i,
        role_name: `name-${i}`,
        level: i + 5,
      });
    }
    await Role.bulkCreate(l);
  };

  const rawQueryUsage = async () => {
    // 实际上报错: 找不到 sequelize.query 这个函数

    const result1 = await sequelize.query('SELECT * FROM roles');
    console.log('result 1:', JSON.stringify(result1));
    console.log('\r\n');
    const result2 = await sequelize.query('SELECT * FROM roles', {
      type: sequelize.QueryTypes.SELECT,
    });
    console.log('result 2:', JSON.stringify(result2));
  };

  (async () => {
    console.log('------------- createTable');
    await createTable();
    console.log('------------- createRoles');
    await createRoles();
    console.log('------------- rawQueryUsage');
    await rawQueryUsage();
  })();
  ```
