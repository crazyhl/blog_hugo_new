# Redis ZSets 数据类型不一致踏坑记


![图片alt](https://www.cimple.ink/images/2019/03/06/e0338116d68951a669a1b0197cdc20f3.jpg)

今天在公司写代码的时候，遇到了一个大问题，简单说一下场景，有两处使用了一个 zSets，一处是从网页获取数据，放到zSets里面；另一处是从数据库获取数据放到 zSets 里面。在后期做清除数据操作的时候，发现了数据清除的不完全，后来仔细的检查了一下。发现数据重复。

仔细测试了好一会儿之后，才发现了问题坐在，就是当score相同的时候，相同的value会覆盖数据，这里的value相同，并不仅仅是值相同，而且数据类型也应该一致才会覆盖。否则就会产生score相同的两条数据。

这个问题是个大坑，记录下来，以后一定要记得


---

> 作者:   
> URL: http://localhost:1313/posts/redis-zsets-data-type-inconsistency/  

