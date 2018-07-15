## 流数据？

流数据是一组顺序、大量、快速、连续到达的数据序列，一般情况下，数据流可被视为一个随时间延续而无限增长的动态数据集合。

### 特点

- 数据实时到达；
- 数据到达次序独立，不受应用系统所控制；
- 数据规模宏大且不能预知其最大值；
- 数据一经处理，除非特意保存，否则不能被再次取出处理，或者再次提取数据代价昂贵。


### js哪里用到流

Stream | Stream
--------------- | ---------------
Readable Stream | Writable Stream
Http response Stream | Http Request Stream
fs Read Stream | fs Write Stream
zlib Stream | zlib Stream
crypto Stream | crypto Stream
TCP Socket | TCP Socket
child process stdout and stderr | child process stdin
process.stdin | process.stdout, process.stderr


这些对象中的一部分是既可读、又可写的流，例如 TCP sockets，zlib 以及 crypto。

虽然一个 HTTP 响应在客户端是一个可读流，但在服务器端它却是一个可写流。这是因为在 HTTP 的情况中，我们基本上是从一个对象（http.IncomingMessage）读取数据，向另一个对象（http.ServerResponse）写入数据。

还需要注意的是 stdio 流（stdin，stdout，stderr）在子进程中有着与父进程中相反的类型。这使得在子进程中从父进程的 stdio 流中读取或写入数据变得非常简单。


###  Node.js流的四种类型

- writable：可写入数据的流（fs.createReadStream）
- readable: 可读取数据的流 (fs.createWriteStream)
- duplex： 可读取，可写入数据的流 (net.Socket)
- transform： 读写过程可以转换修改数据 （zlib.createGzip）

所有的流都是EventEmitter实例。

### 代码分析

Node应用程序有两种缓存的处理方式，第一种是等到所有数据接收完毕，一次性从缓存读取，这就是传统的读取文件的方式；第二种是采用“数据流”的方式，收到一块数据，就读取一块，即在数据还没有接收完成时，就开始处理它。

    var fs = require('fs')
    //管道命令（pipe）就起到在不同命令之间，连接数据流的作用
    fs.createReadStream('./file.txt').pipe(process.stdout)
    
fs.createReadStream方法就是以”数据流“的方式读取文件，这可以在文件还没有读取完的情况下，就输出到标准输出。这显然对大文件的读取非常有利。

##### 数据流事件
readable, writable, drain, data, end, close， error......

每读取一段数据，就会触发一次事件。数据全部读取完毕，就会触发end事件，发送错误触发error事件
    
    let stream = fs.createReadStream('file.txt')
    let data = ''
    
    stream.on('data', (chunk) => {
    	data += chunk
    	console.log(chunk)
    })
    
    stream.on('end', () => {
    	console.log(data)
    })

##### pipe()

pipe方法是自动传送数据的机制，就像管道一样。它从“可读数据流”读出所有数据，将其写出指定的目的地.

    var fs = required('fs')
    var readStream = fs.createReadStream('file1.txt')
    var writeStream = fs.createWriteStream('file2.txt')
    //参数为可写数据流
    readStream.pipe(writeStream)
    
    var zlib = require('zlib')
    //pipe链式处理
    fs.createReadStream('input.txt.gz')
        .pipe(zlib.createGunzip())
        .pipe(fs.createWriteStream('output.txt'));

##### 其他实例

监听readable事件

    let stream = fs.createReadStream('file.txt')
    let data = ''

    stream.on('readable', () => {
    	let chunk
    	while(chunk = stream.read()) {
    		data += chunk
    	}
    	console.log(data)
    })
    
暂停/恢复发送

    steam.pause()
    setTimeout(stream.resume(), 1000)
    
##### Js数据流转换为Json数据流

    var Transform = require('stream').Transform;
    var inherits = require('util').inherits;
    
    module.exports = JSONEncode;
    
    function JSONEncode(options) {
      if ( ! (this instanceof JSONEncode))
        return new JSONEncode(options);
    
      if (! options) options = {};
      options.objectMode = true;
      Transform.call(this, options);
    }
    
    inherits(JSONEncode, Transform);
    
    JSONEncode.prototype._transform = function _transform(obj, encoding, callback) {
      try {
        obj = JSON.stringify(obj);
      } catch(err) {
        return callback(err);
      }
    
      this.push(obj);
      callback();
    };

##### http的流应用

    var http = require('http')
    var server = http.createServer((req, res) => {
    	var data = ''
    	req.setEncoding('utf-8')
    	req.on('data', (chunk) => {
    		data += chunk
    	})
    	req.on('end', () => {
    		try{
    			var _data = JSON.parse(data)
    		} catch(err) {
    			res.statusCode = 400
    			return res.end('error:' + er.message)
    		}
    		res.write(typeof data)
    		res.end()
    	})
    })
    
    server.listen(8000)
    

