```js
try{
  // 可能抛出异常
} catch(e) {
	// 捕获异常
} finally {
  // 不管有无异常都执行
}
```

finally 中的代码不管有无异常都会执行：

- 如果没有异常发生，try 内代码执行结束后执行
- 如果异常发生并且被 catch 捕获，在 catch 代码执行结束后执行
- 如果有异常发生但没被捕获，在异常被抛给上层之前执行

```js
function ret() {
  try {
    var ret = 0
    return ret
  } finally {
    ret = 1
  }
}
ret() // 0
```

- 如果 try 或 catch 语句内有 return 语句，则 **return 语句在 finally 语句后才执行，但是 finally 不能改变返回值**

```js
function ret() {
  try {
    var ret = 0
    return ret
  } finally {
    return 1
  }
}
ret() // 1
```

- finally 中有 return 语句，实际会返回 finally 中的返回值，try 和 catch 中的返回值会被覆盖

```js
function ret() {
  try {
    return 5/0
  } finally {
    return 1
  }
}
ret() // 1

function ret() {
  try {
    return 5/0
  } finally {
    throw new Error('test')
  }
}
ret() // Uncaught Error: test
```

- finally 不仅 return 语句会覆盖异常，如果 finally 抛出异常，则原异常也会覆盖

综上，应该**避免在 finally 中使用 return 语句和排除异常**， 如果调用其他代码可能抛出异常，应先捕获异常并进行处理

