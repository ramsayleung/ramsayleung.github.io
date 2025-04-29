+++
title = "你所不可或缺的 – logging"
description = "an introduction about logging"
date = 2017-04-09T00:00:00+08:00
lastmod = 2022-02-23T21:48:13+08:00
tags = ["python", "log"]
categories = ["python"]
draft = false
toc = true
showQuote = true
+++

## <span class="section-num">1</span> 重要性 {#重要性}

笔者最近都在负责项目中关于日志的部分，因为跟日志打交道比较多，所以有一些关于日 志感受和技巧想要分享一下。

笔者认为对于各种程序和应用，日志都是非常重要的，因为程序在部属到服务器之后，开发者是没办法像在本地开发那样可以充分了解程序发生的状况，而使用日志可以让开发者了解运行中的程序的状态，即使出现了错误，或者是系统挂了，也可以从日志中分析原因。

所以换句话说，日志的重要程度甚至可以称得上是不可或缺。接下来，笔者将会以 Python 中的 _logging_ 模块为例阐述日志。


## <span class="section-num">2</span> 关于日志 {#关于日志}


### <span class="section-num">2.1</span> 使用 print 函数输出？ {#使用-print-函数输出}

日志是为了输出程序的运行状态，那么可否使用 `print` 函数进行 logging 的工作呢？

我并不建议把 `print()` 函数当作日志使用 (当然，如果你一定要这么用，我也拦不住)；不建议使用 **print** 进行logging 原因有：

1.  无法在不修改源代码的情况下，控制日志的输出
2.  日志信息可能跟程序输出的有用数据混杂，导致输出的数据不可读或者非常难读
3.  print 无法将日志信息输出到除标准输出以外的目标 (例如文件，socket,SMTP 服务器等)
4.  无法根据错误信息的等级进行动态输出，因为 **print** 函数的作用只是输出信息

可能对于非常简单的小程序，开发者可以使用 print 进行日志输出，但是对于比较大型的程序，系统内置的 logging 类库或许是更好的选择


### <span class="section-num">2.2</span> 日志需要记录的是什么 {#日志需要记录的是什么}

Python 的日志类库 `logging` 可以让开发者根据不同场景使用不同的日志等级以输出 不同的日志信息。

而日志需要记录的最基本的信息又是什么呢？要想回答这个问题，先和我一起回顾一下日志的功能：记录程序的状态，为程序的开发和调试提供便利！

所谓方便调试，需要记录的必然包括可以帮助更快定位到错误的有用信息：

1.  Logger 的名字 (比较常用的做法都是 **__name\_\_**,即当前文件的信息)
2.  具体日期 (这个可以帮助确定出错的具体场景)
3.  方法名
4.  源代码行数
5.  异常的 traceback 信息

这只是最基本的信息，具体还要根据场景添加其它有用信息；比如对于分布式的程序，肯定还要记录其它节点的名字，IP 等有用信息。


## <span class="section-num">3</span> Logging 的正确姿势 {#logging-的正确姿势}


### <span class="section-num">3.1</span> 使用 Python 的 logging 模块 {#使用-python-的-logging-模块}

我认为，使用 Python 的标准日志库是比较好的实践，因为标准库已经提供了开箱即用的特性，无需重复造轮子。Python 的 logging 模块也很容易上手，举个小例子：

```python
import logging
logging.basicConfig(level=logging.DEBUG)
# define a logger
logger = logging.getLogger(__name__)

#Info level msg
logger.info('Info level message')
#Debug level msg
logger.debug('Debug level message')
#Warning level msg
logger.info('Warning level message')
```

日志输出如下：

```text
INFO:__main__: Info level message
DEBUG:__main__: Debug level message
WARN:__main__: Warning level message
```


### <span class="section-num">3.2</span> 记录异常信息 {#记录异常信息}

日志一个非常重要的作用就是调试，所以记录出现异常的地方是有必要，并且需要记录栈的调用信息。例如：

```python
try:
    open('file_not_exist.txt', 'wt')
except Exception, e:
    logger.error('Failed to write a file',exc_info=True)
```

通过将 `exc_info` 设置成 True, 栈的调用信息就会记录到日志里面。而也可以使用 `logger.exception(message,*args)` 方法，它等同于 `logger.error(msg,exc_info=True,*args)` 方法。


### <span class="section-num">3.3</span> 使用日志文件轮转控制器 (rotating file handler) {#使用日志文件轮转控制器--rotating-file-handler}

如果使用日志文件控制器 (FileHandler), 不断地运行程序，就会产生越来越多的日志 信息或者是日志文件。

为了控制日志文件的数量，可以使用 `RotatingFileHandler` 自 动新建新的日志文件，并且保留旧的日志文件，当产生一定数量的日志文件之后，就会 自动删除掉最旧的日志文件。例如：

```python
handler = logging.handlers.RotatingFileHandler(
    LOG_FILENAME,
    maxBytes=20,
    backupCount=5,
)
my_logger.addHandler(handler)
```

就是日志文件大小超过20个字节 (当然，真实情况不会那么小的阀值)，就创建一个新的日志文件，把原来的日志文件，例如叫 _example.log_ 重命名为 _example.log.1_,然后新建的日志文件就会被命名为_example.log_, 一直到产生了6个日志文件，即 _example.log.5_, 继续记录日志，最开始的第一个日志就会被删除。


### <span class="section-num">3.4</span> 使用日志服务器 {#使用日志服务器}

对于那些分布式的应用，或者部署多台服务器上有不同日志的程序而言，逐个服务器或者节点查看日志实在太可怕了. 这个时候，就可以设置一个日志服务器，把重要的日志信息发送到日志服务器，你就在日志服务器上监控各个节点的日志状态了。

[logging-cookbook](https://docs.python.org/3/howto/logging-cookbook.html) 的例子：

客户端或者节点：

```python
import logging
import logging.handlers

rootLogger = logging.getLogger('')
rootLogger.setLevel(logging.DEBUG)
socketHandler = logging.handlers.SocketHandler('localhost',
						    logging.handlers.DEFAULT_TCP_LOGGING_PORT)
# don't bother with a formatter, since a socket handler sends the event as
# an unformatted pickle
rootLogger.addHandler(socketHandler)

# Now, we can log to the root logger, or any other logger. First the root...
logging.info('Jackdaws love my big sphinx of quartz.')

# Now, define a couple of other loggers which might represent areas in your
# application:

logger1 = logging.getLogger('myapp.area1')
logger2 = logging.getLogger('myapp.area2')

logger1.debug('Quick zephyrs blow, vexing daft Jim.')
logger1.info('How quickly daft jumping zebras vex.')
logger2.warning('Jail zesty vixen who grabbed pay from quack.')
logger2.error('The five boxing wizards jump quickly.')
```

日志服务器：

```python
import logging
import logging.handlers
import pickle
import socketserver
import struct


class LogRecordStreamHandler(socketserver.StreamRequestHandler):
    """Handler for a streaming logging request.

    This basically logs the record using whatever logging policy is
    configured locally.
    """

    def handle(self):
	"""
	Handle multiple requests - each expected to be a 4-byte length,
	followed by the LogRecord in pickle format. Logs the record
	according to whatever policy is configured locally.
	"""
	while True:
	    chunk = self.connection.recv(4)
	    if len(chunk) < 4:
		break
	    slen = struct.unpack('>L', chunk)[0]
	    chunk = self.connection.recv(slen)
	    while len(chunk) < slen:
		chunk = chunk + self.connection.recv(slen - len(chunk))
		obj = self.unPickle(chunk)
		record = logging.makeLogRecord(obj)
		self.handleLogRecord(record)

    def unPickle(self, data):
	return pickle.loads(data)

    def handleLogRecord(self, record):
	# if a name is specified, we use the named logger rather than the one
	# implied by the record.
	if self.server.logname is not None:
	    name = self.server.logname
	else:
	    name = record.name
	    logger = logging.getLogger(name)
	    # N.B. EVERY record gets logged. This is because Logger.handle
	    # is normally called AFTER logger-level filtering. If you want
	    # to do filtering, do it at the client end to save wasting
	    # cycles and network bandwidth!
	logger.handle(record)


class LogRecordSocketReceiver(socketserver.ThreadingTCPServer):
    """
    Simple TCP socket-based logging receiver suitable for testing.
    """

    allow_reuse_address = True

    def __init__(self, host='localhost',
		 port=logging.handlers.DEFAULT_TCP_LOGGING_PORT,
		 handler=LogRecordStreamHandler):
	socketserver.ThreadingTCPServer.__init__(self, (host, port), handler)
	self.abort = 0
	self.timeout = 1
	self.logname = None

    def serve_until_stopped(self):
	import select
	abort = 0
	while not abort:
	    rd, wr, ex = select.select([self.socket.fileno()],
					    [], [],
					    self.timeout)
	    if rd:
		self.handle_request()
		abort = self.abort


def main():
    logging.basicConfig(
	format='%(relativeCreated)5d %(name)-15s %(levelname)-8s %(message)s')
    tcpserver = LogRecordSocketReceiver()
    print('About to start TCP server...')
    tcpserver.serve_until_stopped()


if __name__ == '__main__':
    main()
```

通过给 logger 添加一个SocketHandler 就可以把日志事件发送到服务器端


### <span class="section-num">3.5</span> 使用配置文件 {#使用配置文件}

虽然开发者可以使用 Python 代码来配置日志系统，但是这样是很不灵活的，每次修改日志等级还需要去改动代码。

而使用配置文件无疑是一个更好的选择，例如 json 或者是 yaml 文件，这样就可以在 json/yaml 文件中加载日志配置了。以 [Django](https://docs.djangoproject.com/en/1.9/topics/logging/#configuring-logging) 项目的配置文件为例，我改成了 json 格式：

```json
{
    "version": 1,
    "disable_existing_loggers": True,
    "formatters": {
	"verbose": {
	    "format": "%(levelname)s %(asctime)s %(module)s %(process)d %(thread)d %(message)s"
	},
	"simple": {
	    "format": "%(levelname)s %(message)s"
	},
    },
    "filters": {
	"special": {
	    "()": "project.logging.SpecialFilter",
	    "foo": "bar",
	}
    },
    "handlers": {
	"null": {
	    "level": "DEBUG",
	    "class": "django.utils.log.NullHandler",
	},
	"console": {
	    "level": "DEBUG",
	    "class": "logging.StreamHandler",
	    "formatter": "simple"
	},
	"mail_admins": {
	    "level": "ERROR",
	    "class": "django.utils.log.AdminEmailHandler",
	    "filters": "special"
	}
    },
    "loggers": {
	"django": {
	    "handlers": "null",
	    "propagate": true,
	    "level": "INFO",
	},
	"django.request": {
	    "handlers": ["mail_admins"],
	    "level": "ERROR",
	    "propagate": false,
	},
	"myproject.custom": {
	    "handlers": ["console", "mail_admins"],
	    "level": "INFO",
	    "filters": ["special"]
	}
    }
}
```

以及加载 json 文件到日志配置中：

```python
import json
import logging.config


def setup_logging():
    """
    Setup logging configuration
    """
    with open('logging_configuration.json', 'rt') as f:
	config = json.load(f)
	logging.config.dictConfig(config)
```

使用 json 还有一个好处是标准库已经内置了 json 模块，无需像 yaml 那样需要安装额外的模块，不过我更推崇 yaml, 因为清晰之余，还可以少打很多字 :)


### <span class="section-num">3.6</span> 对于不同的代码，使用不同的日志等级 {#对于不同的代码-使用不同的日志等级}

因为一个项目不同代码要求不一样，也无需把每一个实现细节都记录在日志，只需要根 据不同的实现，使用不同的日志等级，例如使用 `Debug` 记录系统启动，处理业务逻辑 请求的信息，使用 `Error`, 记录系统的出错信息，可以结合堆栈分析原因，等等。

此外，Logger 实例可以被配置成基于名字的树状结构。 每一个部件都定义了一个基础的名字，对应的模块被设置成子节点。而 root logger 没有名字。如图：

{{< figure src="/ox-hugo/example_logger_tree.png" link="/ox-hugo/example_logger_tree.png" >}}

就配置 `logging` 而言，我认为树状结构是非常有用的，因为无需为每一个 logger 都设置handler. 如果一个 logger 没有 handler 的话，它就会让父节点来处理。所以 对于对于大部份的应用而言，只需配置 root logger, 而所有的信息都会发送到同一个 地方

{{< figure src="/ox-hugo/one_logger_handler.png" link="/ox-hugo/one_logger_handler.png" >}}

而树状结构可以对应用的不同部分使用不同的日志等级，不同的 handler, 不同的formatter, 以更好地控制日志信息


### <span class="section-num">3.7</span> 使用结构化日志 {#使用结构化日志}

虽然大部份的日志信息对于人类都是可读的，但是对于程序而言，就很难进行解析了。

这个时候，为了方便程序进行解析，我建议使用结构化格式的日志，这样就不再需要各种复杂的正则表达式来解析日志了。得益于内置的 json 模块，使用 json 就可以很简单地生成的利于程序解析结构化日志，以 [logging cookbook](https://docs.python.org/3/howto/logging-cookbook.html) 中的例子说明：

```python
import json
import logging


class StructuredMessage(object):
    def __init__(self, message, **kwargs):
	self.message = message
	self.kwargs = kwargs

    def __str__(self):
	return '%s >>> %s' % (self.message, json.dumps(self.kwargs))


_ = StructuredMessage   # optional, to improve readability

logging.basicConfig(level=logging.INFO, format='%(message)s')
logging.info(_('message 1', foo='bar', bar='baz', num=123, fnum=123.456))
```

日志输出结果如下：

```json
message 1 >>> {"fnum": 123.456, "num": 123, "bar": "baz", "foo": "bar"}
```


### <span class="section-num">3.8</span> 参考 {#参考}

-   <https://logmatic.io/blog/python-logging-with-json-steroids/>
-   <https://fangpenlin.com/posts/2012/08/26/good-logging-practice-in-python/>
-   <https://docs.python.org/3/howto/logging-cookbook.html>
-   <https://pymotw.com/3/logging/index.html>


### <span class="section-num">3.9</span> 小结 {#小结}

虽然这次的日志阐述是以 Python 的日志模块举例，但是绝大部分的语言都内置或者是有第三方的日志支持，所以我分享的技巧还是可以应用到其他的语言的。

这些都是我在日常项目中的一点体会，与诸君共赏罢。Enjoy :)

<div center class="qr-container">
<img src="/ox-hugo/qrcode_gh_e06d750e626f_1.jpg" alt="qrcode_gh_e06d750e626f_1.jpg" width="160px" height="160px" center="t" class="qr-container" />
公号同步更新，欢迎关注👻
</div>

