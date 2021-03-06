# 方法和属性

#### 事件

* ['message' 事件](#event_message)
* ['exit' 事件](#event_exit)
* ['beforeExit' 事件](#event_beforeExit)
* ['rejectionHandled' 事件](#event_rejectionHandled)
* ['unhandledRejection' 事件](#event_unhandledRejection)
* ['uncaughtException' 事件](#event_uncaughtException)
  - [警告：请正确使用 'uncaughtException' 事件](#using_event_uncaughtException_correctly)

#### 属性

* [process.stdin](#stdin)
* [process.stdout](#stdout)
* [process.stderr](#stderr)
* [process.pid](#pid)
* [process.config](#config)
* [process.env](#env)
* [process.platform](#platform)
* [process.arch](#arch)
* [process.release](#release)
* [process.title](#title)
* [process.connected](#connected)
* [process.exitCode](#exitCode)
* [process.mainModule](#mainModule)
* [process.argv](#argv)
* [process.execPath](#execPath)
* [process.execArgv](#execArgv)
* [process.version](#version)
* [process.versions](#versions)

#### 方法

* [process.send(message[, sendHandle[, options]][, callback])](#send)
* [process.nextTick(callback[, arg][, ...])](#nextTick)
* [process.disconnect()](#disconnect)
* [process.exit([code])](#exit)
* [process.abort()](#abort)
* [process.kill(pid[, signal])](#kill)
* [process.cwd()](#cwd)
* [process.chdir(directory)](#chdir)
* [process.memoryUsage()](#memoryUsage)
* [process.umask([mask])](#umask)
* [process.uptime()](#uptime)
* [process.hrtime()](#hrtime)
* [process.setuid(id)](#setuid)
* [process.getuid()](#getuid)
* [process.setgid(id)](#setgid)
* [process.getgid()](#getgid)
* [process.seteuid(id)](#seteuid)
* [process.geteuid()](#geteuid)
* [process.setegid(id)](#setegid)
* [process.getegid()](#getegid)
* [process.setgroups(groups)](#setgroups)
* [process.getgroups()](#getgroups)
* [process.initgroups(user, extra_group)](#initgroups)

--------------------------------------------------


<div id="event_message" class="anchor"></div>
## 'message' 事件

- `message` {Object} 一个已解析的 JSON 对象或原始值

- `sendHandle` {Handle 对象} 一个 [net.Socket](../net/class_net_Socket.md#) 或 [net.Server](../net/class_net_Server.md#) 或 `undefined`。

通过 [ChildProcess.send()](../child_process/class_ChildProcess.md#) 发送的信息，需要在子进程对象中使用 `'message'` 事件获取。


<div id="event_exit" class="anchor"></div>
## 'exit' 事件

当进程即将退出时触发。在这一点上，没有办法防止事件循环的退出，一旦所有的 `'exit'` 监听器已经完成监听，运行的进程将会退出。因此，你**只能**在处理程序时执行**同步**操作。这有利于对模块的状态进行检查（如单元测试）。进程代码退出时的回调函数有一个参数。

该事件只有当 Node.js 明确的通关过 `process.exit()` 退出或事件循环隐式外泄时才被触发。

监听 `'exit'` 的示例：

```javascript
process.on('exit', (code) => {
    // do *NOT* do this
    setTimeout(() => {
        console.log('This will not run');
    }, 0);
    console.log('About to exit with code:', code);
});
```


<div id="event_beforeExit" class="anchor"></div>
## 'beforeExit' 事件

该事件在 Node.js 清空其事件循环并且没有安排其他事情的情况下触发。通常情况下，Node.js 在没有计划工作时退出，但 `'beforeExit'` 监听器会进行异步调用，从而导致 Node.js 继续运行。

`'beforeExit'` 在显式终止的情况下不会触发，比如 [process.exit()](#exit) 或未捕获的异常，并且不应该将其替代 `'exit'` 事件，除非你是为了安排更多的工作。


<div id="event_rejectionHandled" class="anchor"></div>
## 'rejectionHandled' 事件

每当 Promise 被拒绝时触发，并且在一个事件循环后会给它附加一个错误程序（比如 `.catch()`）。此事件触发时带有以下参数：

* `p` 代表在 `'unhandledRejection'` 事件触发前的那个 promise，但目前获得了一个拒绝程序。

对于 promise 链而言，目前没有一个总可以用于处理拒绝的顶级概念。作为一个在本质上是异步，一个 promise 拒绝将在之后得到处理 —— 可能远远超过了事件循环以后触发的 `'unhandledRejection'` 事件的花费。

说明这一点的另一种方式是，不像在同步代码中有一个不断增长的未处理的异常列表，promise 有着不断增长和收缩的未处理的拒绝列表。在同步代码中，`'uncaughtException'` 事件告诉你何时未处理的异常列表在不断增长。在异步代码中，`'unhandledRejection'` 事件告诉你何时未处理的拒绝列表在增长，`'rejectionHandled'` 事件告诉你何时未处理的拒绝列表在收缩。

例如使用拒绝检查钩子以便于在给给定时间内保持一份所有被拒绝的 promise 的理由的映射：

```javascript
const unhandledRejections = new Map();
process.on('unhandledRejection', (reason, p) => {
    unhandledRejections.set(p, reason);
});
process.on('rejectionHandled', (p) => {
    unhandledRejections.delete(p);
});
```

这份映射表会随时间的推移增长和收缩，反映了拒绝在何时未处理，何时被处理。你可以在一些错误日志中记录这些错误，无论是定期（尤其是针对长期运行的程序，在一个错误可能无限增长的的程序下，允许你清理该映射）还是在进程退出时（对脚本而言更方便）。


<div id="event_unhandledRejection" class="anchor"></div>
## 'unhandledRejection' 事件

每当 Promise 被拒绝时触发，没有错误处理程序附加到事件循环内的 promise 上。当程序设计中的 promise 异常被封装成拒绝 promise 时，这样的 promise 可以使用 [promise.catch(...)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch) 捕捉和处理，并且拒绝可以通过 promise 链传播。该事件有利于检测和保持对还没被处理的那些被拒绝的 promise 的跟踪。该事件触发时带有以下参数：

* `reason` 包含被拒绝的 promise 的对象（通常是一个[错误](../errors/class_Error.md#)实例）。

* `p` 被拒绝的 promise。

这是一个将每一个未处理的拒绝记录到控制台的例子：

```javascript
process.on('unhandledRejection', (reason, p) => {
    console.log("Unhandled Rejection at: Promise ", p, " reason: ", reason);
    // application specific logging, throwing an error, or other logic here
});
```

例如，这是一个将会触发 `'unhandledRejection'` 事件的拒绝：

```javascript
somePromise.then((res) => {
    return reportToUser(JSON.pasre(res)); // note the typo (`pasre`)
}); // no `.catch` or `.then`
```

这是一个也会触发 `'unhandledRejection'` 事件的编码模式的例子：

```javascript
function SomeResource() {
    // Initially set the loaded status to a rejected promise
    this.loaded = Promise.reject(new Error('Resource not yet loaded!'));
}

var resource = new SomeResource();
// no .catch or .then on resource.loaded for at least a turn
```

在这种情况下，你可能不希望将跟踪拒绝作为一个开发者的错误，就像你对其他 `'unhandledRejection'` 事件那样。为了解决这个问题，你可以在 `resource.loaded` 上附加一个假的 `.catch(() => { })` 处理器，防止触发 `'unhandledRejection'` 事件，或者你也可以使用 ['rejectionHandled'](#event_rejectionHandled) 事件。


<div id="event_uncaughtException" class="anchor"></div>
## 'uncaughtException' 事件

当异常一路冒泡回事件循环时会触发 `'uncaughtException'` 事件。默认，Node.js 通过在 stderr 中打印堆栈跟踪并退出来处理这种情况。可以通过给 `'uncaughtException'` 事件添加处理器来覆盖这种默认行为。

例如：

```javascript
process.on('uncaughtException', (err) => {
    console.log(`Caught exception: ${err}`);
});

setTimeout(() => {
    console.log('This will still run.');
}, 500);

// Intentionally cause an exception, but don't catch it.
nonexistentFunc();
console.log('This will not run.');
```


<div id="using_event_uncaughtException_correctly" class="anchor"></div>
#### 警告：请正确使用 'uncaughtException' 事件

请注意，`'uncaughtException'` 是一种异常处理的粗机制，希望只作为最后的手段。该事件*不应该*被用做 `On Error Resume Next` 的替代品。未处理的异常本质上意味着应用程序处于不确定状态。试图恢复应用程序代码而不是从异常恢复正常可能会导致额外的不可预见的和不可预知的问题。

从事件处理程序中抛出的异常不会被捕获。相反，会以非零退出代码退出进程，并且打印堆栈跟踪。这是为了避免无限递归。

尝试从一个未捕获到异常后恢复正常类似于在升级电脑时拔出电源线——十次里的前九次时间是没有任何反应的——但在第十次的时候系统损坏了。

`'uncaughtException'` 的正确用法是在进程关闭前执行分配资源的同步清理（如，文件描述符、处理程序等）。在 `'uncaughtException'` 事件后恢复正常操作是不安全的。


<div id="stdin" class="anchor"></div>
## process.stdin

一个指向标准输入（`stdin` on fd `0`）的`可读（Readable）流`。

打开标准输入并监听两个事件的示例：

```javascript
process.stdin.setEncoding('utf8');

process.stdin.on('readable', () => {
    var chunk = process.stdin.read();
    if (chunk !== null) {
        process.stdout.write(`data: ${chunk}`);
    }
});

process.stdin.on('end', () => {
    process.stdout.write('end');
});
```

作为流（Stream），`process.stdin` 也可以使用兼容 Node.js v0.10 之前版本编写的脚本的“老”模式。更多信息详见[流的兼容性](../stream/under_the_hood.md#compatibility_with_older_Nodejs_versions)。

在“老”的流模式中，标准输入流默认是暂停状态，所以读取时必须调用 `process.stdin.resume()`。请注意，调用 `process.stdin.resume()` 时也会将流自身转换至“老”模式。

如果你开始一个新项目，你应该更喜欢最“新”的流模式而非“老”模式。


<div id="stdout" class="anchor"></div>
## process.stdout

一个指向标准输出（`stdout` on fd `1`）的`可写（Writable）流`。

例如，`console.log` 的等效写法如下：

```javascript
console.log = (msg) => {
    process.stdout.write(`${msg}\n`);
};
```

`process.stderr` 和 `process.stdout` 不像 Node.js 中其他的流，它们无法被关闭（`end()` 将会抛出），它们从不触发 `'finish'` 事件，并且当输出重定向到一个文件时可以阻止其写入（虽然磁盘很快并且操作系统通常采用回写式缓存，但它应该确实是一个非常罕见的情况。）。

要检查 Node.js 是否运行在一个 TTY 上下文中，可以使用 `process.stderr`、`process.stdout` 或 `process.stdin` 中 `isTTY` 属性：

```
$ node -p "Boolean(process.stdin.isTTY)"
true
$ echo "foo" | node -p "Boolean(process.stdin.isTTY)"
false

$ node -p "Boolean(process.stdout.isTTY)"
true
$ node -p "Boolean(process.stdout.isTTY)" | cat
false
```

更多信息详见[tty文档](../tty/tty.md#)。


<div id="stderr" class="anchor"></div>
## process.stderr

一个指向标准错误（`stderr` on fd `2`）的`可写（Writable）流`。

`process.stderr` 和 `process.stdout` 不像 Node.js 中其他的流，它们无法被关闭（`end()` 将会抛出），它们从不触发 `'finish'` 事件，并且当输出重定向到一个文件时可以阻止其写入（虽然磁盘很快并且操作系统通常采用回写式缓存，但它应该确实是一个非常罕见的情况。）。


<div id="pid" class="anchor"></div>
## process.pid

进程的 PID ：

```
console.log(`This process is pid ${process.pid}`);
```


<div id="config" class="anchor"></div>
## process.config

返回包含了用来编译当前 Node.js 可执行程序的配置选项的 JSON 对象。内容与运行 `./configure` 脚本生成的 `config.gypi` 文件相同。

可能的输出示例如下：

```
{
    target_defaults: {
        cflags: [],
        default_configuration: 'Release',
        defines: [],
        include_dirs: [],
        libraries: []
    },
    variables: {
        host_arch: 'x64',
        node_install_npm: 'true',
        node_prefix: '',
        node_shared_cares: 'false',
        node_shared_http_parser: 'false',
        node_shared_libuv: 'false',
        node_shared_zlib: 'false',
        node_use_dtrace: 'false',
        node_use_openssl: 'true',
        node_shared_openssl: 'false',
        strict_aliasing: 'true',
        target_arch: 'x64',
        v8_use_snapshot: 'true'
    }
}
```


<div id="env" class="anchor"></div>
## process.env

返回包含用户环境中的对象。详见[environ(7)](http://man7.org/linux/man-pages/man7/environ.7.html)。

此对象的示例如下：

```
{
    TERM: 'xterm-256color',
    SHELL: '/usr/local/bin/bash',
    USER: 'maciej',
    PATH: '~/.bin/:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin',
    PWD: '/Users/maciej',
    EDITOR: 'vim',
    SHLVL: '1',
    HOME: '/Users/maciej',
    LOGNAME: 'maciej',
    _: '/usr/local/bin/node'
}
```

你可以写入该对象，但修改不会在你的程序之外得到反映。这意味着以下代码不会起作用：

```
$ node -e 'process.env.foo = "bar"' && echo $foo
```

但这个是可以的：

```
process.env.foo = 'bar';
console.log(process.env.foo);
```

在 `process.env` 上分配的属性会隐式的将值转换为字符串。

示例：

```
process.env.test = null;
console.log(process.env.test);
// => 'null'
process.env.test = undefined;
console.log(process.env.test);
// => 'undefined'
```

可以使用 `delete` 删除 `process.env` 上的属性。

示例：

```
process.env.TEST = 1;
delete process.env.TEST;
console.log(process.env.TEST);
// => undefined
```


<div id="platform" class="anchor"></div>
## process.platform

你正运行在什么平台上：`'darwin'`, `'freebsd'`, `'linux'`, `'sunos'` 或 `'win32'`。

```
console.log(`This platform is ${process.platform}`);
```


<div id="arch" class="anchor"></div>
## process.arch

你正运行在什么处理器架构上：`'arm'`, `'ia32'` 或 `'x64'`。

```
console.log(`This processor architecture is ${process.arch}`);
```


<div id="release" class="anchor"></div>
## process.release

一个包含当前版本元数据的对象，包括源码包网址和压缩包头。

`process.release` 包含以下属性：

* `name`：带值的字符串。Node.js 中永远是 `'node'`；io.js 中永远是 `'io.js'`。

* `sourceUrl`：一个指向当前版本的 *.tar.gz* 文件的完整 URL。

* `headersUrl`：一个指向当前版本的 *.tar.gz* 头文件的完整 URL。该文件比完整源文件明显要小，并且可用于编译针对 Node.js 的插件。

* `libUrl`：一个指向匹配当前版本的结构和版本号的 *node.lib* 文件。该文件可用于编译针对 Node.js 的插件。*此属性仅出现在 Windows 版的 Node.js 中，并且不会出现在所有其它平台上*。

例如：

```
{
    name: 'node',
    sourceUrl: 'https://nodejs.org/download/release/v4.0.0/node-v4.0.0.tar.gz',
    headersUrl: 'https://nodejs.org/download/release/v4.0.0/node-v4.0.0-headers.tar.gz',
    libUrl: 'https://nodejs.org/download/release/v4.0.0/win-x64/node.lib'
}
```

在从源代码树自定义构建的非发布版本中，只可能存在 `name` 属性，附加属性不应该存在。


<div id="title" class="anchor"></div>
## process.title

获取/设置 (Getter/setter) `ps` 中显示的进程名。

当设置该属性时，所能设置的字符串最大长度视具体平台而定，如果超过的话会自动截断。

在 Linux 和 OS X 上，它受限于名称的字节长度加上命令行参数的长度，因为它覆写了参数内存（argv memory）。

v0.8 版本允许更长的进程标题字符串，也支持覆盖环境内存，但在一些案例中这是潜在的不安全/混淆（很难说清楚）。


<div id="connected" class="anchor"></div>
## process.connected

- {Boolean} 在调用 [process.disconnect()](disconnect) 后设为 false 。

如果 `process.connected` 为 false，它不再能够发送信息。


<div id="exitCode" class="anchor"></div>
## exitCode

当进程正常退出或通过不带指定代码的 [process.exit()](#exit) 退出时，那个代表进程退出代码的数字。

指定 `process.exit(code)` 的代码，将会覆盖任何先前设定的 `process.exitCode`。


<div id="mainModule" class="anchor"></div>
## process.mainModule

检索 [require.main](../modules/module.md) 的另一种方法。不同的是，如果主模块在运行时改变，`require.main` 仍然会指向发生变化之前请求的模块的原始主模块。通常可以安全地假设两个都是指向相同的模块。

针对 `require.main` 而言，如果没有入口脚本，它可能会变成 `undefined`。


<div id="argv" class="anchor"></div>
## process.argv

一个包含命令行参数的数组。第一个元素会是 'node'，第二个元素会是该 JavaScript 文件的名称。接下来的元素会是任何额外的命令行参数。

```javascript
// print process.argv
process.argv.forEach((val, index, array) => {
    console.log(`${index}: ${val}`);
});
```

这将产生：

```
$ node process-2.js one two=three four
0: node
1: /Users/mjr/work/node/process-2.js
2: one
3: two=three
4: four
```


<div id="execPath" class="anchor"></div>
## process.execPath

这是启动进程可执行文件的绝对路径。

示例：

```
/usr/local/bin/node
```


<div id="execArgv" class="anchor"></div>
## process.execArgv

这是启动该进程的指定的 Node.js 可执行文件的命令行参数的设置。这些选项不在 `process.argv` 中显示，并且不包括 Node.js 的可执行文件，脚本名称或任何跟在脚本名称后的参数。这些参数在为了使衍生子进程和父进程有相同的执行环境时非常有用。

示例：

```
$ node --harmony script.js --version
```

`process.execArgv` 的结果是：

```
['--harmony']
```

`process.argv` 的结果是：

```
['/usr/local/bin/node', 'script.js', '--version']
```


<div id="version" class="anchor"></div>
## process.version

一个暴露编译时的 `NODE_VERSION` 的属性。

```javascript
console.log(`Version: ${process.version}`);
```


<div id="versions" class="anchor"></div>
## process.versions

一个暴露 Node.js 及其依赖的版本字符串的属性。

```javascript
console.log(process.versions);
```

将打印类似于：

```
{
    http_parser: '2.3.0',
    node: '1.1.1',
    v8: '4.1.0.14',
    uv: '1.3.0',
    zlib: '1.2.8',
    ares: '1.10.0-DEV',
    modules: '43',
    icu: '55.1',
    openssl: '1.0.1k'
}
```


<div id="send" class="anchor"></div>
## process.send(message[, sendHandle[, options]][, callback])

- `message` {Object}

- `sendHandle` {Handle object}

- `options` {Object}

- `callback` {Function}

- 返回：{Boolean}

当 Node.js 衍生出一个附加的 IPC 通道时，它可以使用 `process.send()` 向父进程发送信息。每个父进程上的 `ChildProcess` 对象都会收到 ['message'](../child_process/class_ChildProcess.md#event_message) 事件。

注意：*该函数使用 [JSON.stringify()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) 内部序列化 `message`*。

如果 Node.js 没有衍生出一个 IPC 通道，那么 `process.send()` 会是未定义的。


<div id="nextTick" class="anchor"></div>
## process.nextTick(callback[, arg][, ...])

- `callback` {Function}

一旦当前的事件循环完成了一圈的运行，就会调用该回调函数。

这*不是* [setTimeout(fn, 0)](../timers/scheduling_timers.md#settimeoutcallback-delay-args) 的一个简单的别名，因为它的效率高多了。在事件循环的后续时间刻度（subsequent ticks）中，它运行在任何附加的 I/O 事件（包括定时器）触发之前。

```javascript
console.log('start');
process.nextTick(() => {
    console.log('nextTick callback');
});
console.log('scheduled');
// Output:
// start
// scheduled
// nextTick callback
```

如果你想要在对象创建后，但在任何 I/O 操作发生前执行某些操作，那么这个函数对你而言就十分重要了。

```javascript
function MyThing(options) {
    this.setupOptions(options);

    process.nextTick(() => {
        this.startDoingStuff();
    });
}

var thing = new MyThing();
thing.getReadyForStuff();

// thing.startDoingStuff() gets called now, not before.
```

在使用该函数时，请保证你的函数一定是同步或者一定是异步执行的。请思考这个例子：

```javascript
// WARNING!  DO NOT USE!  BAD UNSAFE HAZARD!
function maybeSync(arg, cb) {
    if (arg) {
        cb();
        return;
    }

    fs.stat('file', cb);
}
```

这样执行是很危险。如果你还不清楚上述行为的危害请看下面的例子：

```javascript
maybeSync(true, () => {
    foo();
});
bar();
```

那么，使用刚才那个不知道是同步还是异步的操作，在执行时你就会发现，你无法确定到底是 `foo()` 先执行，还是 `bar()` 先执行。

以下是更好的解决方案：

```javascript
function definitelyAsync(arg, cb) {
    if (arg) {
        process.nextTick(cb);
        return;
    }

    fs.stat('file', cb);
}
```

注意：nextTick 的队列会在完全执行完毕之后才会去完成其他的 I/O 操作。因此，递归设置 nextTick 的回调就像一个 `while(true);` 循环一样，将会阻止任何 I/O 操作的发生。


<div id="disconnect" class="anchor"></div>
## process.disconnect()

关闭父进程的 IPC 通道，一旦没有进程与其保持连接状态，就会允许子进程正常退出。

等同于父进程的 [ChildProcess.disconnect()](../child_process/class_ChildProcess.md#disconnect)。

如果 Node.js 没有衍生出一个 IPC 通道，那么 `process.disconnect()` 会是未定义的。


<div id="exit" class="anchor"></div>
## process.exit([code])

通过指定的 `code` 结束进程。如果省略，“成功”退出时将使用代码 `0`。

以失败状态（'failure' code）退出：

```javascript
process.exit(1);
```

在执行 Node.js 的 shell 中就可以看到退出代码为 1。


<div id="abort" class="anchor"></div>
## process.abort()

这将导致 Node.js 触发一个 'abort' 事件，这会导致 Node.js 退出并且生成一个核心文件。


<div id="kill" class="anchor"></div>
## process.kill(pid[, signal])

向进程发送一个信号。`pid` 是进程的 id 而 `signal` 则是描述信号的字符串名称。信号名称都类似于 `SIGINT` 或 `SIGHUP`。如果省略 signal 参数，则默认为 `SIGTERM`。详见[信号事件](./signal_events.md#)和[kill(2)](http://man7.org/linux/man-pages/man2/kill.2.html)。

如果目标不存在将抛出错误。作为一个特殊情况，`0` 信号可以用于测试信号是否存在。如果在 Windows 平台上的 `pid` 被用于杀死一个进程组，那么它将抛出一个错误。

注意，尽管函数名叫 `process.kill`，但它实际上只是一个信号发送器，就像 `kill` 系统调用。信号的发送可能会做除了杀死目标进程以外的一些其他事情。

将信号发送到自己的例子：

```javascript
process.on('SIGHUP', () => {
    console.log('Got SIGHUP signal.');
});

setTimeout(() => {
    console.log('Exiting.');
    process.exit(0);
}, 100);

process.kill(process.pid, 'SIGHUP');
```

注意：当 SIGUSR1 是由 Node.js 接收时，它将开启调试器，见[信号事件](./signal_events.md#)。


<div id="cwd" class="anchor"></div>
## process.cwd()

返回进程的当前工作目录。

```javascript
console.log(`Current directory: ${process.cwd()}`);
```


<div id="chdir" class="anchor"></div>
## process.chdir(directory)

改变进程的当前工作目录，如果操作失败，则抛出一个异常。

```javascript
console.log(`Starting directory: ${process.cwd()}`);
try {
    process.chdir('/tmp');
    console.log(`New directory: ${process.cwd()}`);
} catch (err) {
    console.log(`chdir: ${err}`);
}
```


<div id="uptime" class="anchor"></div>
## process.uptime()

返回 Node.js 运行的秒数。


<div id="hrtime" class="anchor"></div>
## process.hrtime()

以 `[seconds, nanoseconds]` 元组数组的形式返回当前的高精度真实时间。它相对于过去的任意时间。该值与日期无关，因此不受时钟漂移的影响。主要用途是可以通过精确的时间间隔，来衡量程序的性能。

你可以通过之前调用的 `process.hrtime()` 的结果和当前的 `process.hrtime()` 来获取一个差异读数，这对于基准和衡量时间间隔非常有用：

```javascript
var time = process.hrtime();
// [ 1800216, 25 ]

setTimeout(() => {
    var diff = process.hrtime(time);
    // [ 1, 552 ]

    console.log('benchmark took %d nanoseconds', diff[0] * 1e9 + diff[1]);
    // benchmark took 1000000527 nanoseconds
}, 1000);
```


<div id="memoryUsage" class="anchor"></div>
## process.memoryUsage()

返回一个描述 Node.js 进程的内存使用量的对象，以字节（bytes）为单位。

```javascript
const util = require('util');

console.log(util.inspect(process.memoryUsage()));
```

这会产生：

```javascript
{
    rss: 4935680,
    heapTotal: 1826816,
    heapUsed: 650472
}
```

`heapTotal` 和 `heapUsed` 引用 V8 引擎的内存使用量。


<div id="umask" class="anchor"></div>
## process.umask([mask])

设置或读取进程的文件模式的创建掩码。子进程从父进程中继承掩码。如果设定了 `mask` 参数，则返回旧的掩码，否则返回当前的掩码。

```javascript
const newmask = 0o022;
const oldmask = process.umask(newmask);
console.log(
    `Changed umask from ${oldmask.toString(8)} to ${newmask.toString(8)}`
);
```


<div id="setgid" class="anchor"></div>
## process.setuid(id)

注意：该函数仅在 POSIX 平台（如，非 Windows 和 Android）上有效。

设置进程用户的标识（见 [setuid(2)](http://man7.org/linux/man-pages/man2/setuid.2.html)）。它可以接收数字 ID 或一个用户名字符串。如果指定了用户名，该方法会阻塞运行直到将它解析为一个数字 ID 为止。

```javascript
if (process.getuid && process.setuid) {
    console.log(`Current uid: ${process.getuid()}`);
    try {
        process.setuid(501);
        console.log(`New uid: ${process.getuid()}`);
    } catch (err) {
        console.log(`Failed to set uid: ${err}`);
    }
}
```

<div id="getuid" class="anchor"></div>
## process.getuid()

注意：该函数仅在 POSIX 平台（如，非 Windows 和 Android）上有效。

获取进程用户标识（见 [getuid(2)](http://man7.org/linux/man-pages/man2/getuid.2.html)）。这是数字的用户 id，而非用户名。


<div id="setgid" class="anchor"></div>
## process.setgid(id)

注意：该函数仅在 POSIX 平台（如，非 Windows 和 Android）上有效。

设置进程组标识（见 [setgid(2)](http://man7.org/linux/man-pages/man2/setgid.2.html)）。它可以接收数字 ID 或一个组名称的字符串。如果指定了组名称，该方法会阻塞运行直到将它解析为一个数字 ID 为止。

```javascript
if (process.getgid && process.setgid) {
    console.log(`Current gid: ${process.getgid()}`);
    try {
        process.setgid(501);
        console.log(`New gid: ${process.getgid()}`);
    } catch (err) {
        console.log(`Failed to set gid: ${err}`);
    }
}
```


<div id="getgid" class="anchor"></div>
## process.getgid()

注意：该函数仅在 POSIX 平台（如，非 Windows 和 Android）上有效。

获取进程组标识（见 [getgid(2)](http://man7.org/linux/man-pages/man2/getgid.2.html)）。这是数字的组 id，而非组名称。

```javascript
if (process.getgid) {
    console.log(`Current gid: ${process.getgid()}`);
}
```


<div id="seteuid" class="anchor"></div>
## process.seteuid(id)

注意：该函数仅在 POSIX 平台（如，非 Windows 和 Android）上有效。

设置有效的进程用户的标识（见 [seteuid(2)](http://man7.org/linux/man-pages/man2/seteuid.2.html)）。它可以接收数字 ID 或一个用户名字符串。如果指定了用户名，该方法会阻塞运行直到将它解析为一个数字 ID 为止。

```javascript
if (process.geteuid && process.seteuid) {
    console.log(`Current uid: ${process.geteuid()}`);
    try {
        process.seteuid(501);
        console.log(`New uid: ${process.geteuid()}`);
    } catch (err) {
        console.log(`Failed to set uid: ${err}`);
    }
}
```


<div id="geteuid" class="anchor"></div>
## process.geteuid()

注意：该函数仅在 POSIX 平台（如，非 Windows 和 Android）上有效。

获取有效的进程用户标识（见 [geteuid(2)](http://man7.org/linux/man-pages/man2/geteuid.2.html)）。这是数字的用户 id，而非用户名。

```javascript
if (process.geteuid) {
    console.log(`Current uid: ${process.geteuid()}`);
}
```


<div id="setegid" class="anchor"></div>
## process.setegid(id)

注意：该函数仅在 POSIX 平台（如，非 Windows 和 Android）上有效。

设置有效的进程组标识（见 [setegid(2)](http://man7.org/linux/man-pages/man2/setegid.2.html)）。它可以接收数字 ID 或一个组名称的字符串。如果指定了组名称，该方法会阻塞运行直到将它解析为一个数字 ID 为止。

```javascript
if (process.getegid && process.setegid) {
    console.log(`Current gid: ${process.getegid()}`);
    try {
        process.setegid(501);
        console.log(`New gid: ${process.getegid()}`);
    } catch (err) {
        console.log(`Failed to set gid: ${err}`);
    }
}
```


<div id="getegid" class="anchor"></div>
## process.getegid()

注意：该函数仅在 POSIX 平台（如，非 Windows 和 Android）上有效。

获取有效的进程组标识（见 [getegid(2)](http://man7.org/linux/man-pages/man2/getegid.2.html)）。这是数字的组 id，而非组名称。

```javascript
if (process.getegid) {
    console.log(`Current gid: ${process.getegid()}`);
}
```


<div id="setgroups" class="anchor"></div>
## process.setgroups(groups)

注意：该函数仅在 POSIX 平台（如，非 Windows 和 Android）上有效。

设置补充群组的 ID。这是一个特权操作，意味着你需要使用 root 账户或拥有 `CAP_SETGID` 能力。

该列表参数可以包含组 ID、组名称或者混在一起。


<div id="getgroups" class="anchor"></div>
## process.getgroups()

注意：该函数仅在 POSIX 平台（如，非 Windows 和 Android）上有效。

返回补充群组的 ID 数组。如果有效的组 ID 在 POSIX 上未指定，但在 Node.js 中会确保它永远都是包含在内的。（POSIX leaves it unspecified if the effective group ID is included but Node.js ensures it always is. [翻译君](https://github.com/Amery2010)表示已阵亡...）


<div id="initgroups" class="anchor"></div>
## process.initgroups(user, extra_group)

注意：该函数仅在 POSIX 平台（如，非 Windows 和 Android）上有效。

读取 /etc/group 并通过该用户所在的所有分组初始化组（group）访问列表。这是一个特权操作，意味着你需要使用 root 账户或拥有 `CAP_SETGID` 能力。

`user` 是一个用户名或用户 ID。`extra_group` 是一个群组名称或群组 ID。

当你在注销权限时有时需要注意。例如：

```javascript
console.log(process.getgroups());         // [ 0 ]
process.initgroups('bnoordhuis', 1000);   // switch user
console.log(process.getgroups());         // [ 27, 30, 46, 1000, 0 ]
process.setgid(1000);                     // drop root gid
console.log(process.getgroups());         // [ 27, 30, 46, 1000 ]
```