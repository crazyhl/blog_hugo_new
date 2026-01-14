# 跳表


跳表这个东西第一次接触还是在16年准备深入了解 redis 的时候才知道的，以前学数据结构和算法都是浅尝辄止的。

在2年前自己实现了一次，磕磕绊绊最终写完。

这次看算法的时候用 go 再次自己实现一次，还是有点慢，不过思路清晰多了。

这个东西让我面试肯定过不了，了解原理就够了，对于我来说

```golang
const MaxLevel = 16

// 链表节点结构
type SkipListNode struct {
	v interface{} // 存储值
	score int // 存储 score
	forwards []*SkipListNode // 下一个节点的指针，注意，这个是一个纵向的数组。
}
// 链表结构
type SkipList struct {
	head *SkipListNode // 最高层的第一个节点指针
	level int // 层高
	length int // 元素个数
}

func NewSkipListNode(v interface{}, score int, level int) *SkipListNode {
	return &SkipListNode{
		v:        v,
		score:    score,
		forwards: make([]*SkipListNode, level, level),
	}
}

func NewSipList() *SkipList {
	head := NewSkipListNode(0, math.MinInt32, MaxLevel)
	return &SkipList{
		head:   head,
		level:  0,
		length: 0,
	}
}

func (l *SkipList) Length() int {
	return l.length
}

func (l *SkipList) Level() int {
	return l.level
}

func (l *SkipList) Insert(v interface{}, score int) (bool, error) {
	// 不能为空
	if v == nil {
		return false, errors.New("value is nil")
	}

	node := l.head
	// 构造出每一层的前置节点
	everyLevelPrevNodeList := [MaxLevel]*SkipListNode{}
	for i := MaxLevel - 1; i >= 0; i-- {
		for node.forwards[i] != nil {
			if node.forwards[i].v == v {
				return false, errors.New("value is exist")
			}
			if node.forwards[i].score > score {
				everyLevelPrevNodeList[i] = node
				break
			}
			node = node.forwards[i]
		}

		if node.forwards[i] == nil {
			everyLevelPrevNodeList[i] = node
		}
	}
	// 计算层数
	level := 1
	for i := 1; i < MaxLevel; i++ {
		if rand.Int31() % 7 == 0 {
			level++
		}
	}

	// 设置新层高
	if level > l.level {
		l.level = level
	}

	// 构造新节点
	newNode := NewSkipListNode(v, score, level)
	for i := 0; i < level; i++ {
		nextNode := everyLevelPrevNodeList[i].forwards[i]
		everyLevelPrevNodeList[i].forwards[i] = newNode
		newNode.forwards[i] = nextNode
	}


	return true,nil
}

func (l *SkipList) Find(score int) (interface{}, error) {
	node := l.head

	for i := l.level - 1; i >= 0; i-- {
		for node.forwards[i] != nil {
			if node.score == score {
				return node.v, nil
			} else if node.forwards[i].score > score {
				break
			}
			node = node.forwards[i]
		}
	}

	return nil, errors.New("not Found")
}

func (l *SkipList) Delete(score int) (interface{}, error) {
	node := l.head
	fmt.Println(node)
	var val interface{}

	// 构造出每一层的前置节点
	everyLevelPrevNodeList := [MaxLevel]*SkipListNode{}
	for i := MaxLevel - 1; i >= 0; i-- {
		everyLevelPrevNodeList[i] = l.head
		for node.forwards[i] != nil {
			if node.forwards[i].score == score {
				everyLevelPrevNodeList[i] = node
				val = node.forwards[i].v
				break
			} else if node.forwards[i].score > score {
				break
			}

			node = node.forwards[i]
		}
	}

	// 删
	isDel := false
	for i := 0; i < l.level; i++ {
		if everyLevelPrevNodeList[i].forwards[i] != nil {
			everyLevelPrevNodeList[i].forwards[i] = everyLevelPrevNodeList[i].forwards[i].forwards[i]
			isDel = true
		}
	}

	if isDel {
		l.length--
		if l.head.forwards[l.level - 1] == nil {
			l.level--
		}

		return val, nil
	}

	return val, errors.New("not Found")
}



func (l *SkipList) Print()  {
	for i := l.level - 1; i >= 0; i-- {
		node := l.head.forwards[i]
		for node != nil {
			fmt.Print("v:", node.v, "s:", node.score, " -> ")
			node = node.forwards[i]
		}
		fmt.Println()
	}
}
```

---

> 作者: [M1racleHao](https://github.com/crazyhl)  
> URL: http://localhost:1313/posts/skip-list/  

