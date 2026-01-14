---
title: "使用tdd构建golang Web 应用（1）"
date: 2021-10-01T10:30:50+08:00
categories: [Golang]
slug: build-golang-web-application-with-tdd
---

## 前置说明

本文以及接下来的文章都是来自 [https://quii.gitbook.io/learn-go-with-tests/](https://quii.gitbook.io/learn-go-with-tests/) 这个系列文章的。

主要分析说明的部分是 `Build An Application` 部分。

这并不是原文的翻译，而是记录一些自己的东西。混合式笔记，实现书中代码，加上我的思考

## 正文开始

我们要创建一个 HTTP 服务，用户可以追踪一个玩家获得胜利的数量。有两个接口

* GET /player/{name} GET 请求 会返回传入玩家的胜利总数
* POST /player/{name} POST 请求 会记录传入玩家的胜场，每次请求递增胜场

TDD 方法，为尽快获得工作的软件进行最小的迭代改进，直到找到解决方案。应遵循下面几点

* 让问题空间尽可能保持小（也就是说要把问题拆分尽可能的独立。不要在一个范围内解决非常多的问题）
* Don't go down rabbit holes 不要掉进兔子洞（参考这里 [https://www.zhihu.com/question/268877435](https://www.zhihu.com/question/268877435) 。个人理解就是要是问题规模可控，而不是每次修改还会引发其他的影响。）
* 如果我们被卡主或者遗失部分，在进行还原的时候不会进行大量的工作（依然还是保持精简的思想。）

上面说了一堆，就是要求小步快速迭代，每次重构 or 修改只进行少量小范围的改动，使其可用，然后在逐步进化为期望状态。这里我会在后续代码中，再次添加自己的见解。

> The more changes you make while in red, the more likely you are to add more problems, not covered by tests.
> The idea is to be iteratively writing useful code with small steps, driven by tests so that you don't fall into a rabbit hole for hours.
>

这里我附上了原文，说的很好，我们在开发中也经常会有这种问题，当然不仅仅是写代码，也包含很多东西，比如每次提交只提交一个问题的修改，而不是一次提交很多解决问题。否则回滚的时候也会也引发噩梦。代码依然如此，不要一次修改很多问题，这也会致使你忽略一些必要的测试。

Mocking 这个东西在我以前的开发中是经常忽略的，在测的时候如果需要数据，我经常会直连数据库去获取数据，当测试出现问题，就要查两方面的内容，到底是数据处理有问题，还是获取数据有问题。 而 mocking 就解决了这个问题，我们不从真实的数据源获取数据，而是构造一个稳定安全的模拟数据源。这样当排查问题的时候，只要检测我们的逻辑部分就可以了。这也就是解耦。

还有一个原则，先写测试。

```go
package go_http_application_with_tdd

import (
	"net/http"
	"net/http/httptest"
	"testing"
)

func TestGETPlayers(t *testing.T) {
	t.Run("returns Pepper's score", func(t *testing.T) {
		request, _ := http.NewRequest(http.MethodGet, "/players/Pepper", nil)
		response := httptest.NewRecorder()

		PlayerServer(response, request)

		got := response.Body.String()
		want := "20"

		if got != want {
			t.Errorf("got %q, want %q", got, want)
		}
	})
}

```

所以，第一步编写测试，测试获取名为 Pepper 的玩家的分数。我们预期的分数是 20。目前的代码是会报错的，我们不用管，先分析一下这里都做什么什么，构造一个 request 这个就是标准的流程了。response 是通过 httptest 的方法创造额，我们看下说明

> ```go
> // ResponseRecorder is an implementation of http.ResponseWriter that
> // records its mutations for later inspection in tests.
> ```
>

可以看到这个就是为我们测试的时候准备的 response。具体的可以看这个系列前面的一些文章，里面有详尽的解释。

`PlayerServer` 这个是我们整个代码报错的地方，这个稍后再说，对了，这里要说下的思考，在我们测试的时候如果有报错，不要超过 1 ，否则就是个麻烦。

后面的就是从 response 获取内容跟预期比较是否一致了。

那我们就要仔细看看 `PlayerServer` 这个方法了。为什么要定义这个方法，并且传入那两个参数，以及 response 在前 request 在后呢？因为这个是一个 web 项目，启动 web 服务通过方法 `func ListenAndServe(addr string, handler Handler) error` 第一个参数是监听的地址和端口，第二个参数一个 handler。handler 是一个接口

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

所以我们的 `PlayerServer` 方法就是实现这个接口的 `ServerHttp`，来处理我们的请求。所以这个方法的疑惑我们解决了。（注意，这里还有一个问题没说清楚，就是我们仅仅实现了这个方法，但是我们传入的应该是个实现了这个接口的 `strruct`，别急，后面会把这个坑解决掉）。

好了，此时我们运行这个测试，不出意外就会报错了。报错内容就是没有 `PlayerServer` 这个方法，接下来我们就要解决掉这个报错。时刻记住，只需解决掉这个报错就好，不要做太多的操作，一步一步来。谨遵上面 TDD 的方法论。

好了，接下来就是创建这个方法了。所以 `func PlayerServer() {}` 我们只要声明这个方法就好了。运行依然报错，

```go
# github.com/crazyhl/go-http-application-with-tdd [github.com/crazyhl/go-http-application-with-tdd.test]
./server_test.go:14:15: too many arguments in call to PlayerServer
	have (*httptest.ResponseRecorder, *http.Request)
	want ()
```

可以看到，告诉我们传入了太多的参数，我们定义方法没有接受参数。那么下一步，就是补全这个参数，依然记住，异步一步来。

```go
func PlayerServer(w http.ResponseWriter, r *http.Request) {}
```

注意这里我们跟提示传入的参数不一致的，提示我们传入的是 `*httptest.ResponseRecorder` 实际上我们接受的是 `http.ResponseWriter` 为什么要这样呢，因为 `ResponseRecorder` 是我们测试用的，通过说明我们可以看到 `NewRecorder returns an initialized ResponseRecorder` 是 `ResponseRecorder` 的实例，而 `ResponseRecorder` 又实现了 `http.ResponseWriter` 接口，而参数通过接受接口作为参数的方式，将会使我们后续的代码编写更具有通用性。（这里我觉得我的说明的不是很好，还可以理解为等我们写真正的业务的时候就不用改动这个方法了，不改动更安全。另外关于多态我就没说了，我觉得实际体验下来才是最重要的，关于多态可以看这个系列文章）。

好了再次运行测试，可以看到，报错解决掉了，我们已经前进了很大异步，不过测试依然没有通过因为我们返回内容跟预期不一致 `server_test.go:20: got "", want "20"` 想要 20 却返回空字符串。既然如此那就让方法直接返回 20 就好了。

```go
func PlayerServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "20")
}
```

为什么用 fmt.Fprintf 方法呢，通过说明可以知道这个方法是向 `io.Writer` 写内容的。而 `http.ResponseWriter` 中也有 `io.Writer` 的方法。所以这是可以正常写入的。

现在执行测试，通过了。接下来，我们就可以实现 `http` 服务器了。

```go
package main

import (
	go_http_application_with_tdd "github.com/crazyhl/go-http-application-with-tdd"
	"log"
	"net/http"
)

func main() {
	handler := http.HandlerFunc(go_http_application_with_tdd.PlayerServer)
	log.Fatal(http.ListenAndServe(":5000", handler))
}
```

到这，可以看到我们的服务也可以正常跑起来了，注意这里的 `http.HandlerFunc` 这个类型转换，把 `PlayerServer` 转换为 `http.HandlerFunc` 类型，这个类型实现了 `Handler` 接口，至此，我们上面的那个坑也搞定了。另外这里还需要夸 `golang` 了，可以吧一个方法定义为一个类型，并且还能实现其他方法。高级！！！

现在，程序已经正产跑起来了，但是正确么？不，我们不论输入什么返回的都是 20，这与我们预期的不一样，我们预期是输入不同人，输出每个人自己的结果。

好继续编写测试代码。

```go
t.Run("returns Floyd's score", func(t *testing.T) {
		request, _ := http.NewRequest(http.MethodGet, "/players/Floyd", nil)
		response := httptest.NewRecorder()

		PlayerServer(response, request)

		got := response.Body.String()
		want := "10"

		if got != want {
			t.Errorf("got %q, want %q", got, want)
		}
	})
```

可以看到这里跟上面的那个很像，区别在哪呢我们输入的玩家姓名不一样。这里我们预期的分数是 10 分。运行测试，失败了。因为我们输出的还是 20。

这里文档有说一段，

> You may have been thinking
>
>> Surely we need some kind of concept of storage to control which player gets what score. It's weird that the values seem so arbitrary in our tests.
>>
>
> Remember we are just trying to take as small as steps as reasonably possible, so we're just trying to break the constant for now.
>

这段我觉得非常好，我们测试的时候也是要演进的，一步一步来。

继续，现在我们不能够固定返回 20 了。而是要根据传入的玩家姓名来获取对应玩家的分数。修改代码

```go
func PlayerServer(w http.ResponseWriter, r *http.Request) {
	player := strings.TrimPrefix(r.URL.Path, "/players/")

	if player == "Pepper" {
		fmt.Fprint(w, "20")
		return
	}

	if player == "Floyd" {
		fmt.Fprint(w, "10")
		return
	}
}
```

获取玩家名字，通过 trim 字符串来获取的，然后通过不用的用户名返回不同的分数。注意这并不是最终代码，代码是一步步进化出来的。我们来做个简单的进化。

```go
func PlayerServer(w http.ResponseWriter, r *http.Request) {
	player := strings.TrimPrefix(r.URL.Path, "/players/")

	fmt.Fprint(w, GetPlayerScore(player))
}

func GetPlayerScore(player string) string {
	if player == "Pepper" {
		return "20"
	}

	if player == "Floyd" {
		return "10"
	}
	return ""
}
```

我们把通过用户名获取玩家分数剥离出来，让我们代码不仅方便测试，也使 `PlayerServer` 更加精简，并且业务没有耦合进去。好了，下一步我们进化一下测试代码。

```go
func TestGETPlayers(t *testing.T) {
	t.Run("returns Pepper's score", func(t *testing.T) {
		request := newGetScoreRequest("Pepper")
		response := httptest.NewRecorder()

		PlayerServer(response, request)

		assertResponseBody(t, response.Body.String(), "20")
	})

	t.Run("returns Floyd's score", func(t *testing.T) {
		request := newGetScoreRequest("Floyd")
		response := httptest.NewRecorder()

		PlayerServer(response, request)

		assertResponseBody(t, response.Body.String(), "10")
	})
}

func newGetScoreRequest(name string) *http.Request {
	request, _ := http.NewRequest(http.MethodGet, fmt.Sprintf("/players/%s", name), nil)
	return request
}

func assertResponseBody(t testing.TB, got, want string) {
	t.Helper()
	if got != want {
		t.Errorf("response body is wrong, got %q, want %q", got, want)
	}
}
```

这里，我们就是把重复的代码独立成为方法，这个在我们的开发中也要注意，重复的方法抽出来，方便修改管理。

修改完别忘了测试哟，如果正常，依然会测试通过。如果出问题了就说明使我们抽象代码的时候出问题了。修改起来也很块。

接下来，考虑一个问题，上面我们说过，我们的玩家分数是存储在某个地方的。那么我们获取数据的数据源在测试和生产环境可能不是一个。上面我们也讨论过这个东西，这时候我们应该怎么做呢，抽象，生成一个接口。让各自的数据源，有不同的获取方式。开工。

```go
type PlayerStore interface {
	GetPlayerScore(name string) int
}

type PlayerServer struct {
	store PlayerStore
}

func (p *PlayerServer)ServeHTTP(w http.ResponseWriter, r *http.Request) {
	player := strings.TrimPrefix(r.URL.Path, "/players/")

	fmt.Fprint(w, p.store.GetPlayerScore(player))
}
```

这里我们删除了 `PlayerServer` 方法，并且创建了 `PlayerStore` 接口，创建了 `PlayerServer` 结构体，并且这个结构体有个 `ServeHTTP` 方法，这个方法实现了我们删除的方法，接口我们找到了，为什么还要创先一个结构体呢，那么是为了有地方拿到 `store` 的引用。为什么要拿到引用呢，原因就是能够调用接口的方法。另外为什么要有 `ServerHttp` 方法呢，也是要让我们的这个结构体实现接口 `Handler` 方法，在我们正式运行代码的时候就不用转换为 `HanderFunc` 了。 此时测试的时候就会报错了，现在我们来修改代码

```go
func TestGETPlayers(t *testing.T) {
	server := PlayerServer{}
	t.Run("returns Pepper's score", func(t *testing.T) {
		request := newGetScoreRequest("Pepper")
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertResponseBody(t, response.Body.String(), "20")
	})

	t.Run("returns Floyd's score", func(t *testing.T) {
		request := newGetScoreRequest("Floyd")
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertResponseBody(t, response.Body.String(), "10")
	})
}
```

此时，已经不会报错了，但是会报错空指针。为什么会空呢，因为我们的 store 初始化，继续修改。

```go
type StubPlayerScore struct {
	scores map[string]int
}

func (s *StubPlayerScore) GetPlayerScore(name string) int {
	score := s.scores[name]
	return score
}
```

`Store` 是个接口，在测试的时候我们就需要提供测试用的数据源获取了。如同上面的代码。紧接着修改测试代码初始化 `Store` 。

```go
store := StubPlayerScore{
		map[string]int{
			"Pepper": 20,
			"Floyd": 10,
		},
	}
	server := PlayerServer{&store}
```

只放了修改的部分，初始化测试用 `Store` 并且初始化了测试用数据。紧接着把 `Store` 赋值给 `Server` 。紧接着测试已经可以通过了。接下来要弄正式运行的部分的，我们也要为正式运行的代码提供数据源。

```go
type InMemoryPlayerStore struct {}

func (i *InMemoryPlayerStore) GetPlayerScore(name string) int {
	return 123
}
```

我们提供了一个在内存中的 `Store` ，注意这里我们提供的是可运行的最小化代码。现在我们的测试和正式运行都可以运行并且通过测试了。

目前，我们的代码还是有不少问题没有结局的，比如访问任何路径都会返回 123。还有 POST 没有处理，以及用户名不存在的情况没有处理。ok，继续。先弄获取用户不存在的情况。开始写测试。

```go
t.Run("returns 404 on missing players", func(t *testing.T) {
		request := newGetScoreRequest("Apollo")
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		got := response.Code
		want := http.StatusNotFound
		if got != want {
			t.Errorf("got status %d want %d", got, want)
		}
	})
```

执行会报错 `got status 200 want 404` 。返回的是 200，而不是期望的 404。既然是想要获取 `StatusCode` 那么就开始做最小化的修改，先让 response 返回 404。

```go
func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	player := strings.TrimPrefix(r.URL.Path, "/players/")
	w.WriteHeader(http.StatusNotFound)
	fmt.Fprint(w, p.Store.GetPlayerScore(player))
}
```

现在，测试都可以通过了。不过现在是争取的么，没有，因为我们之前的测试并没有测试返回的 `StatusCode` 现在我们添加上相关的代码，注意，这里有重复判断 `StatusCode` 代码，那我们依然给抽象出来。

```go
func assertStatusCode(t testing.TB, got, want int) {
	t.Helper()
	if got != want {
		t.Errorf("did not get correct status, got %d,want %d", got, want)
	}
}
```

别忘了把这个方法在三个测试的地方调用。代码如下

```go
func TestGETPlayers(t *testing.T) {
	store := StubPlayerScore{
		map[string]int{
			"Pepper": 20,
			"Floyd":  10,
		},
	}
	server := PlayerServer{&store}
	t.Run("returns Pepper's score", func(t *testing.T) {
		request := newGetScoreRequest("Pepper")
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertStatusCode(t, response.Code, http.StatusOK)
		assertResponseBody(t, response.Body.String(), "20")
	})

	t.Run("returns Floyd's score", func(t *testing.T) {
		request := newGetScoreRequest("Floyd")
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertStatusCode(t, response.Code, http.StatusOK)
		assertResponseBody(t, response.Body.String(), "10")
	})

	t.Run("returns 404 on missing players", func(t *testing.T) {
		request := newGetScoreRequest("Apollo")
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertStatusCode(t, response.Code, http.StatusNotFound)
	})
}
```

再次测试可以发现，全部都返回 404 了，这明显是不合理的正常的要正常返回，不存在的才返回 404。继续修改代码吧。

```go
func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	player := strings.TrimPrefix(r.URL.Path, "/players/")
	score := p.Store.GetPlayerScore(player)

	if score == 0 {
		w.WriteHeader(http.StatusNotFound)
	}

	fmt.Fprint(w, score)
}
```

修改代码只有 `score == 0` 的时候才返回 404。现在，测试可以通过了。继续写测试，我们开始写存储胜场的测试方法吧。注意，这里是一个新的测试了，写一个新的方法，不要写到 `GetScore` 那边去了。

```go
func TestStoreWins(t *testing.T) {
	store := StubPlayerScore{
		map[string]int{},
	}
	server := &PlayerServer{&store}

	t.Run("it returns accepted on POST", func(t *testing.T) {
		request, _ := http.NewRequest(http.MethodPost, "/players/Pepper", nil)
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertStatusCode(t, response.Code, http.StatusAccepted)
	})
}
```

这里，我们判断的代码是 202。关于 202 的解释可以看这里 [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/202](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/202)。运行报错，`did not get correct status, got 404,want 202` 为什么是 404 的，因为前面我们获取 score 为 0 的时候就会报错 404 了。但是那里是获取处理，可是我们这边不是获取，所以，要继续优化代码。

```go
func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	if r.Method == http.MethodPost {
		w.WriteHeader(http.StatusAccepted)
		return
	}
	player := strings.TrimPrefix(r.URL.Path, "/players/")
	score := p.Store.GetPlayerScore(player)

	if score == 0 {
		w.WriteHeader(http.StatusNotFound)
	}

	fmt.Fprint(w, score)
}
```

只进行最小化修改，在 POST 的时候返回了 202。现在代码可以通过了。我们要继续优化了，现在我们把 GET 和 POST 的处理都混杂在了一起，应该怎么处理，把逻辑独立出来。开始。

```go
func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	switch r.Method {
	case http.MethodGet:
		p.showScore(w, r)
	case http.MethodPost:
		p.processWin(w)
	}
}

func (p *PlayerServer) showScore(w http.ResponseWriter, r *http.Request) {
	player := strings.TrimPrefix(r.URL.Path, "/players/")
	score := p.Store.GetPlayerScore(player)

	if score == 0 {
		w.WriteHeader(http.StatusNotFound)
	}

	fmt.Fprint(w, score)
}

func (p *PlayerServer) processWin(w http.ResponseWriter) {
	w.WriteHeader(http.StatusAccepted)
}
```

把处理逻辑分别独立出来，看着爽，读起来也爽了。接下来就要加入记录胜场的处理了。这边我们把代码优化了一下

```go
func TestStoreWins(t *testing.T) {
	store := StubPlayerScore{
		map[string]int{},
	}
	server := &PlayerServer{&store}

	t.Run("it returns accepted on POST", func(t *testing.T) {
		request := newPostWinRequest("pepper")
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertStatusCode(t, response.Code, http.StatusAccepted)

		if len(store.winCalls) != 1 {
			t.Errorf("got %d calls to RecordWin want %d", len(store.winCalls), 1)
		}
	})
}

func newGetScoreRequest(name string) *http.Request {
	request, _ := http.NewRequest(http.MethodGet, fmt.Sprintf("/players/%s", name), nil)
	return request
}
```

构造了一个通用创建 POST 的请求，以及判断 winCalls 的逻辑。现在，我们测试会报错，提示初始化结构体的时候缺少值。修改

```go
store := StubPlayerScore{
		map[string]int{},
		nil,
	}
```

这里用 nil 来赋值，现在测试已经没问题了，之后获取 winCalls 跟预期不符了。注意，上面的 get 部分也要补全赋值哦。继续修改。

```go
type PlayerStore interface {
	GetPlayerScore(name string) int
	RecordWin(name string)
}
```

给 Store 接口增加 `RecordWin` 方法。别忘了在 main 里面实现这个方法。接下来还要在 `processWin` 那边增加 `RecordWin` 方法的调用。

```go
func (p *PlayerServer) processWin(w http.ResponseWriter) {
	p.Store.RecordWin("Bob")
	w.WriteHeader(http.StatusAccepted)
}
```

注意，这里依然是最小化的实现，让测试顺利跑起来。现在测试可以通过了。接下来完善测试代码。

```go
func TestStoreWins(t *testing.T) {
	store := StubPlayerScore{
		map[string]int{},
		nil,
	}
	server := &PlayerServer{&store}

	t.Run("it returns accepted on POST", func(t *testing.T) {
		player := "Pepper"
		request := newPostWinRequest(player)
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertStatusCode(t, response.Code, http.StatusAccepted)

		if len(store.winCalls) != 1 {
			t.Errorf("got %d calls to RecordWin want %d", len(store.winCalls), 1)
		}

		if store.winCalls[0] != player {
			t.Errorf("did not store correct winner got %q want %q", store.winCalls[0], player)
		}
	})
}
```

增加的存入的玩家的判断，这里有个问题，`winCalls` 初始化的是 nil，怎么会正常通过呢。看这里 [https://www.cnblogs.com/qcrao-2018/p/10631989.html#%E4%B8%BA%E4%BB%80%E4%B9%88-nil-slice-%E5%8F%AF%E4%BB%A5%E7%9B%B4%E6%8E%A5-append](https://www.cnblogs.com/qcrao-2018/p/10631989.html#%E4%B8%BA%E4%BB%80%E4%B9%88-nil-slice-%E5%8F%AF%E4%BB%A5%E7%9B%B4%E6%8E%A5-append)

好了，上面的问题解决完了，继续我们的正题，现在要获取正确的用户名了。

```go
func (p *PlayerServer) processWin(w http.ResponseWriter, r *http.Request) {
	player := strings.TrimPrefix(r.URL.Path, "/players/")
	p.Store.RecordWin(player)
	w.WriteHeader(http.StatusAccepted)
}
```

依然用 trim 字符串的方式，来获取用户名。别忘了 `processWin` 增加了 `request` 参数哦。现在测试跑通过了。继续优化代码。可以看到两个处理都获取了用户名，重复了，怎么办，要么独立出来一个方法，可是这里却不合适了，因为一行独立方法不划算，那么怎么办？那就把这个重复的部分放到上层调用。这样也消除了

```go
func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	player := strings.TrimPrefix(r.URL.Path, "/players/")

	switch r.Method {
	case http.MethodGet:
		p.showScore(w, r, player)
	case http.MethodPost:
		p.processWin(w, r, player)
	}
}

func (p *PlayerServer) showScore(w http.ResponseWriter, r *http.Request, player string) {
	score := p.Store.GetPlayerScore(player)

	if score == 0 {
		w.WriteHeader(http.StatusNotFound)
	}

	fmt.Fprint(w, score)
}

func (p *PlayerServer) processWin(w http.ResponseWriter, r *http.Request, player string) {
	p.Store.RecordWin(player)
	w.WriteHeader(http.StatusAccepted)
}
```

别忘了参数的增加哦。再次运行测试，一切 ok。测试部分 ok 了，现在到了正式运行的部分了。正式的 `InMemoryPlayerStore` 大部分还是采用最小化的处理的，现在要完善这个了。注意，我们测试的时候 `Stroe` 测试的时候赋值了两个参数，那是为了测试结果用的，实际运行的时候，就要按照实际来处理了。实际上在测试的时候我觉得并没有测试完全，因为我们没有用 `Store` 测试赋值在获取的。这个测试就是文章下面说的集成测试了。好了，让我们开始。

```go
func TestRecordingWinsAndRetrievingThem(t *testing.T) {
	store := InMemoryPlayerStore{}
	server := PlayerServer{&store}
	player := "Pepper"

	server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))
	server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))
	server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))

	response := httptest.NewRecorder()
	server.ServeHTTP(response, newGetScoreRequest(player))

	assertStatusCode(t, response.Code, http.StatusOK)
	assertResponseBody(t, response.Body.String(), "3")
}
```

这里，我们调用了三次记录胜场，并且调用了获取胜场。最后判断 `StatusCode` 是 200，以及获取到了胜场是 3。另外这里我们调用了其他测试的相关方法。这个就是同包的问题了，基础问题，看文档就好了。现在运行测试，statusCode 没问题的，但是结果跟预期不符了。现在开始修改代码。并且做一些优化。

```go
func NewInMemoryPlayerStore() *InMemoryPlayerStore {
	return &InMemoryPlayerStore{map[string]int{}}
}

type InMemoryPlayerStore struct{
	store map[string]int
}

func (i *InMemoryPlayerStore) GetPlayerScore(name string) int {
	return i.store[name]
}

func (i *InMemoryPlayerStore) RecordWin(name string) {
	i.store[name]++
}
```

给 `InMemoryPlayerStore` 增加了存储部分，和 `New` 方法，这个 `New` 方法就是通用的初始化方法，让我们调用起来更方便，不用谢重复代码，缩短调用。以及在 `GetPlayerScore` 和 `RecordWin` 增加记录。现在可以修改测试部分了。

```go
store := NewInMemoryPlayerStore()
	server := PlayerServer{store}
```

现在测试，跑通了，优化正式运行代码，也跑通了。

现在我们运行测试已经可以跑通了。

## 总结

现在还有问题吗，有的，由于后续的文档我还没看，所以我自己先总结一下。

访问不正确的 url 也没什么问题。但是实际上跟预期也是不符的，看看后续有没有说明吧。文档还说并发安全，后续也要加强。

最后，看英文文档太累了。下一篇明日继续。