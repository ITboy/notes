# Stream

`Stream`是在Node.js中处理流式数据的抽象接口，`Stream`提供了一个基础的api，使得构造一个实现流的对象变得非常容易。
Node.js提供了许多流对象，比如Http模块的request对象，`process.stdout`都是`Stream`的实例。
流可以是可读的，可写的，或者既可读又可写，所有的流都是`EventEmitter`的实例。
流模块可以使用如下方式访问：

```js
const stream = require('stream');
```

对所有Node.js的使用者来说，理解Stream的工作原理都是非常重要的，`Stream`模块本身对构造流对象的开发者更加重要，只是使用Stream的开发者会很少直接使用`Stream`模块。

## Organization of this Document

这篇文档由两个主要部分和一个附加注释的部分组成。第一部分解释在应用中使用流对象的用户需要了解的api， 第二部分是针对使用流接口实现流对象的用户需要了解的api。

## Types of Streams

在Node.js中有4中基础的流对象：
- Readable 可以从中读取数据的流 (比如`fs.createReadStream())`）
- Writable 可以向里写数据的流 (比如`fs.createWriteStream(`）
- Duplex 既可以从中读数据，又可以向里写数据的流 （比如 `net.Socket`）
- Transform 是一种Duplex的流，当向他写数据并读出来时转换数据 （比如 `zlib.createDeflate`)

### Object Mode

所有在Node.js中流对象的api专门操作`String`和`Buffer`对象，但是流的实现其实是可以操作其他JavaScript对象的（除了null，null在Stream中有特殊的含义），这样的流被认为是操作在“Object Mode”。

译者注：
这里指的是在原生Node.js中提供的Stream实例，比如Http的Request或`process.stdout`都是专门操作`Stream`和`Buffer`对象的。
但是开发者可以实现出工作在”Object Mode“模式下的流，这些流可以操作其他除`null`以外的JavaScript对象。

### Buffering

[Writable]()和[Readable]()流都会把数据存放在内部的一个buffer中，可以使用`writable._writableState.getBuffer()`和`readable._readableState.buffer`来访问这个buffer。

这个buffer最多可以存多少数据，由流的构造函数的参数中`highWaterMark`选项决定，在正常模式下，它的单位是字节，在Object Mode下，他的单位是对象数。

在[Readable]()流中，当实现者调用stream.push(trunk)时，数据就别存储在这个buffer中，如果消费者不调用stream.read()，数据就会一直存在在这个buffer中，直到被消费为止。

一旦buffer的大小达到了`highWaterMark`所设置的临界点，流将临时停止读取底层资源的数据直到buffer中的数据被消费（这就是说，流会停止调用内部的`readable._read()` 方法，这个方法使用来填充read buffer的）。

在[Writable]()流中，当用户调用`writable.write(chunk)`时会往内部buffer中填充数据，如果buffer大小小于`highWaterMark`指定的临界值，方法返回true，否则返回false。

流中有一个专门的api`stream.pipe`，最主要的设计目的就是限制数据的buffer到一个可以接受的级别，这样，即便源和目标的读写速度不一致时，仍然不会压垮可使用的内存。

由于[Duplex]()流和[Retransform]()流既可读又写，所以内部维护了两个buffer，一个用来读，另一个用来写，使得在维护流的数据时每边不影响另一边的操作。
比如`net.Socket`是一个[Duplex]()流，他的Readable边可以从socket读取数据，Writable边也可以写数据，两边可以互不影响，互不依赖，因为写的速度可以比读的速度快也可以比他慢。

## API for Stream Consumers

不论一个Node.js应用程序多么简单，几乎都会用到流，下面是在Node.js中实现Http server的简单应用：

``` js
const http = require('http');

const server = http.createServer( (req, res) => {
  // req is an http.IncomingMessage, which is a Readable Stream
  // res is an http.ServerResponse, which is a Writable Stream

  let body = '';
  // Get the data as utf8 strings.
  // If an encoding is not set, Buffer objects will be received.
  req.setEncoding('utf8');

  // Readable streams emit 'data' events once a listener is added
  req.on('data', (chunk) => {
    body += chunk;
  });

  // the end event indicates that the entire body has been received
  req.on('end', () => {
    try {
      const data = JSON.parse(body);
      // write back something interesting to the user:
      res.write(typeof data);
      res.end();
    } catch (er) {
      // uh oh!  bad json!
      res.statusCode = 400;
      return res.end(`error: ${er.message}`);
    }
  });
});

server.listen(1337);

// $ curl localhost:1337 -d '{}'
// object
// $ curl localhost:1337 -d '"foo"'
// string
// $ curl localhost:1337 -d 'not json'
// error: Unexpected token o
```

[Writable]()流（比如在例子中的`res`）暴露方法向流写数据，比如`write()`和`end()`。

[Writable]()流使用`EventEmitter`API来通知应用程序的代码，存在可用的数据可以从流中读取，读取数据的方式可以有多种。

[Writable]()和[Readable]()流都使用`EventEmitter`来通知流内部当前状态的变化。

[Duplex]()和[Transform]()流既是[Writable]()又是[Readable]()。

应用程序开发者可以轻易的从流中读取数据或写入数据，而不需要去实现一个流，因此也就不需要调用`require 'stream'`。

想要实现一个新的流的开发者可以参考[API for Stream Implementer](#api-for-stream-implementer)。

### Writable Streams

可写的流其实是一个目的地的抽象，数据可以写入这个目的地。所以可以理解为可写流就是一个可以写入数据的地方。

[Writable]()流有如下几个例子：
* HTTP requests, on the client
* HTTP responses, on the server
* fs write streams
* zlib streams
* crypto streams
* TCP sockets
* child process stdin
* process.stdout, process.stderr

注意：以上例子中有的流其实是`Duplex`流，但是实现了[Writable]()接口。

所有实现[Writable]()的接口都在`stream.Writable` class中定义。

实现Writable接口的类在细节上可以有不同，但是都遵循一些基础的模型如下所示：

```js
const myStream = getWritableStreamSomehow();
myStream.write('some data');
myStream.write('some more data');
myStream.end('done writing data');
```

#### Class: stream.Writable

##### Event: 'close'

当底层资源被关闭的时候会产生一个`close`事件（比如文件描述符关闭的时候）。这意味着之后没有任何事件产生，也不会有任何的计算发生。

并非所有的Writable流都会产生`close`事件。

##### Event: 'drain'

如果调用[stream.write(chunk)]()返回`false`，当重新可以向流中写入数据时，会产生`drain`事件。

```js
// Write the data to the supplied writable stream one million times.
// Be attentive to back-pressure.
function writeOneMillionTimes(writer, data, encoding, callback) {
  let i = 1000000;
  write();
  function write() {
    var ok = true;
    do {
      i--;
      if (i === 0) {
        // last time!
        writer.write(data, encoding, callback);
      } else {
        // see if we should continue, or wait
        // don't pass the callback, because we're not done yet.
        ok = writer.write(data, encoding);
      }
    } while (i > 0 && ok);
    if (i > 0) {
      // had to stop early!
      // write some more once it drains
      writer.once('drain', write);
    }
  }
}
```

##### Event: 'error'

* &lt;Error&gt;

当写入数据或pipe数据失败时，会产生`error`事件，监听者的回调函数会被传入一个`Error`参数。

注意： 当`error`事件产生时，流并不会被关闭。

##### Event: 'finish'

当调用`stream.end()`后，会产生`finish`，并且所有的数据会被flush到底层系统。

```js
const writer = getWritableStreamSomehow();
for (var i = 0; i < 100; i ++) {
  writer.write(`hello, #${i}!\n`);
}
writer.end('This is the end\n');
writer.on('finish', () => {
  console.error('All writes are now complete.');
});
```

##### Event: 'pipe'

* src [&lt;stream.Readable&gt;]() 正在pipe到当前writable流的源stream

当`stream.pipe()`被调用在一个可读的流上，把当前可写的流作为他写入的目的地时，会产生一个pipe事件。

##### writable.cork()

`writable.cork()`方法会强制使得所有已经写入stream的数据被缓存在内存中，直到调用`writable.uncork()`或者`writable.end()`缓存数据才会被flush。

`writable.cork()`的主要意图是避免写入太多小的数据包到流中，可能不会引起在内部buffer中备份，这样就会产生性能上的影响。在这种情况下，对`writable._writev()`的实现会执行缓存来优化性能。

译者注：这里不是非常确定是这样理解的。

##### writable.end([chunk][, encoding][, callback])

* `chunk` &lt;string&gt;|&lt;Buffer&gt;|&lt;any|&gt; 可选参数。指定要写入的数据，如果在非Object模式，可以时string或Buffer，否则可以时除了null以外的任意类型
* `encoding` &lt;string&gt; 可选参数。如果chunk时string，指定他的编码类型。
* `callback` &lt;Function&gt; 可选参数。当流结束的时候调用的回调函数。

调用`writable.end()`来声明以后再没有数据写入，参数`chunk`允许在关闭流之前立即写入最后一个数据，如果提供了，最后一个参数callback就会监听`finish`事件。

在`stream.end()`之后再调用`stream.write()`会抛出错误。

```js
// write 'hello, ' and then end with 'world!'
const file = fs.createWriteStream('example.txt');
file.write('hello, ');
file.end('world!');
// writing more now is not allowed!
```

##### writable.setDefaultEncoding(encoding)
* encoding &lt;String&gt; 设置默认编码类型
* Returns: this

##### writable.uncork()

`writable.uncork()`会flush所有在`writable.cork()`调用之后缓存的数据，推荐把`writable.cork()`的调用放到`process.nextTick()`中。

```js
stream.cork();
stream.write('some ');
stream.write('data ');
process.nextTick(() => stream.uncork());
```

如果`writable.cork()`被调用多次，那么同样`writable.uncork()`也应该被调用相同的次数。

```js
stream.cork();
stream.write('some ');
stream.cork();
stream.write('data ');
process.nextTick(() => {
  stream.uncork();
  // The data will not be flushed until uncork() is called a second time.
  stream.uncork();
});
```

##### writable.write(chunk[, encoding][, callback])
* chunk <String> | <Buffer> 要写入的数据
* encoding <String> 如果chunk是字符串，指定他的编码类型
* callback <Function> 当chunk被flush的时候调用的回调函数
* Returns: <Boolean> 如果流希望调用代码等待`drain`事件，收到后继续写入额外的数据的话就返回false，否则返回true

译者注： 返回false并不意味着数据写入失败，只是代表buffer已经满了，下次再写入的时候要在收到drain事件之后。

写入一些数据到stream后，一旦数据被处理完成，接下来会调用提供的`callback`，如果发生错误，可能也可能不会调用并且传递一个error参数，可靠的办法时监听`error`事件

如果内部buffer的大小小于`highWaterMark`，返回true，如果大于，那么返回false，并且下次write需要在收到`drain`事件之后。

当流还没有被用尽时（buffer已经被写满，但是还没有被底层系统完全把他消化完全），调用`write()`会返回false，一旦所有被缓存的chunks被用尽，会产生`drain`事件，推荐一旦`write()`返回false，就不要再调用`write()`只等到`drain`事件发生，但是继续调用也时允许的，Node.js会继续缓存数据，直到达到内存的最大内存限制，这时程序会异常退出。甚至在退出之前，高内存使用会引起内存回收机制性能低下。如果TCP socket可能永远不会被用尽，如果对端不消费数据，所以在没有用尽的时候就调用`write()`会产生潜在弱点。

对于[Transform]()流,在没有用尽的情况下写入数据是一个典型的问题，因为默认情况下，没有为[Transform]()监听`data`或`redable`事件而且没有pipe，那么流处于暂停状态，也就是没有任何消费者。

如果要写入流的数据可以按照需要去取，那么可以把生成数据这部分封装到[Readable]()流中，然后使用pipe，写入[Writable]()流。可是如果写更优先时，为了避免内存问题，就需要使用`drain`事件。

```js
function write (data, cb) {
  if (!stream.write(data)) {
    stream.once('drain', cb)
  } else {
    process.nextTick(cb)
  }
}

// Wait for cb to be called before doing any other write.
write('hello', () => {
  console.log('write completed, do more writes now')
})
```

在Object模式下，[Writable]()流会忽略`encoding`参数。

### Readable Streams

可读流是可以被消费的数据源的抽象。

Readable流的例子如下：

* HTTP responses, on the client
* HTTP requests, on the server
* fs read streams
* zlib streams
* crypto streams
* TCP sockets
* child process stdout and stderr
* process.stdin

所有只读流的都实现`stream.Readable`的接口类。

#### 两个模式

可读流可以在两个模式下工作：
1. flowing
2. paused

在flowing模式下，底层子系统自动读取数据，并且以`EventEmitter`接口，尽量快的速度提供给应用程序

在paused模式下，必须显式调用stream.read()来获取数据。

所有可读流开始都处于paused模式，但是可以通过以下方式切换到flowing模式：
1. 添加'data'事件监听器
2. 调用`stream.resume()`方法
3. 调用`stream.pipe()`把数据发送给一个[Writable]()流

可读流也可以切换回paused模式，方法如下：
1. 如果没有pipe的目的地，调用`stream.pause()`
2. 如果有pipe的目的地，调用`stream.unpipe()`移除所有的pipe的目的地，并且移除`data`事件监听器

综上，有一个重要的概念，如果没有提供一种方式消费或忽略数据，将不会有数据产生，同样如果当数据产生时，如果把消费或忽略的数据移除时，流也就不会产生数据（paused）。

译者注：
1. 只要pipe了目的地，就一定在flowing模式，哪怕调用了`pause()`
2. 如果没有pipe目的地，但时绑定了`data`事件，默认时在flowing模式，但是可以使用`pause()`来进入paused状态。
3. 如果没有pipe的目的地，也没有绑定`data`，那么就处于paused状态，但是可以使用`resume()`来进入flowing模式，会忽略掉生成的数据。

注意：处于向后兼容的考虑，移除`data`事件不会自动暂停流，同样，如果有pipe，调用`paused()`也不会暂停流。

注意：如果流处于flowing模式，而没有可用的消费者，数据会丢失。比如当没有添加`data`事件监听器却调用了`resume()`时，或者移除了`data`事件监听器时。

#### 三种状态

paused和flowing两种模式只是三种内部实现状态的一种简单抽象，更细节的要看这三种状态：
* readable.\_readableState.flowing = null
* readable.\_readableState.flowing = false
* readable.\_readableState.flowing = true

当`readable.\_readableState.flowing = null`，没有提供消费数据的机制，数据不会被生成。

监听`data`事件，调用`stream.resume()`，调用`stream.pipe()`都会设置`readable.\_readableState.flowing = true`，引起流当数据生成时产生`data`事件。

调用`stream.pause()`， `stream.unpipe()`会设置`readable.\_readableState.flowing = false`，会停止产生`data`事件，却不会停止生成数据。

当`readable._readableState.flowing`值为`false`时，数据会被累计到内部buffer中。

#### 选择一种方式

可读流在Node.js的多个版本中提供多种方式消费数据，通常情况下，开发者应该选择其中一种方式，而不应该在一个流中使用多种方式消费数据。

对于大多数用户推荐使用`readable.pipe()`，因为他是已经实现的最简单的方式使用数据，如果开发者想更细粒度控制，可以使用`EventEmitter`接口和`readable.pause()`， `readable.resume()` API。

#### Class: stream.Readable

##### Event: 'close'

当底层资源被关闭的时候会产生一个`close`事件（比如文件描述符关闭的时候）。这意味着之后没有任何事件产生，也不会有任何的计算发生。

并非所有的Readable流都会产生`close`事件。

##### Event: 'data'

* chunk <Buffer> | <String> | <any>
  在非Object模式，chunk是`Buffer`或string，但是在Object模式下，chunk可以是除null以外的所有对象。


##### Event: 'data'
## API for Stream Implementers

## Additional Notes
