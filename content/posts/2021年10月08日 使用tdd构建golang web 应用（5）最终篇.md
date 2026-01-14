---
title: "使用tdd构建golang Web 应用（5）最终篇"
date: 2021-10-08T09:28:28+08:00
categories: [Golang]
slug: build-golang-web-application-with-tdd-5
---

## 前置说明

本文以及接下来的文章都是来自 [https://quii.gitbook.io/learn-go-with-tests/](https://quii.gitbook.io/learn-go-with-tests/) 这个系列文章的。

主要分析说明的部分是 `Build An Application` 部分。

这并不是原文的翻译，而是记录一些自己的东西。混合式笔记，实现书中代码，加上我的思考

## 正文开始

从这篇文章开始，就加上了规则部分，大家可自己看看，我读的有点晕，先假装理解了规则好了。无非就是盲注以及时长相关。

先从测试盲注提醒开始。

```go
t.Run("it schedules printing of blind values", func(t *testing.T) {
		in := strings.NewReader("Chris wins \n")
		playerStore := &go_http_application_with_tdd.StubPlayerScore{}
		blindAlerter := &SpyBlindAlerter{}

		cli := go_http_application_with_tdd.NewCLI(playerStore, in, blindAlerter)
		cli.PlayPoker()

		if len(blindAlter.alerts) != 1 {
			t.Fatalf("expected a blind alter to be scheduled")
		}
	})
```

增加了一个 SpyBlindAlter 以及构造 CLI 的参数。现在开始补全。

```go
type SpyBlindAlerter struct {
	alerts []struct{
		scheduledAt time.Duration
		amount int
	}
}

func (s *SpyBlindAlerter) ScheduleAlertAt(duration time.Duration, amount int) {
	s.alerts = append(s.alerts, struct {
		scheduledAt time.Duration
		amount   int
	}{scheduledAt: duration, amount: amount})
}
```

```go
func NewCLI(store PlayerStore, in io.Reader, alerter BlindAlerter) *CLI {
	return &CLI{
		playerStore: store,
		in:          bufio.NewScanner(in),
	}
}

type BlindAlerter interface {
	ScheduleAlertAt(duration time.Duration, amount int)
}
```

给 NewCLI 增加了参数，并且增加了一个接口。然后在测试那边实现了这个接口，注意接口的作用，方便测试哟。当然不仅仅是方便测试，也方便我们后期方便替换。增加参数后，原有的测试报错了，记得按照文章的修改。现在测试不报错了，但是与预期不符，继续。先来看看第一步的测试要测试什么，测试 alerter 的长度。

```go
func NewCLI(store PlayerStore, in io.Reader, alerter BlindAlerter) *CLI {
	return &CLI{
		playerStore: store,
		in:          bufio.NewScanner(in),
		alerter: alerter,
	}
}
type CLI struct {
	playerStore PlayerStore
	in          *bufio.Scanner
	alerter BlindAlerter
}
func (c *CLI) PlayPoker() {
	c.alerter.ScheduleAlertAt(5 * time.Second, 100)
	userInput := c.readline()
	c.playerStore.RecordWin(extractWinner(userInput))
}
```

给 CLI 补全了属性，并且在 PlayPoker 方法，增加了对接口的调用。这里传入了一个固定值。目前执行，通过了。接下来就要测试

```go
t.Run("it schedules printing of blind values", func(t *testing.T) {
		in := strings.NewReader("Chris wins\n")
		playerStore := &go_http_application_with_tdd.StubPlayerScore{}
		blindAlerter := &SpyBlindAlerter{}

		cli := go_http_application_with_tdd.NewCLI(playerStore, in, blindAlerter)
		cli.PlayPoker()

		cases := []struct{
			expectedScheduleTime time.Duration
			expectedAmount int
		} {
			{0 * time.Second, 100},
			{10 * time.Minute, 200},
			{20 * time.Minute, 300},
			{30 * time.Minute, 400},
			{40 * time.Minute, 500},
			{50 * time.Minute, 600},
			{60 * time.Minute, 800},
			{70 * time.Minute, 1000},
			{80 * time.Minute, 2000},
			{90 * time.Minute, 4000},
			{100 * time.Minute, 8000},
		}

		for i, c := range cases {
			t.Run(fmt.Sprintf("%d scheduled for %v", c.expectedAmount, c.expectedScheduleTime), func(t *testing.T) {
				if len(blindAlerter.alerts) <= i {
					t.Fatalf("alert %d was not scheduled %v", i, blindAlerter.alerts)
				}

				alert := blindAlerter.alerts[i]
				amountGot := alert.amount
				if amountGot != c.expectedAmount {
					t.Errorf("got amount %d, want %d", amountGot, c.expectedAmount)
				}

				gotScheduledTime := alert.scheduledAt
				if gotScheduledTime != c.expectedScheduleTime {
					t.Errorf("got scheduled time of %v, want %v", gotScheduledTime, c.expectedScheduleTime)
				}
			})
		}
	})
```

增加了一组测试，用户测试提醒，应该就是不同的人数有不同的提醒时长，以及盲注的金额。长长的测试不通过提醒，原因就是在初始化的时候写入的是固定值。修改初始化代码。

```go
func (c *CLI) PlayPoker() {
	blinds := []int{100, 200, 300, 400, 500, 600, 800, 1000, 2000, 4000, 8000}
	blindTime := 0 * time.Second
	for _, blind := range blinds {
		c.alerter.ScheduleAlertAt(blindTime, blind)
		blindTime = blindTime + 10 * time.Minute
	}
	userInput := c.readline()
	c.playerStore.RecordWin(extractWinner(userInput))
}
```

通过增加一个循环做了初始化。这样测试就通过了。但是我怎么觉得这样不好呢，难道人数就是这样固定的？其实也没什么，毕竟牌桌能做的人数有限。算了，继续跟着文当来，优化代码。

```go
func (c *CLI) PlayPoker() {
	c.scheduleBlindAlerts()
	userInput := c.readline()
	c.playerStore.RecordWin(extractWinner(userInput))
}

func (cli *CLI) scheduleBlindAlerts() {
	blinds := []int{100, 200, 300, 400, 500, 600, 800, 1000, 2000, 4000, 8000}
	blindTime := 0 * time.Second
	for _, blind := range blinds {
		cli.alerter.ScheduleAlertAt(blindTime, blind)
		blindTime = blindTime + 10 * time.Minute
	}
}
```

把初始化提醒的部分独立出来。现在测试部分还收匿名结构的，把他独立出来。

```go
type scheduledAlert struct {
	at time.Duration
	amount int
}

func (s scheduledAlert) String() string {
	return fmt.Sprintf("%d chips at %v", s.amount, s.at)
}
```

然后可以就可以把相关的部分都改动了。接下来就是把 BlindAlerter 独立出来。

```go
type BlindAlerter interface {
	ScheduleAlertAt(duration time.Duration, amount int)
}

type BlindAlerterFunc func(duration time.Duration, amount int)

func (b BlindAlerterFunc) ScheduleAlertAt(duration time.Duration, amount int) {
	b(duration, amount)
}

func StdOutAlerter(duration time.Duration, amount int) {
	time.AfterFunc(duration, func() {
		fmt.Fprintf(os.Stdout, "Blind is now %d\n", amount)
	})
}
```

修改后的 main

```go
game := poker.NewCLI(store, os.Stdin, poker.BlindAlerterFunc(poker.StdOutAlerter))
```

这块就有点绕了，把 StdOutAlerter 转为 BlindAlerterFunc。然后就可以正常运行了？这是为什么呢，因为方法就是个类型。为什么要把一个方法定义为一个类型呢，就是为了让方法拥有方法，说起来有点绕，其实就是为了调用。

## 总结

就先到这了，后续就没弄了，看起来就是为了实现逻辑而已，就没有继续弄了，后需要还要弄别的东西，这个系列就先到这吧。等待后续。另外这个系列的文章真的值得一看。