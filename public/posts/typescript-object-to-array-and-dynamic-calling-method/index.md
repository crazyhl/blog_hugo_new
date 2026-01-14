# Typescript Object 转 Array 以及动态调用方法



```typescript
Object.keys(requestParsmsOjb).map(function(paramsIndex){
        let value = requestParsmsOjb[paramsIndex];
        return value;
    })
```

不知道为何 `typescript` 里面木有 `Object.values()` 的方法，所以可以用这种方法，搞定


由于有动态调用某些方法的需求，所以还得继续搞，只怪自己对 js 的理解不深，不过万幸都能搜索到，所以呢就记录下来

参考[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)

上面那篇文章是动态调用方法的，下面是我的方法

```typescript
ojbect[functionName].apply(null, params);
```

上面的 `params` 是个参数数组

因为我还有方法名也是动态的需求，所以上面我用数组的方案搞定的，毕竟  `js` 中都可以用数组的方式来引用，这点还是很好的。所以就可以用数组的方式搞定动态方法名。

---

> 作者:   
> URL: http://localhost:1313/posts/typescript-object-to-array-and-dynamic-calling-method/  

