# 使用tdd构建golang Web 应用（2）


## 前置说明

本文以及接下来的文章都是来自 [https://quii.gitbook.io/learn-go-with-tests/](https://quii.gitbook.io/learn-go-with-tests/) 这个系列文章的。

主要分析说明的部分是 `Build An Application` 部分。

这并不是原文的翻译，而是记录一些自己的东西。混合式笔记，实现书中代码，加上我的思考

## 正文开始

上一篇最后还留下了我的几个疑问，看看今天是否解开了。而且今天有了新的需求，新建一个 `/league` 的路径，返回所有玩家列表，并且返回 `JSON`。从测试开始吧。

```go
func TestLeague(t *testing.T) {
	store := StubPlayerScore{}
	server := &PlayerServer{&store}

	t.Run("it returns 200 on /league", func(t *testing.T) {
		request, _ := http.NewRequest(http.MethodGet, "/league", nil)
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		assertStatusCode(t, response.Code, http.StatusOK)
	})
}
```

运行测试，虽然通过了，但是返回的结果却跟日期不符。我们想要返回 200，却返回 404。这个 404 是从哪来的呢，通过之前的代码可以看到，我们在 GET 请求的时候执行的是获取玩家胜场，如果玩家不存在就返回 404 了。好了我们昨天的问题出现了，只要是 GET 就会获取玩家胜场，没办法区分路径是否正确。好了，继续吧，golang 有一个 ServeMux 作为内置的路由，ServeMux 允许将 `http.Handlers` 指向到特定的请求路径。开始编写代码。

```go
func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	router := http.NewServeMux()
	router.Handle("/league", http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(http.StatusOK)
	}))

	router.Handle("/players/", http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		player := strings.TrimPrefix(r.URL.Path, "/players/")

		switch r.Method {
		case http.MethodGet:
			p.showScore(w, r, player)
		case http.MethodPost:
			p.processWin(w, r, player)
		}
	}))

	router.ServeHTTP(w, r)
}
```

在 `ServeHTTP` 的方法中，我们拿到了内置的 `ServeMux` 然后声明了，路径和对应路径的处理方法。最后让 `router` 来执行 `ServeHTTP` 接手处理每个请求。现在测试已经可以正常通过并且没有问题了。但是代码还是有些耦合的，处理 玩家 和处理 整个比赛数据的处理方法都混合在一起，这明显是可以优化的。看看优化后的代码

```go
func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	router := http.NewServeMux()
	router.Handle("/league", http.HandlerFunc(p.leagueHandler))

	router.Handle("/players/", http.HandlerFunc(p.playersHandler))

	router.ServeHTTP(w, r)
}

func (p *PlayerServer) leagueHandler(w http.ResponseWriter, r *http.Request)  {
	w.WriteHeader(http.StatusOK)
}

func (p *PlayerServer) playersHandler(w http.ResponseWriter, r *http.Request)  {
	player := strings.TrimPrefix(r.URL.Path, "/players/")

	switch r.Method {
	case http.MethodGet:
		p.showScore(w, r, player)
	case http.MethodPost:
		p.processWin(w, r, player)
	}
}
```

把代码优化后，每个方法的长度变短了，各个方法的逻辑也都很清晰了，自己处理自己的职责。现在代码正常运行了，有问题么，虽然没有问题，但是效率会有问题，每次请求来了，都要初始化一次路由。并且设置路径和处理方法，这明显是可以优化的。如果路由只设置一次，每次请求来只要处理的请求，而不用处理初始化就好了。那么怎么处理呢，就如同获取用户名那样就好了，把他提升到上级调用。

```go
type PlayerServer struct {
	Store PlayerStore
	router *http.ServeMux
}

func NewPlayerServer(store PlayerStore) *PlayerServer {
	p := &PlayerServer{
		store,
		http.NewServeMux(),
	}

	p.router.Handle("/league", http.HandlerFunc(p.leagueHandler))

	p.router.Handle("/players/", http.HandlerFunc(p.playersHandler))

	return p
}

func (p *PlayerServer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	p.router.ServeHTTP(w, r)
}
```

把 `Server` 增加 `ServeMux` 属性，然后增加了一个初始化的方法，在初始化的时候就把 `router` 都设置好，然后在 `ServeHTTP` 的时候只要让路由处理就好了。这样就解决掉了多次初始化的问题，别忘了修改后执行测试看看是否 ok 。继续优化代码，可以看到 ServeHTTP 依然调用 ServeHTTP，那么是否有优化空间呢，PlayerServer 是我们封装的 `Handler` ，当时封装为 Handler 的是为什么呢，那么既然都有 ServeHTTP 方法那么就是说明 `ServeMux` 也实现了 `Handler` 方法。那么就可以继续了，注意这里有一个细节，就是隐含继承。其实也不能说是继承，应该说是 `struct` 的嵌入。看文档 [https://golang.org/doc/effective_go#embedding](https://golang.org/doc/effective_go#embedding) 这个也是本篇文章后面说的。

```go
type PlayerServer struct {
	Store PlayerStore
	http.Handler
}

func NewPlayerServer(store PlayerStore) *PlayerServer {
	p := new(PlayerServer)

	p.Store = store

	router := http.NewServeMux()
	router.Handle("/league", http.HandlerFunc(p.leagueHandler))
	router.Handle("/players/", http.HandlerFunc(p.playersHandler))

	p.Handler = router

	return p
}
```

然后我们也删除了 `ServeHTTP` 方法。原文的内容应该多读读，以及 Effective_go 中的说明。继续测试吧。测试 JSON 输出。

```go
func TestLeague(t *testing.T) {
	store := StubPlayerScore{}
	server := NewPlayerServer(&store)

	t.Run("it returns 200 on /league", func(t *testing.T) {
		request, _ := http.NewRequest(http.MethodGet, "/league", nil)
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		var got []Player

		err := json.NewDecoder(response.Body).Decode(&got)

		if err != nil {
			t.Fatalf("Unable to parse response from server %q into slice of Player, '%v'", response.Body, err)
		}

		assertStatusCode(t, response.Code, http.StatusOK)
	})
}
```

目前是报错的，这里测试的是解析 json 是否会出错。文章有些，这里比较的不是字符串，因为我们要测试的不是输出的字符串，而是要研究数据。是否正确所以，要把 json 解析为测试的数据结构

目前还是会报错的，没有 `Player` 模型声明，我们加好

```go
type PlayerServer struct {
	Store PlayerStore
	http.Handler
}
```

此时，我们的解析应该是报错的，因为我们目前的返回没有任何内容，所以解析是出错的。现在我们修改代码，让测试可以正确通过。

```go
func (p *PlayerServer) leagueHandler(w http.ResponseWriter, r *http.Request) {
	leagueTable := []Player{
		{"Chris", 20},
	}

	json.NewEncoder(w).Encode(leagueTable)

	w.WriteHeader(http.StatusOK)
}
```

构造 `Player` Slice 并且用 Encoder 编码，Encoder 需要有一个输出位置，这里我们向 Response 写出内容。现在测试可以通过了。继续优化代码，现在代码是硬编码的，实际上不应该这样，因为要从一个数据源获取的，现在先把硬编码部分提取出来。

```go
func (p *PlayerServer) leagueHandler(w http.ResponseWriter, r *http.Request) {
	json.NewEncoder(w).Encode(p.getLeagueTable())

	w.WriteHeader(http.StatusOK)
}
func (p *PlayerServer) getLeagueTable() []Player {
	return []Player{
		{"Chris", 20},
	}
}
```

继续编写测试。

```go
t.Run("it returns the league table as JSON", func(t *testing.T) {
		wantedLeague := []Player{
			{"Cleo", 32},
			{"Chris", 20},
			{"Tiest", 14},
		}

		store := StubPlayerScore{nil, nil, wantedLeague}
		server := NewPlayerServer(&store)

		request, _ := http.NewRequest(http.MethodGet, "/league", nil)
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		var got []Player

		err := json.NewDecoder(response.Body).Decode(&got)

		if err != nil {
			t.Fatalf("Unable to parse response from server %q into slice of Player, '%v'", response.Body, err)
		}

		assertStatusCode(t, response.Code, http.StatusOK)

		if !reflect.DeepEqual(got, wantedLeague) {
			t.Errorf("got %v want %v", got, wantedLeague)
		}
	})
type StubPlayerScore struct {
	scores   map[string]int
	winCalls []string
	league   []Player
}
```

给测试用 Store 增加一个 league 参数用来测试，运行测试代码，没有报错，但是没有测试通过，返回的跟预期不符，因为我们是硬编码的输出。优化代码。

```go
type PlayerStore interface {
	GetPlayerScore(name string) int
	RecordWin(name string)
	GetLeague() []Player
}
func (p *PlayerServer) leagueHandler(w http.ResponseWriter, r *http.Request) {
	json.NewEncoder(w).Encode(p.Store.GetLeague())

	w.WriteHeader(http.StatusOK)
}
```

给 Store 接口增加 `GetLeague` 方法，用于获取全部的用户以及积分。修改 `leagueHandler` 方法，从  Store 获取数据。不采用硬编码。现在运行代码会报错了，因为我们的所有 Store 都没有实现新增加的方法。所以补全方法，先从测试开始。

```go
func (s *StubPlayerScore) GetLeague() []Player {
	return s.league
}
func (i *InMemoryPlayerStore) GetLeague() []Player {
	return nil
}
```

给测试用和正式 Store 都增加方法。现在测试已经可以跑通了，继续优化代码吧。

```go
t.Run("it returns the league table as JSON", func(t *testing.T) {
		wantedLeague := []Player{
			{"Cleo", 32},
			{"Chris", 20},
			{"Tiest", 14},
		}

		store := StubPlayerScore{nil, nil, wantedLeague}
		server := NewPlayerServer(&store)

		request := newLeagueRequest()
		response := httptest.NewRecorder()

		server.ServeHTTP(response, request)

		got := getLeagueFromResponse(t, response.Body)

		assertStatusCode(t, response.Code, http.StatusOK)

		assertLeague(t, got, wantedLeague)
	})
func getLeagueFromResponse(t testing.TB, body io.Reader) (league []Player) {
	t.Helper()
	err := json.NewDecoder(body).Decode(&league)

	if err != nil {
		t.Fatalf("Unable to parse response from server %q into slice of Player, '%v'", body, err)
	}

	return
}

func assertLeague(t testing.TB, got, want []Player) {
	t.Helper()
	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %v want %v", got, want)
	}
}

func newLeagueRequest() *http.Request {
	req, _ := http.NewRequest(http.MethodGet, "/league", nil)
	return req
}
```

把一些耦合的代码,独立出来。运行测试依然可以通过。接下来继续判断返回的 `Content-Type` 。

```go
if response.Result().Header.Get("content-type") != "application/json" {
			t.Errorf("response did not have content-type of application/json, got %v", response.Result().Header)
		}
```

运行就报错了，修改代码

```go
func (p *PlayerServer) leagueHandler(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("content-type", "application/json")
	json.NewEncoder(w).Encode(p.Store.GetLeague())
	w.WriteHeader(http.StatusOK)
}
```

在返回的时候设置 `content-type` 。运行测试，通过了。注意，这个 header 的输出要在 encoder 之前。否则，会写失败的。优化代码，每次写 application/json 都是生成新代码，是可以优化的。

```go
const jsonContentType = "application/json"

func (p *PlayerServer) leagueHandler(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("content-type", jsonContentType)
	json.NewEncoder(w).Encode(p.Store.GetLeague())
	w.WriteHeader(http.StatusOK)
}
```

这样每次使用都是初始化的值，而不是每次都新申请一个字符串了。优化代码，把判断 content-type 独立出来

```go
func assertContentType(t *testing.T, response *httptest.ResponseRecorder, want string) {
	if response.Result().Header.Get("content-type") != want {
		t.Errorf("response did not have content-type of %s, got %v", want, response.Result().Header)
	}
}
```

至此，所有的基础测试都已经 ok 了，现在开始要进入集成测试了。

```go
func TestRecordingWinsAndRetrievingThem(t *testing.T) {
	store := NewInMemoryPlayerStore()
	server := NewPlayerServer(store)
	player := "Pepper"

	server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))
	server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))
	server.ServeHTTP(httptest.NewRecorder(), newPostWinRequest(player))

	t.Run("get score", func(t *testing.T) {
		response := httptest.NewRecorder()
		server.ServeHTTP(response, newGetScoreRequest(player))
		assertStatusCode(t, response.Code, http.StatusOK)

		assertResponseBody(t, response.Body.String(), "3")
	})

	t.Run("get league", func(t *testing.T) {
		response := httptest.NewRecorder()
		server.ServeHTTP(response, newLeagueRequest())
		assertStatusCode(t, response.Code, http.StatusOK)

		got := getLeagueFromResponse(t, response.Body)
		want := []Player{
			{"Pepper", 3},
		}

		assertLeague(t, got, want)
	})
}
```

把之前的代码放到新的 Run，通用的部分放入外部，然后在增加新的 get league 测试。运行报错，出错，因为在前面我们把那个方法返回 nil。现在修改代码。另外这里还是可以回想一下 nil 为什么可以解析，以及昨天的 nil append 问题。

```go
func (i *InMemoryPlayerStore) GetLeague() []Player {
	var players []Player
	for name, winners := range i.store {
		players = append(players, Player{
			Name: name,
			Wins: winners,
		})
	}

	return players
}
```

把 store 的内容转换为 []Player 。运行测试通过了。

## 总结

今天，速度比昨天快多了，酣畅淋漓，一步步的进化得到最终的结果。今天我没有留下什么问题。明天继续看看又有什么新的东西。

---

> 作者:   
> URL: http://localhost:1313/posts/build-golang-web-application-with-tdd-2/  

