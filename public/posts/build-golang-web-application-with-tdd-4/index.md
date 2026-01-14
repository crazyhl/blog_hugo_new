# 使用tdd构建golang Web 应用（4）


## 前置说明

本文以及接下来的文章都是来自 [https://quii.gitbook.io/learn-go-with-tests/](https://quii.gitbook.io/learn-go-with-tests/) 这个系列文章的。

主要分析说明的部分是 `Build An Application` 部分。

这并不是原文的翻译，而是记录一些自己的东西。混合式笔记，实现书中代码，加上我的思考

## 正文开始

看到标题，问题就来了，前面的文件结构都是按照我以前的认知来构建的，不过 golang 的结构，和跟以往了解的 php 有一些区别，虽然看了一些 golang 的结构，但是感觉使用起来不是很顺手。正好借着本篇文章在了解一下吧。

另外关于 TDD 的问题，我个人觉得随着经验的增长可以把每一步前进的快一点，比如最开始构造空方法，可以前进到，直接构建默认返回值的步骤。

今天的需求就是利用命令行实现一些请求的方法，而不仅仅是用 http。

哦吼，文章的第一段就是把 main 移动到 cmd/webserver 文件夹下，与我们的很相似，只要改个名就好了，展示一下我们当下的结构。另外包名文章中用的 poker，与我们的不一致，那我们就保持好我们当下的就好了。

![](https://www.cimple.ink/static/images/20211003101701.png)

现在开始在 cmd 下新建一个 cli 的文件夹，并且创建 main.go 文件。让其成为命令行的入口。

下面开始解决输入用户名记录获胜。从测试开始。新建 cli_test 注意，这个文件在根目录一下，而不是在 cmd 目录下。

```go
func TestCli(t *testing.T) {
	playerStore := &StubPlayerScore{}
	cli := &CLI{playerStore}
	cli.PlayPoker()

	if len(playerStore.winCalls) != 1 {
		t.Fatalf("expected a win call but didn't get any")
	}
}
```

可以看到，与我们前面的测试是很相似的，不过就是初始化不同了，这边初始化的是 CLI，以前的是 Server，这里是 PlayPoker，以前是 ServerHTTP。而且，参数也是一致的，都是 store。目前是会报错的，因为没有 CLI，准备实现它。

```go
// CLI.go
type CLI struct {
	playerStore PlayerStore
}

func (c *CLI) PlayPoker()  {

}
```

注意这个文件是在根目录下创建的。目前测试不会报错，但是测试会不通过，因为没有任何返回。

```go
func (c *CLI) PlayPoker() {
	c.playerStore.RecordWin("Cleo")
}
```

优化代码，现在测试可以通过了，但是无法通过输入的方式，传入用户名。所以接下来就要处理用户输入了。

```go
type CLI struct {
	playerStore PlayerStore
	in io.Reader
}
func TestCli(t *testing.T) {
	in := strings.NewReader("Chris wins\n")
	playerStore := &StubPlayerScore{}
	cli := &CLI{playerStore, in}
	cli.PlayPoker()

	if len(playerStore.winCalls) != 1 {
		t.Fatalf("expected a win call but didn't get any")
	}

	got := playerStore.winCalls[0]
	want := "Chirs"

	if got != want {
		t.Errorf("didn't record correct winner, got %q, want %q", got, want)
	}
}
```

增加了一个 reader 来处理用户输入。

```go
type CLI struct {
	playerStore PlayerStore
	in io.Reader
}

func (c *CLI) PlayPoker() {
	reader := bufio.NewScanner(c.in)
	reader.Scan()
	c.playerStore.RecordWin(extractWinner(reader.Text()))
}

func extractWinner(userInput string) string {
	return strings.Replace(userInput, " wins", "", 1)
}
```

这是优化后的代码，增加了一个 reader 来处理用户输入。为什么用 io.Reader 呢，这是个通用的接口，os.Stdin 也实现了这个接口。测试的时候用 strings.NewReader 来生成 reader 来测试。最后的 extractWinner 就是过滤一些文本，现在执行测试，可以通过了。另外 bufio，这个需要看下文档了。因为这个 io 实现了一些后续需要用到的东西。Scan 方法读取一行，Text 方法返回读取到的文本。

```go
func main() {
	fmt.Println("Let's play poker")
	fmt.Println("Type {Name} wins to record a win")

	db, err := os.OpenFile(dbFileName, os.O_RDWR | os.O_CREATE, 0666)

	if err != nil {
		log.Fatalf("problem opening %s %v", dbFileName, err)
	}

	store, err := go_http_application_with_tdd.NewFileSystemPlayerStore(db)

	if err != nil {
		log.Fatalf("problem creating file system player store, %v", err)
	}

	game := go_http_application_with_tdd.CLI{store, os.Stdin}
	game.PlayPoker()
}
```

现在处理运行部分，不过目前会报错，因为 store 和 in 都是私有的，要解决掉，后续文章讲述了如何更早的发现这个问题。

将测试文件的包名改为 包名_test 。这样就只能可以访问导出的公开的东西了。修改完包名后，就很多报错了。因为很多测试的数据都是在 _test 文件中定义的。创建一个 testing 文件用来保存我们的测试数据。把方法都独立出来。

```go
func TestCli(t *testing.T) {
	t.Run("record chris win from user input", func(t *testing.T) {
		in := strings.NewReader("Chris wins\n")
		playerStore := &go_http_application_with_tdd.StubPlayerScore{}
		cli := &go_http_application_with_tdd.CLI{playerStore, in}
		cli.PlayPoker()

		go_http_application_with_tdd.AssertPlayerWin(t, playerStore, "Chris")
	})

	t.Run("record cleo win from user input", func(t *testing.T) {
		in := strings.NewReader("Cleo wins\n")
		playerStore := &go_http_application_with_tdd.StubPlayerScore{}
		cli := &go_http_application_with_tdd.CLI{playerStore, in}
		cli.PlayPoker()

		go_http_application_with_tdd.AssertPlayerWin(t, playerStore, "Cleo")
	})
}
```

修改测试代码。

```go
type CLI struct {
	playerStore PlayerStore
	in          *bufio.Scanner
}

func NewCLI(store PlayerStore, in io.Reader) *CLI {
	return &CLI{
		playerStore: store,
		in: bufio.NewScanner(in),
	}
}

func (c *CLI) PlayPoker() {
	userInput := c.readline()
	c.playerStore.RecordWin(extractWinner(userInput))
}

func extractWinner(userInput string) string {
	return strings.Replace(userInput, " wins", "", 1)
}

func (c *CLI) readline() string {
	c.in.Scan()
	return c.in.Text()
}
```

通过本篇文章，我们了解了 New 方法的作用，这个方法，可以是我们的属性不用暴露出去，保持封装。

现在应该都 ok 了，不过两个 main 里面。是有重复的，那么就把这个重复的独立出来。

```go
func FileSystemPlayerStoreFromFile(path string) (*FileSystemPlayerStore, func(), error) {
	db, err := os.OpenFile(path, os.O_RDWR | os.O_CREATE, 0666)

	if err != nil {
		return nil, nil, fmt.Errorf("Problem opening %s %v", path, err)
	}

	closeFunc := func() {
		db.Close()
	}

	store, err := NewFileSystemPlayerStore(db)
	if err != nil {
		return nil, nil, fmt.Errorf("problem creating file system player store, %v", err)
	}

	return store, closeFunc, nil
}
```

然后修改两个 main 的调用就 ok 了。

## 总结

今天了解了 cli，以及如何更好的测试，还有关于包结构的问题。这篇文章让我有了很多的思考。但是还没法写出来。先把要总结的问题或者思考列出来吧，以后再总结

* 合理的总结包结构
* 测试的时候创建 testing 文件存放所有的测试方法，不污染其他方法。
* 测试包的包名 要 包名_test 这样可以测试公开的属性，不会让未暴露的属性被使用，模拟真正的调用
* New 方法就是方式 struct 的属性被暴露出来，保持合适的封装性。

---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/build-golang-web-application-with-tdd-4/  

