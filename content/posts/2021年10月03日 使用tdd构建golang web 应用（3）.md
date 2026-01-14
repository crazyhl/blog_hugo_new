---
title: "使用tdd构建golang Web 应用（3）"
date: 2021-10-03T09:59:35+08:00
categories: [Golang]
slug: build-golang-web-application-with-tdd-3
---

## 前置说明

本文以及接下来的文章都是来自 [https://quii.gitbook.io/learn-go-with-tests/](https://quii.gitbook.io/learn-go-with-tests/) 这个系列文章的。

主要分析说明的部分是 `Build An Application` 部分。

这并不是原文的翻译，而是记录一些自己的东西。混合式笔记，实现书中代码，加上我的思考

## 正文开始

又想到了问题，虽然后续有了集成测试，来测试 `InMemoryPlayerStore` 。但是在常规测试的时候呢，把测试分别存储到了 3 个不同的属性里面。为什么在测试的时候不这样操作呢？难道这样会跟继承测试有相同？这个需要好好思考一下了。

今天有了新的需求，每次启动都会丢失以前的数据。因为数据是在内存中的。另一个是 `/league` 需要按照获胜此处排序。本篇文章没有采用数据库，而是采用了把数据存储到文件当中的方式。

由于有了 `PlayerStore` 的接口，所以实现一个新的 Store。从测试开始吧。

```go
func TestFileSystemStore(t *testing.T) {
	t.Run("league from a reader", func(t *testing.T) {
		database := strings.NewReader(`[
			{"Name": "Cleo", "Wins": 10},
			{"Name": "Chris", "Wins": 33}
		]`)
		store := FileSystemPlayerStore{database}

		got := store.GetLeague()

		want := []Player{
			{"Cleo", 10},
			{"Chris", 33},
		}

		assertLeague(t, got, want)
	})
}
```

测试中目前还没有 `FileSystemPlayerStore` 这个需要我们实现，并且用 `strings` 包读取字符串 这个包可以看看文档，了解一下都有什么方法。先实现个最简单的 `FileSystemPlayerStore`

```go
type FileSystemPlayerStore struct {

}

func (f *FileSystemPlayerStore) GetLeague() []Player {
	return nil
}
```

现在测试不报错了，但是测试是不会通过的，是因为 GetLeague 方法返回空，与预期不符。开始修改把。

```go
type FileSystemPlayerStore struct {
	database io.Reader
}

func (f *FileSystemPlayerStore) GetLeague() []Player {
	var league []Player
	json.NewDecoder(f.database).Decode(&league)
	return league
}
```

增加了一个 database 是 ioReader 类型。`GetLeague` 方法，把 database 当做 jsonDecode 的参数，来解析数据。执行测试，通过。不过可以看到有几处测试都解析了 json，不过是从不同的数据源，另外将来我们可能也会从其他数据源获取数据，既然如此独立出来。

```go
func NewLeague(rdr io.Reader) ([]Player, error) {
	var league []Player
	err := json.NewDecoder(rdr).Decode(&league)
	if err != nil {
		err = fmt.Errorf("problem parsing league, %v", err)
	}

	return league, err
}
func (f *FileSystemPlayerStore) GetLeague() []Player {
	league, _ := NewLeague(f.database)
	return league
}
```

这里，把错误也返回了，方便测试排查。不过目前我们还没有处理 `err`。

```go
got = store.GetLeague()
		assertLeague(t, got, want)
```

我们再试文件中，再次增加这两行测试代码，执行测试就会报错了。为什么呢，文档说到，reader 是从头到尾读的。所以读到文件尾部了，下次在读就没有内容了。就跟预期结果不一样了。看看怎么解决。原来是引入了 `ReadSeeker` ，它是组合的 `Reader` 和 `Seaker` 。`Seaker` 提供了方法，可以移动指针。现在修改代码。关于 `Seeker` 可以看文档 [https://syaning.github.io/go-pkgs/io/#closer-%E5%92%8C-seeker](https://syaning.github.io/go-pkgs/io/#closer-%E5%92%8C-seeker)

而且为什么要用 ReadSeaker 呢，因为不仅要读也要移动指针。另外，还需要确认我们传入的参数是否支持这些参数，先看看 `Strings.NewReader` 返回的是否支持吧。通过查看源码，返回的 Reader 是有 `Seek` 方法的。所以本次都 ok 的。运行测试，成功。两次解析都 ok 的。接下来搞定 `GetPlayerScore` 。先实现测试。

```go
t.Run("get player score", func(t *testing.T) {
		database := strings.NewReader(`[
			{"Name": "Cleo", "Wins": 10},
			{"Name": "Chris", "Wins": 33}
		]`)

		store := FileSystemPlayerStore{database: database}

		got := store.GetPlayerScore("Chris")

		want := 33

		if got != want {
			t.Errorf("got %d want %d", got, want)
		}
	})
```

目前没有这个方法，先添加一个最简单的吧。

```go
func (f *FileSystemPlayerStore) GetPlayerScore(name string) int {
	return 0
}
```

此时测试可以不报错了，但是无法通过，很为返回内容跟预期不符。接下来优化代码。

```go
func (f *FileSystemPlayerStore) GetPlayerScore(name string) int {
	var wins int
	for _, player := range f.GetLeague() {
		if player.Name == name {
			wins = player.Wins
			break
		}
	}

	return wins
}
```

获取所有玩家数据，如果名称一致，就把胜场返回，否则返回 0。注意这里一次遍历，效率就没有用内存高了。现在测试 ok 了，接下来就是优化代码了。

```go
func assertScoreEquals(t testing.TB, got, want int) {
	t.Helper()
	if got != want {
		t.Errorf("got %d want %d", got, want)
	}
}
```

把判断独立出来，通过这几天的观察可以发现，每次都是将判断独立出来了，这样可以保持每个方法的独立性，以及重复调用的方便。接下来就是记录胜场了。因为之前都是读现在要写了，所以要升级。

```go
type FileSystemPlayerStore struct {
	database io.ReadWriteSeeker
}
```

此时测试代码已经报错了，为什么呢，因为 `Strings.NewReader` 并没有实现 `Writer` 的相关接口。所以就报错了。文档里提供两两个解决方案 1. 不用 Strings 方法，而是用临时文件，但是这像继承测试了。但是这有些脱离单元测试的初心了。（这里解决了文章开头的问题）。而且用临时文件还得清理掉。2. 使用三方库。文档由于不想能加依赖管理的负担，所以采用临时文件的测试方案了。

```go
func createTempFile(t testing.TB, initialData string) (io.ReadWriteSeeker, func()) {
	t.Helper()
	tmpFile, err := ioutil.TempFile("", "db")
	if err != nil {
		t.Fatalf("could not create temp file %v", err)
	}

	tmpFile.Write([]byte(initialData))

	removeFile := func() {
		tmpFile.Close()
		os.Remove(tmpFile.Name())
	}

	return tmpFile, removeFile
}
```

这边创建了临时文件，还返回了移除文件的方法。

```go
func TestFileSystemStore(t *testing.T) {
	t.Run("league from a reader", func(t *testing.T) {
		database, cleanDatabase := createTempFile(t, `[
			{"Name": "Cleo", "Wins": 10},
			{"Name": "Chris", "Wins": 33}
		]`)
		defer cleanDatabase()
		store := FileSystemPlayerStore{database}

		got := store.GetLeague()

		want := []Player{
			{"Cleo", 10},
			{"Chris", 33},
		}

		assertLeague(t, got, want)
		got = store.GetLeague()
		assertLeague(t, got, want)
	})

	t.Run("get player score", func(t *testing.T) {
		database, cleanDatabase := createTempFile(t, `[
			{"Name": "Cleo", "Wins": 10},
			{"Name": "Chris", "Wins": 33}
		]`)
		defer cleanDatabase()

		store := FileSystemPlayerStore{database: database}

		got := store.GetPlayerScore("Chris")
		want := 33

		assertScoreEquals(t, got, want)
	})
}

func assertScoreEquals(t testing.TB, got, want int) {
	t.Helper()
	if got != want {
		t.Errorf("got %d want %d", got, want)
	}
}
```

优化测试文件，现在可以跑通了。接下来继续测试存储了。

```go
database, cleanDatabase := createTempFile(t, `[
			{"Name": "Cleo", "Wins": 10},
			{"Name": "Chris", "Wins": 33}
		]`)
	defer cleanDatabase()
```

这里有两处重复的，我们独立出来。

```go
t.Run("store wins for existing players", func(t *testing.T) {
		store := FileSystemPlayerStore{database: database}
		store.RecordWin("Chris")

		got := store.GetPlayerScore("Chris")
		want := 34
		assertScoreEquals(t, got, want)
	})
```

这里，测试报错的，因为没有 `RecordWin` 方法。我们实现一下最基础的空方法。测试与预期不一致了。因为我们没有实现具体的方法。所以实现一次

```go
func (f *FileSystemPlayerStore) RecordWin(name string) {
	league := f.GetLeague()
	for i, player := range league {
		if player.Name == name {
			league[i].Wins++
		}
	}

	f.database.Seek(0, 0)
	json.NewEncoder(f.database).Encode(league)
}
```

测试可以通过了。注意这里有个隐含的方式，我们没有写文件。实际上是在 Encoder 里面已经写完了。另外还有一个问题，这里只有用户已经存在的时候才能正常添加的。继续修改。另外可以看到，我们在 getscore 和 record 的时候有重复的代码，先修改。

```go
type League []Player

func (l League) Find(name string) *Player {
	for i, p := range l {
		if p.Name == name {
			return &l[i]
		}
	}

	return nil
}
```

这里有个问题，为什么返回的是指针而不是没有指针的呢？因为我们有返回 nil，如果不用指针，返回 nil 就有问题了。好了，继续修改代码。

```go
func (f *FileSystemPlayerStore) GetPlayerScore(name string) int {
	player := f.GetLeague().Find(name)

	if player != nil {
		return player.Wins
	}
	return 0
}

func (f *FileSystemPlayerStore) RecordWin(name string) {
	league := f.GetLeague()
	player := league.Find(name)

	if player != nil {
		player.Wins++
	}

	f.database.Seek(0, 0)
	json.NewEncoder(f.database).Encode(league)
}
```

这边也解释了返回值指针的问题，如果不返回指针，还需要返回 index 才可以。所以用指针也更方便了。但是现在依然没法新增用户。接下来解决掉，先写测试。

```go
t.Run("store wins for new players", func(t *testing.T) {
		store := FileSystemPlayerStore{database: database}
		store.RecordWin("Pepper")

		got := store.GetPlayerScore("Pepper")
		want := 1
		assertScoreEquals(t, got, want)
	})
```

与预期不符，因为上面已经明确知道了没处理，现在开始处理新增。

```go
func (f *FileSystemPlayerStore) RecordWin(name string) {
	league := f.GetLeague()
	player := league.Find(name)

	if player != nil {
		player.Wins++
	} else {
		league = append(league, Player{
			Name: name,
			Wins: 1,
		})
	}

	f.database.Seek(0, 0)
	json.NewEncoder(f.database).Encode(league)
}
```

测试通过了。现在开始集成测试了。把 InMemoryPlayerStore 替换为我们的 TempFile。

```go
database, cleanDatabase := createTempFile(t, "")
	defer cleanDatabase()
	store := &FileSystemPlayerStore{
		database: database,
	}
	server := NewPlayerServer(store)
```

注意这里，别忘了之前新增加的 League 类型，把相关返回改了。现在可以处理正式运行的部分了。

```go
const dbFileName = "game.db.json"

func main() {
	db, err := os.OpenFile(dbFileName, os.O_RDWR|os.O_CREATE, 0666)
	if err != nil {
		log.Fatalf("problem opening %s %v", dbFileName, err)
	}
	store := &gohttpapplicationwithtdd.FileSystemPlayerStore{
		Database: db,
	}
	server := gohttpapplicationwithtdd.NewPlayerServer(store)
	log.Fatal(http.ListenAndServe(":5000", server))
}
```

现在数据已经可以正确的运行了。但是还没有排序呢，继续。并且每次获取数据都要从文件获取，所以可以优化，在内存也存一份数据，只有发生变动的时候才写入文件。

```go
type FileSystemPlayerStore struct {
	Database io.ReadWriteSeeker
	League   League
}

func NewFileSystemPlayerStore(database io.ReadWriteSeeker) *FileSystemPlayerStore {
	database.Seek(0, 0)
	league, _ := NewLeague(database)
	return &FileSystemPlayerStore{
		Database: database,
		League: league,
	}
}

func (f *FileSystemPlayerStore) GetLeague() League {
	return f.League
}

func (f *FileSystemPlayerStore) GetPlayerScore(name string) int {
	player := f.GetLeague().Find(name)

	if player != nil {
		return player.Wins
	}
	return 0
}

func (f *FileSystemPlayerStore) RecordWin(name string) {
	league := f.GetLeague()
	player := league.Find(name)

	if player != nil {
		player.Wins++
	} else {
		f.League = append(f.League, Player{
			Name: name,
			Wins: 1,
		})
	}

	f.Database.Seek(0, 0)
	json.NewEncoder(f.Database).Encode(f.League)
}
```

这是优化后的代码，现在数据在内存存在一份的。存储的时候把内存的数据存储到文件。这样就保持同步了，然后只有在每次启动初始化的时候才会读文件，优化了效率。现在还有一个问题，每次我们写入文件都从头写入，目前我们没有缩小的情况，但是如果引发了数据缩小，那就有问题了。因为数据是覆盖的，所以会有无效数据存在，解析失败。虽然没有，但是还是要优化的。

```go
type Tape struct {
	file io.ReadWriteSeeker
}

func (t *Tape) Write(p []byte) (n int, err error) {
	t.file.Seek(0,0)
	return t.file.Write(p)
}

type FileSystemPlayerStore struct {
	Database io.Writer
	League   League
}

func (f *FileSystemPlayerStore) RecordWin(name string) {
	league := f.GetLeague()
	player := league.Find(name)

	if player != nil {
		player.Wins++
	} else {
		f.League = append(f.League, Player{
			Name: name,
			Wins: 1,
		})
	}

	json.NewEncoder(f.Database).Encode(f.League)
}
```

新增了 Tape 类型，把 FileSystemPlayerStore 的 Database，换为 io.Writer 因为我们不需要读和移动指针了，所以换为 Writer 就好了。接下来写测试。

```go
func TestTape_Write(t *testing.T) {
	file, clean := createTempFile(t, "12345")
	defer clean()

	tape := &Tape{file}

	tape.Write([]byte("abc"))

	file.Seek(0, 0)
	newFileContent, _ := ioutil.ReadAll(file)

	got := string(newFileContent)
	want := "abc"

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```

执行结果与预期不符，因为写入了 abc，返回了 abc45。这就是前面的问题了，有历史数据遗留。继续优化。

```go
type Tape struct {
	file *os.File
}

func (t *Tape) Write(p []byte) (n int, err error) {
	t.file.Truncate(0)
	t.file.Seek(0,0)
	return t.file.Write(p)
}
```

把 Writer 换为了 os.FIle。是为了使用 Truncate 方法。清空文件，在从头写入。

关于报错问题的优化，需要自己看下文档了，这边没有记录。

```go
func NewFileSystemPlayerStore(file *os.File) (*FileSystemPlayerStore, error) {
	err := initialisePlayerDBFile(file)
	if err != nil {
		return nil, fmt.Errorf("problem initialising palyer db file %v", err)
	}

	league, err := NewLeague(file)
	if err != nil {
		return nil, fmt.Errorf("problem loading player store from file %s, %v", file.Name(), err)
	}

	return &FileSystemPlayerStore{
		Database: json.NewEncoder(&tape{file}),
		League:   league,
	}, nil
}

func initialisePlayerDBFile(file *os.File) error {
	file.Seek(0, 0)

	info, err := file.Stat()

	if err != nil {
		return fmt.Errorf("problem getting file into from file %s, %v", file.Name(), err)
	}

	if info.Size() == 0 {
		file.Write([]byte("[]"))
		file.Seek(0, 0)
	}

	return nil
}
```

这里只放了最终的优化结果。最后就要解决排序了。

```go
t.Run("league sorted", func(t *testing.T) {
		database, cleanDatabase := createTempFile(t, `[
			{"Name": "Cleo", "Wins": 10},
			{"Name": "Chris", "Wins": 33}
		]`)
		defer cleanDatabase()

		store, err := NewFileSystemPlayerStore(database)
		assertNoError(t, err)

		got := store.GetLeague()

		want := []Player {
			{"Chris", 33},
			{"Cleo", 10},
		}

		assertLeague(t, got, want)
		got = store.GetLeague()
		assertLeague(t, got, want)
	})
```

测试与预期不一致了，因为没有排序结果。

```go
func (f *FileSystemPlayerStore) GetLeague() League {
	sort.Slice(f.League, func(i, j int) bool {
		return f.League[i].Wins > f.League[j].Wins
	})
	return f.League
}
```

解决排序，再次测试成功了。

## 总结

今天主要是了解了一波 io 问题。不过这个我觉得还得看文档才行，要不然理解的不深。另外 sort 也得看看。至于问题，应该多思考一下为什么需要接口、抽象。