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
- Transform 是一种Duplex的流，当向他写数据并读出来时，转换数据 （比如 `zlib.createDeflate`)

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

## API for Stream Implementers

## Additional Notes
