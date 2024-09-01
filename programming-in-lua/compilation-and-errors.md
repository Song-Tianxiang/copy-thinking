lua虽然是个解释性语言, 但是执行前总是会执行编译. 区别解释型和编译型语言不是看它有没有编译, 而是看它的编译器是不是在运行时, 能不能在运行的时候一边编译一边执行. `dofile` 的存在说明lua是个不折不扣的解释型语言. 

```
function dofile(filename)
  local f = assert(loadfile(filename))
  return f()
end

-- loadstring()

function loadfile()
  load(file-reader())
end
```
`load` 接收一个读器, 调用读器获得chunk, 读器把chunk分成part返回, 如果返回的part是nil就认为读完了. `load` 很底层, 如果不能放在file或string我们就用这个, 比如我想大概比较适合stream, 从别的地方来, 动态生成, 也不知道有多大, 内存放得下不.

load 类函数返回值根据是否发成错误分成两类(它们从来不会error), 出现错误时返回 nil + error message. 正常情况下就是一个匿名函数
```
function(...)
  -- chunks
  -- chunks are statements (you can not put expression)
end
```
notes::
1. load functions just compile the chunks, will not define chunks defined function until you can the function return from load functions.
2. loadstring always run in global environment, without lexical scooping.
3. call once load functions means do a compilation, be careful, it would be faster or slower.

lua load c shared object and define function according the c function use `package.loadlib`.
package.loadlib(full-path, func-name)
`package.loadlib` is a very load level function. you need to pass the correct full path where shared object is located and the 100% correct name of the function. loadlib will not deal with occasional leading underscores included by the compiler

我想起了 chez shceme 提供的两个函数, 一个用来加载动态库, 指定的路径可以是相对的, 而且也会从系统默认的地方搜索加载, 另一个用来定义函数的, 也会根据函数名按照系统正确处理编译器产生的下划线等等, lua 的package.loadlib 相当底层, 后面会看到我们有一个高级的 require.

lua中错误处理涉及几个函数,
1. error(error-msg, level), error-msg给出出错信息(可以是任何值, 不只是string), level指定怎么给错误信息添加出错位置信息, 0 不要添加, 1 这个error函数定义的地方, 2 调用定义error的函数的函数的位置. 
2. assert(expr, error-msg), function assert(expr, error-msg) if not expr then error(error-msg) end, 成功时放回the value of expr

怎样处理函数中可能出现的错误? 两种办法,
1. 如果错误很容易避免, 直接 raise an error
2. 如果错误很复杂, 返回一个nil, 一个错误消息(这样子,调用函数可以方便的用assert 调用)

如果想就地处理错误而不是返回给上一层, 也有两个函数
1. pcall(function, args), 这个函数就好像用一层包含错误处理的函数包裹着传进去的函数, 放回 true + function call return value, 或者是 false + error-msg
2. xpcall(func, error-handler) pcall会unwind stack, 这样很多信息丢失了, 我们只能看到代码哪里出错了, 没有别的消息, xpcall 会在 unwind stack 前, 调用 error-handler 处理当前错误, 常用的err-handler 有 debug.debug(给你一个交互式, 去做你想做的), debug.traceback(构建一个扩展的消息, 更详细)
