

## go 日志标准库

### 快速使用

log默认输出到标准错误（stderr），每条日志前会自动加上日期和时间。如果日志不是以换行符结尾的，那么log会自动加上换行符。即每条日志会在新行中输出。

log提供了三组函数：

- Print/Printf/Println：正常输出日志；
- Panic/Panicf/Panicln：输出日志后，以拼装好的字符串为参数调用panic；
- Fatal/Fatalf/Fatalln：输出日志后，调用os.Exit(1)退出程序。

命名比较容易辨别，带`f`后缀的有格式化功能，带`ln`后缀的会在日志后增加一个换行符。

```go

type User struct {
  Name string
  Age  int
}

// 基本使用
func quicklyStart() {
	u := User{
		Name: "lisi",
		Age:  18,
	}
	log.Printf("%s login, age :%d", u.Name, u.Age)
	log.Fatalf("Danger! hacker %s login", u.Name)        // 会触发os.Exit(1)
	log.Panicf("Oh, system error when %s login", u.Name) // 触发 panic(s)
}
```



### 定制

#### 前缀

调用`log.SetPrefix`为每条日志文本前增加一个前缀。例如，在上面的程序中设置`Login:`前缀：

```go
// 2. 设置前缀
func prefixExample() {
	u := User{
		Name: "lisi",
		Age:  18,
	}
	log.SetPrefix("login:")
	log.Printf("%s login, age:%d", u.Name, u.Age)
}

```

输出：

```go
login:2023/12/19 11:58:11 lisi login, age:18
```

调用`log.Prefix`可以获取当前设置的前缀。

#### 选项

设置选项可在每条输出的文本前增加一些额外信息，如日期时间、文件名等。

`log`库提供了 6 个选项：

```go
const (
	Ldate         = 1 << iota     // the date in the local time zone: 2009/01/23
	Ltime                         // the time in the local time zone: 01:23:23
	Lmicroseconds                 // microsecond resolution: 01:23:23.123123.  assumes Ltime.
	Llongfile                     // full file name and line number: /a/b/c/d.go:23
	Lshortfile                    // final file name element and line number: d.go:23. overrides Llongfile
	LUTC                          // if Ldate or Ltime is set, use UTC rather than the local time zone

)

```

- Ldate：输出当地时区的日期，如2020/02/07；
- Ltime：输出当地时区的时间，如11:45:45；
- Lmicroseconds：输出的时间精确到微秒，设置了该选项就不用设置Ltime了。如11:45:45.123123；
- Llongfile：输出长文件名+行号，含包名，如github.com/go-quiz/go-daily-lib/log/flag/main.go:50；
- Lshortfile：输出短文件名+行号，不含包名，如main.go:50；
- LUTC：如果设置了Ldate或Ltime，将输出 UTC 时间，而非当地时区。

调用`log.SetFlag`设置选项，可以一次设置多个：

```go
func setFlagExample() {
	u := User{
		Name: "lisi",
		Age:  18,
	}
	// 设置多个选项
	log.SetFlags(log.Lshortfile | log.Ldate | log.Lmicroseconds)
	log.Printf("%s login, age:%d", u.Name, u.Age)
}
	// 2023/12/19 12:04:20.108220 main.go:46: lisi login, age:18
```

调用`log.Flags()`可以获取当前设置的选项。

注意，调用`log.SetFlag`之后，原有的选项会被覆盖掉！

另外还定义了一个包含了日期和时间初始化选项

```go
LstdFlags     = Ldate | Ltime // initial values for the standard logger

var std = New(os.Stderr, "", LstdFlags)
```

这就是为什么默认情况下，每条日志前会自动加上日期和时间。

#### 自定义输出

实际上，`log`库为我们定义了一个默认的`Logger`，名为`std`，意为标准日志。我们直接调用的`log`库的方法，其内部是调用`std`的对应方法：

```go
// src/log/log.go
var std = New(os.Stderr, "", LstdFlags)

func Printf(format string, v ...interface{}) {
  std.Output(2, fmt.Sprintf(format, v...))
}

func Fatalf(format string, v ...interface{}) {
  std.Output(2, fmt.Sprintf(format, v...))
  os.Exit(1)
}

func Panicf(format string, v ...interface{}) {
  s := fmt.Sprintf(format, v...)
  std.Output(2, s)
  panic(s)
}

```

当然，我们也可以定义自己的`Logger`：

```go
func customLogger() {
	u := User{
		Name: "lisi",
		Age:  18,
	}

	buf := &bytes.Buffer{} // 定义 io.writer
	logger := log.New(buf, "", log.Lshortfile|log.LstdFlags)
	logger.Printf("%s login, age:%d", u.Name, u.Age)
	fmt.Print(buf.String())

}
```

`log.New`接受三个参数：

- `io.Writer`：日志都会写到这个`Writer`中；
- `prefix`：前缀，也可以后面调用`logger.SetPrefix`设置；
- `flag`：选项，也可以后面调用`logger.SetFlag`设置。

上面代码将日志输出到一个`bytes.Buffer`，然后将这个`buf`打印到标准输出。

注意到，第一个参数为`io.Writer`，我们可以使用`io.MultiWriter`实现多目的地输出。下面我们将日志同时输出到标准输出、`bytes.Buffer`和文件中：

```go
// 自定义多个输出
func multiWriter() {
	u := User{
		Name: "lisi",
		Age:  18,
	}

	writer1 := &bytes.Buffer{}                                            // 输出到 buffer 缓存
	writer2 := os.Stdout                                                  // 标准输出
	writer3, err := os.OpenFile("log.txt", os.O_WRONLY|os.O_CREATE, 0755) // 输出到文件
	if err != nil {
		log.Fatalf("create file log.txt failed: %v", err)
	}

	logger := log.New(io.MultiWriter(writer1, writer2, writer3), "", log.Lshortfile|log.LstdFlags)
	logger.Printf("%s login, age:%d", u.Name, u.Age)
}

```

### log核心实现

`log`库的核心是`Output`方法

```go
// Output writes the output for a logging event. The string s contains
// the text to print after the prefix specified by the flags of the
// Logger. A newline is appended if the last character of s is not
// already a newline. Calldepth is used to recover the PC and is
// provided for generality, although at the moment on all pre-defined
// paths it will be 2.
func (l *Logger) Output(calldepth int, s string) error {
	now := time.Now() // get this early.
	var file string
	var line int
	l.mu.Lock()
	defer l.mu.Unlock()
	if l.flag&(Lshortfile|Llongfile) != 0 {
		// Release lock while getting caller info - it's expensive.
		l.mu.Unlock()
		var ok bool
		_, file, line, ok = runtime.Caller(calldepth)
		if !ok {
			file = "???"
			line = 0
		}
		l.mu.Lock()
	}
	l.buf = l.buf[:0]
	l.formatHeader(&l.buf, now, file, line)
	l.buf = append(l.buf, s...)
	if len(s) == 0 || s[len(s)-1] != '\n' {
		l.buf = append(l.buf, '\n')
	}
	_, err := l.out.Write(l.buf)
	return err
}

```

如果设置了`Lshortfile`或`Llongfile`，`Ouput`方法中会调用`runtime.Caller`获取文件名和行号。`runtime.Caller`的参数`calldepth`表示获取调用栈向上多少层的信息，当前层为 0。

一般的调用路径是：

- 程序中使用`log.Printf`之类的函数；
- 在`log.Printf`内调用`std.Output`。

在`Output`方法中需要获取调用`log.Printf`的文件和行号。

`calldepth`传入 0 表示`Output`方法内调用`runtime.Caller`的那一行信息，传入 1 表示`log.Printf`内调用`std.Output`那一行的信息，传入 2 表示程序中调用`log.Printf`的那一行信息。显然这里要用 2。

然后调用`formatHeader`处理前缀和选项。

最后将生成的字节流写入到`Writer`中。

这里有两个优化技巧：

- 由于`runtime.Caller`调用比较耗时，先释放锁，避免等待时间过长；
- 为了避免频繁的内存分配，`logger`中保存了一个类型为`[]byte`的`buf`，可重复使用。前缀和日志内容先写到这个`buf`中，然后统一写入`Writer`，减少 io 操作。