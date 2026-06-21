# fs 模块

### 引入

```javascript
const fs = require("fs");
// fs 模块提供了 promises 版本的 API
const fs = require("fs").promises;
```

### fs.readFile

`fs.readFile` 以异步的方式读取整个文件内容，在读取完成后通过回调函数返回结果。这意味着在文件读取过程中，Node.js 可以继续执行其他代码，不会阻塞事件循环。

```javascript
fs.readFile(path[, options], callback)
```

#### 参数说明

-   **`path`**：表示要读取的文件的路径，可以是字符串、`Buffer` 或 `URL` 对象。

-   `options`

    （可选）：一个对象或字符串，用于指定读取文件的选项，常用属性如下：

    -   **`encoding`**：指定文件内容的编码格式，例如 `'utf8'` 表示以 UTF-8 编码读取文件。如果不指定该属性，读取的结果将是一个 `Buffer` 对象。
    -   **`flag`**：指定文件的打开标志，默认值是 `'r'`，表示以只读模式打开文件。

-   `callback`

    ：读取完成后的回调函数，它接收两个参数：

    -   **`err`**：如果读取过程中发生错误，该参数将包含错误对象；如果没有错误，该参数为 `null`。
    -   **`data`**：读取到的文件内容，如果指定了 `encoding` 选项，`data` 是一个字符串；否则，`data` 是一个 `Buffer` 对象。

### fs.writeFile

`fs.writeFile` 以异步的方式将数据写入文件。如果文件不存在，它会创建该文件；如果文件已存在，默认情况下会覆盖原有内容。在写入完成后，通过回调函数返回结果。这种异步操作方式不会阻塞 Node.js 的事件循环，在文件写入过程中可以继续执行其他代码

```
fs.writeFile(file, data[, options], callback)
```

#### 参数说明

-   **`file`**：表示要写入的文件的路径，可以是字符串、`Buffer` 或 `URL` 对象。

-   **`data`**：要写入文件的数据，可以是字符串、`Buffer`、`TypedArray` 或 `DataView`。

-   `options`

    （可选）：一个对象或字符串，用于指定写入文件的选项，常用属性如下：

    -   **`encoding`**：指定写入数据的编码格式，默认值是 `'utf8'`。
    -   **`mode`**：指定文件的权限，默认值是 `0o666`。
    -   **`flag`**：指定文件的打开标志，默认值是 `'w'`，表示以写入模式打开文件，如果文件存在则覆盖，不存在则创建。其他常用标志还有 `'a'` 表示追加写入。

-   `callback`

    ：写入完成后的回调函数，它接收一个参数：

    -   **`err`**：如果写入过程中发生错误，该参数将包含错误对象；如果没有错误，该参数为 `null`。

### \_\_dirname

`__dirname` 是一个全局变量，它代表当前执行脚本所在的目录的绝对路径

# path 模块

### 引入

```javascript
const path = require("path");
```

### path.join([...paths])

用于将多个路径片段拼接成一个规范化的路径。它会处理路径分隔符，确保在不同操作系统上都能正常工作

```javascript
const filePath = path.join("home", "user", "documents", "file.txt");
console.log(filePath);
// 在 POSIX 系统上输出: home/user/documents/file.txt
// 在 Windows 系统上输出: home\user\documents\file.txt
```

### path.resolve([...paths])

将路径或路径片段解析为绝对路径。它从右到左处理路径片段，直到构建出一个绝对路径。如果没有提供绝对路径片段，它会使用当前工作目录

```javascript
const absolutePath = path.resolve("home", "user", "documents", "file.txt");
console.log(absolutePath); // 输出当前工作目录下的绝对路径，例如: /Users/currentuser/home/user/documents/file.txt
```

### path.basename(path[, ext])

返回路径的最后一部分，即文件名。可以通过第二个参数指定文件扩展名，若指定则会将其从返回值中去除。

```javascript
const fullPath = "/home/user/documents/file.txt";
const fileName = path.basename(fullPath);
console.log(fileName); // 输出: file.txt

const withoutExt = path.basename(fullPath, ".txt");
console.log(withoutExt); // 输出: file
```

### path.dirname(path)

返回路径的目录名，即路径中除最后一部分（文件名）之外的部分。

```javascript
const fullPath = "/home/user/documents/file.txt";
const dirName = path.dirname(fullPath);
console.log(dirName); // 输出: /home/user/documents
```

### path.extname(path)

返回路径的文件扩展名，包括点号（`.`）。如果路径中没有扩展名，则返回空字符串。

```javascript
const fullPath = "/home/user/documents/file.txt";
const ext = path.extname(fullPath);
console.log(ext); // 输出: .txt

const noExtPath = "/home/user/documents";
const noExt = path.extname(noExtPath);
console.log(noExt); // 输出: ''
```

### path.normalize(path)

规范化给定的路径，解析 `.` 和 `..` 等相对路径部分，并处理多余的分隔符。

```javascript
const pathToNormalize = "/home/user/../documents/./file.txt";
const normalizedPath = path.normalize(pathToNormalize);
console.log(normalizedPath);
// 在 POSIX 系统上输出: /home/documents/file.txt
// 在 Windows 系统上输出: \home\documents\file.txt
```

# HTTP 模块

### 引入

```javascript
const http = require("http");
```

### 创建一个 http 服务器实例

```javascript
const http = require("http");
// 创建一个 HTTP 服务器实例
const server = http.createServer((req, res) => {
	//...
});
// 让服务器监听 3000 端口
server.listen(3000, () => {
	console.log("Server is running on port 3000");
});
```

-   `http.createServer` 方法用于创建一个 HTTP 服务器实例，它接受一个回调函数作为参数，该回调函数会在每次接收到请求时被调用。
-   `req` 参数是一个 `http.IncomingMessage` 对象，它包含了客户端请求的信息，如请求方法、请求头、请求 URL 等。
-   `res` 参数是一个 `http.ServerResponse` 对象，用于向客户端发送响应。

```javascript
server.on("request", (req, res) => {
	let body = "";
	/**
	 * req.method 获取请求的方法
	 * req.url 获取请求的URL地址
	 * req.headers 获取请求头
	 * req.setTimeout(timeout[, callback]) 设置请求的超时时间
	 */
	// 获取请求体数据
	req.on("data", (chunk) => {
		// chunk 是请求体的数据片段
		body += chunk;
	});
	// 当请求体数据全部接收完毕时触发
	req.on("end", () => {
		console.log("完整的请求体数据:", body);
	});
	/**
	 * res.statusCode 设置响应状态码
	 * res.statusMessage 设置响应状态信息
	 * res.setHeader(name, value) 设置响应头
	 * res.getHeader(name) 获取响应头
	 * res.writeHead(statusCode[, statusMessage][, headers]) 设置响应头和状态码
	 * res.write(chunk[, encoding][, callback]) 写入响应体数据
	 * res.end([data[, encoding]][, callback]) 结束响应
	 * res.destroy([error]) 销毁响应
	 * res.on('finish') 当响应结束时触发
	 */
});
```

# Express 模块

### 初始化与安装

```
npm init
npm install express
```

### 路由

#### 路由参数

当客户端访问 `/users/123` 时，`req.params.id` 的值将是 `123`

```javascript
// 定义带有路由参数的 GET 请求
app.get("/users/:id", (req, res) => {
	const userId = req.params.id;
	res.send(`You requested user with ID: ${userId}`);
});
```

#### 查询字符串参数

当客户端访问 `/search?keyword=apple` 时，`req.query.keyword` 的值将是 `apple`

```javascript
// 处理带有查询字符串参数的 GET 请求
app.get("/search", (req, res) => {
	const keyword = req.query.keyword;
	res.send(`You searched for: ${keyword}`);
});
```

#### 路由处理多个回调函数

```javascript
app.get(
	"/multiple",
	(req, res, next) => {
		console.log("First middleware");
		next();
	},
	(req, res) => {
		res.send("Response from the second middleware");
	}
);
```

#### 路由模块的模块化

**创建一个路由模块** **`userRoutes.js`**

```javascript
const express = require("express");
const router = express.Router();

// 定义用户相关的路由
router.get("/users", (req, res) => {
	res.send("List of users");
});

router.get("/users/:id", (req, res) => {
	const userId = req.params.id;
	res.send(`User with ID: ${userId}`);
});

module.exports = router;
```

**在主应用中使用路由模块**

```javascript
const express = require("express");
const app = express();
const userRoutes = require("./userRoutes");

// 使用用户路由模块
app.use("/api", userRoutes);

const port = 3000;
app.listen(port, () => {
	console.log(`Server is running on port ${port}`);
});
```

#### 路由中间件

##### 使用

中间件是在路由处理函数之前执行的函数，可用于执行各种任务，如日志记录、身份验证等。可以在路由定义中使用中间件，也可以全局使用

```javascript
/**
 * 中间件
 */
const express = require("express");
const app = express();
const port = 8080;

// 定义一个简单的中间件
const logger = (req, res, next) => {
	console.log(`Received ${req.method} request for ${req.url}`);
	next();
};

const mm = (req, res, next) => {
	console.log(`局部中间件`);
	next();
};

// 定义错误中间件 （必须放在路由之后）
const errorMiddleware = (err, req, res, next) => {
	console.error(err.stack);
	res.status(500).send("Something broke!");
};

// 全局使用中间件
app.use(logger);
// 定义路由
app.get("/middleware", (req, res) => {
	res.send("This route uses 全局中间件");
});

// 局部使用中间件
app.get("/middleware2", mm, (req, res) => {
	res.send("This route uses 局部中间件");
});
// 局部使用多个中间件
app.get("/middleware3", [mm, mm], (req, res) => {
	res.send("This route uses 多个局部中间件");
});

app.listen(port, () => {
	console.log(`Example app listening on port ${port}`);
});
```

##### express 内置中间件

-   express.static

    用于静态文件服务。它可以将指定目录下的文件（如 HTML、CSS、JavaScript、图片等）直接提供给客户端访问

    ```javascript
    const express = require("express");
    const app = express();
    app.use(express.static("pulice"));
    // 访问
    // http://localhost:3000/images/bg.jpn
    ```

-   express.json

    该中间件用于解析 JSON 格式的请求体。当客户端发送的请求体是 JSON 数据时，使用 `express.json` 可以将其解析为 JavaScript 对象，并挂载到 `req.body` 上，方便后续处理

    ```javascript
    const express = require("express");
    const app = express();

    // 使用 express.json 中间件解析 JSON 请求体
    app.use(express.json());

    app.post("/data", (req, res) => {
    	const data = req.body;
    	res.send(`Received data: ${JSON.stringify(data)}`);
    });

    const port = 3000;
    app.listen(port, () => {
    	console.log(`Server is running on port ${port}`);
    });
    ```

-   express.urlencoded

    用于解析 `application/x-www-form-urlencoded` 格式的请求体，这种格式通常用于 HTML 表单提交。解析后的数据会挂载到 `req.body` 上

    ```javascript
    const express = require("express");
    const app = express();
    
    // 使用 express.urlencoded 中间件解析表单数据
    app.use(express.urlencoded({ extended: true }));
    
    app.post("/form", (req, res) => {
    	const formData = req.body;
    	res.send(`Received form data: ${JSON.stringify(formData)}`);
    });
    
    const port = 3000;
    app.listen(port, () => {
    	console.log(`Server is running on port ${port}`);
    });
    ```



### cors 跨域

#### 安装

```
npm install cors
```

#### 使用

```
// 使用cors中间件，允许所有来源的请求(一定要放在路由之前)
app.use(cors());
```



### 身份认证

#### 1.session 认证

安装

```js
npm install express-session
```

使用

```js
const express = require("express");
const session = require("express-session");

const app = express();
// 使用 express.json 中间件解析 JSON 请求体
app.use(express.json());
// 使用 express.urlencoded 中间件解析表单数据
app.use(express.urlencoded({ extended: true }));
// 配置 session 中间件
app.use(
	session({
		secret: "your_secret_key", // 用于对 session ID 进行签名的密钥
		resave: false, // 是否在每次请求时重新保存 session
		saveUninitialized: true, // 是否保存新创建但未修改的 session
		cookie: { maxAge: 3600000 }, // session 过期时间，单位为毫秒
	})
);

// 登录接口
app.post("/login", (req, res) => {
	// 模拟用户验证
	const { username, password } = req.body;
	if (username === "admin" && password === "123456") {
		// 验证成功，设置 session
		req.session.user = { username };
		req.session.isLogin = true;
		res.send("登录成功");
	} else {
		res.status(401).send("用户名或密码错误");
	}
});

// 受保护的接口
app.get("/protected", (req, res) => {
	console.log("req.session", req.session);
	/**
    {
        cookie: {
          path: '/',
          _expires: 2025-03-21T08:15:03.559Z,
          originalMaxAge: 3600000,
          httpOnly: true
        },
        user: { username: 'admin' },
        isLogin: true
      }
     */

	if (req.session.isLogin) {
		res.send(`欢迎，${req.session.user.username}`);
	} else {
		res.status(401).send("未登录，请先登录");
	}
});

// 退出登录接口
app.get("/logout", (req, res) => {
	// 销毁 session
	req.session.destroy((err) => {
		if (err) {
			console.error("销毁 session 时出错:", err);
		}
		res.send("已注销");
	});
});

const port = 8080;
app.listen(port, () => {
	console.log(`服务器运行在端口 ${port}`);
});
```

#### 2.JSON Web Token（JWT）认证

##### 安装

```js
npm install jsonwebtoken express-jwt
```

##### 说明

JWT 基于 JSON 数据格式，由服务器生成并发送给客户端，客户端在后续请求中携带该令牌，服务器通过验证令牌的有效性来识别用户身份和授权信息

JWT 通常由三部分组成，分别是头部（Header）、载荷（Payload）和签名（Signature），它们之间用点号（.）分隔。

- **头部**：一般包含两部分信息，令牌的类型（如 `JWT`）和所使用的签名算法（如 `HMAC SHA256` 或 `RSA`）。例如：`{"alg":"HS256","typ":"JWT"}`。
- **载荷**：是存放有效信息的地方，这些信息称为声明（Claims）。包含用户相关的信息，如用户 ID、用户名、角色等，也可以包含一些其他的元数据，如令牌的过期时间、颁发时间等。例如：`{"sub":"1234567890","name":"John Doe","admin":true,"iat":1691587200,"exp":1691673600}`。
- **签名**：是对头部和载荷进行签名的结果，用于验证令牌的完整性和真实性。服务器使用一个只有它自己知道的密钥对头部和载荷进行签名，客户端在接收到令牌后，可以使用相同的算法和密钥来验证签名的有效性。

##### 使用

```js
const express = require("express");
const cors = require("cors");
// 导入专门生成JWT的包
const jwt = require("jsonwebtoken");
// 将JWT字符串解析还原成JSON对象
const { expressjwt: expressJwt } = require("express-jwt");

const app = express();

const secretKey = "mysecretKey";
console.log(expressJwt);

// 验证通过后，JWT 中的有效载荷会挂载到 req.auth (express-jwt 7.x 及以上版本)
app.use(
	expressJwt({ secret: secretKey, algorithms: ["HS256"] }).unless({
		path: ["/login"],
	})
);
app.use(cors());
// 解析 JSON 请求体
app.use(express.json());
// 解析 URL 编码请求体
app.use(express.urlencoded({ extended: true }));

// 用户登录成功后，生成JWT
function generateToken(user) {
	const payload = {
		username: user.username,
		// 可以根据需要添加更多用户信息
	};

	// 设置JWT的过期时间
	const options = {
		expiresIn: "30s", // 30s后过期
	};

	// 生成JWT
	const token = jwt.sign(payload, secretKey, options);
	return token;
}

// 登录接口
app.post("/login", (req, res) => {
	const { username, password } = req.body;
	if (username === "admin" && password === "123456") {
		res.send({
			code: 200,
			message: "登录成功",
			token: "Bearer " + generateToken({ username }),
		});
	} else {
		res.status(401).send("用户名或密码错误");
	}
});
// 需要认证的接口
app.get("/admin", (req, res) => {
	console.log(req.auth);
	// 加header Authorization: Bearer token
	res.send({
		code: 200,
		message: "获取数据成功",
		data: req.auth,
	});
});

// 错误处理中间件
app.use((err, req, res, next) => {
	if (err.name === "UnauthorizedError") {
		res.status(401).json({ message: "Invalid token" });
	} else {
		next(err);
	}
});

const port = 8080;
app.listen(port, () => {
	console.log(`Server is running on port ${port}`);
});
```





# Mysql 数据库

### 安装

```
npm install mysql2
```

### 基本使用

`mysql2` 提供了两种连接数据库的方式：**单连接**和**连接池**。

#### 单连接

单连接适用于简单的应用场景，每次操作都创建一个新的连接。

```js
const mysql = require('mysql2');

// 创建连接
const connection = mysql.createConnection({
    host: 'localhost',
    user: 'your_username',
    password: 'your_password',
    database: 'your_database'
});

// 连接数据库
connection.connect((err) => {
    if (err) {
        console.error('连接数据库失败:', err);
        return;
    }
    console.log('成功连接到数据库');
    // 在这里可以执行数据库操作
    connection.end(); // 关闭连接
});
```

#### 连接池

连接池适用于高并发的应用场景，它可以复用连接，提高性能。

##### 使用 `query` 方法

```js
const mysql = require('mysql2');

const connection = mysql.createConnection({
    host: 'localhost',
    user: 'your_username',
    password: 'your_password',
    database: 'your_database'
});

const sql = 'SELECT * FROM users WHERE age > ?';
const values = [18];

connection.query(sql, values, (err, results) => {
    if (err) {
        console.error('查询出错:', err);
        return;
    }
    console.log('查询结果:', results);
    connection.end();
});
```

##### 使用 `execute` 方法

`execute` 方法支持预处理语句，可以防止 SQL 注入攻击。

```js
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
    host: 'localhost',
    user: 'your_username',
    password: 'your_password',
    database: 'your_database'
});

const sql = 'SELECT * FROM users WHERE age > ?';
const values = [18];

(async () => {
    try {
        const [rows] = await pool.execute(sql, values);
        console.log('查询结果:', rows);
    } catch (error) {
        console.error('查询数据库出错:', error);
    }
})();
```

#### 事务处理

事务可以确保一组数据库操作要么全部成功，要么全部失败。

```js
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
    host: 'localhost',
    user: 'your_username',
    password: 'your_password',
    database: 'your_database'
});

(async () => {
    const connection = await pool.getConnection();
    try {
        // 开始事务
        await connection.beginTransaction();

        // 执行一系列操作
        const insertSql1 = 'INSERT INTO users (username, age) VALUES (?, ?)';
        const insertValues1 = ['user1', 20];
        await connection.execute(insertSql1, insertValues1);

        const insertSql2 = 'INSERT INTO users (username, age) VALUES (?, ?)';
        const insertValues2 = ['user2', 25];
        await connection.execute(insertSql2, insertValues2);

        // 提交事务
        await connection.commit();
        console.log('事务提交成功');
    } catch (error) {
        // 回滚事务
        await connection.rollback();
        console.error('事务执行出错，已回滚:', error);
    } finally {
        // 释放连接
        connection.release();
    }
})();
```





### 用例

```js
// const mysql = require('mysql2')
const mysql = require("mysql2/promise");
// 创建连接池
const pool = mysql.createPool({
	host: "localhost", // 数据库主机地址
	user: "root", // 数据库用户名
	password: "123456", // 数据库密码
	database: "my_db", // 要使用的数据库名
});

/**
 * 查询数据
 */
async function searchTable() {
	try {
		const [rows, fields] = await pool.query("SELECT * FROM users");
		console.log(rows);
	} catch (error) {
		console.error("执行查询时出错:", error);
	} finally {
		// 关闭连接池
		await pool.end();
	}
}

/**
 * 新增单条数据
 */
async function insertData() {
	try {
		const sql = `INSERT INTO users (username,password) VALUES (?,?)`;
		const values = ["lisa", 635241];
		const [result] = await pool.execute(sql, values);
		console.log("插入成功，自增ID为", result.insertId);
	} catch (error) {
		console.log("插入数据时出错:", error);
	} finally {
		// 关闭连接池
		await pool.end();
	}
}

/**
 * 新增单条数据(方便写法)
 */
async function insertData2() {
	try {
		const sql = `INSERT INTO users set ?`;
		const values = {
			username: "xiohong",
			password: "963258",
		};
		const [result] = await pool.query(sql, values);
		console.log("插入成功，自增ID为", result.insertId);
	} catch (error) {
		console.log("插入数据时出错:", error);
	} finally {
		// 关闭连接池
		await pool.end();
	}
}

/**
 * 新增多条数据
 */
async function insertManyData() {
	try {
		const sql = `INSERT INTO users (username,password) VALUES ?`;
		const datas = [
			{ username: "mike", password: 635241 },
			{ username: "lucy", password: 635241 },
			{ username: "lily", password: 635241 },
		];
		const values = datas.map((item) => [item.username, item.password]);
		const [result] = await pool.query(sql, [values]);
		console.log("插入成功,受影响的记录数为:", result.affectedRows);
	} catch (error) {
		console.log("插入数据时出错:", error);
	} finally {
		// 关闭连接池
		await pool.end();
	}
}
// searchTable()
// insertData()
insertData2();
// insertManyData()
```
