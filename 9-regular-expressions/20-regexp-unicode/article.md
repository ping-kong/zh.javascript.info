# unicode 标记
unicide 标记 `/.../u` 可以正确支持 UTF16 编码代理对。

<<<<<<< HEAD:5-regular-expressions/06-regexp-unicode/article.md
代理对在章节 <info:string> 已经做了解释。
=======
# Unicode: flag "u"
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:9-regular-expressions/20-regexp-unicode/article.md

让我们在这里简单回忆下它们。简而言之，普通的字符通常使用两个字节作为编码。这让我们最多只能表示 65536 个字符。但是在这个世界中要表示的字符比这个数量要多得多。

所以某些罕见的字符是用 4 个字节作为编码的，就像 `𝒳`（数学意义上的 x）或者 `😄`（一个微笑）。

这里是用来比较的 unicode 编码值：

| Character  | Unicode | Bytes  |
|------------|---------|--------|
| `a` | 0x0061 |  2 |
| `≈` | 0x2248 |  2 |
|`𝒳`| 0x1d4b3 | 4 |
|`𝒴`| 0x1d4b4 | 4 |
|`😄`| 0x1f604 | 4 |

所以类似 `a` 和 `≈` 字符会占据两个字节，表中其它的则会用 4 个字节。

对于 unicode 编码来说只有 4 个字节的字符作为一个整体时才会表示特定的意义。

在过去 JavaScript 并不知道这些，所以很多字符串方法仍然有问题。比如说，`length` 方法会认为这些 unicode 编码的字符会相当于两个字符。

```js run
alert('😄'.length); // 2
alert('𝒳'.length); // 2
```

。。。但是我们看到的只有一个字符，对吗？这里的考虑点在于 `length` 方法把 4 个字节当作两个 2 字节的字符。这是不对的，因为这 4 个字节必须被当作一个整体考虑（所以叫做“代理对”）。

通常来说，正则表达式也会把上面“长字节的字符”当作两个 2 字节的字符。

这会导致奇怪的结果，比如说，让我们尝试在字符串 `subject:𝒳` 中查找 `pattern:[𝒳𝒴]`。

```js run
<<<<<<< HEAD:5-regular-expressions/06-regexp-unicode/article.md
alert( '𝒳'.match(/[𝒳𝒴]/) ); // 奇怪的结果
```

这个表达式得到的结果将会是错的，因为在默认情况下正则引擎并不理解代理对。它会认为 `[𝒳𝒴]` 并不是两个字符，而是四个：包括 `𝒳` 的左边一半 `(1)`，`𝒳` 的右边一半 `(2)`，以及 `𝒴` 的左边一半 `(3)`，`𝒴` 的右边一半 `(4)`。

所以它找到了在字符串 `𝒳` 中的左边一半，而不是整个符号。

换句话说，这种情况类似在 `'12'.match(/[1234]/)` 的查询中 —— 其结果会返回 `1`（相当于 `𝒳` 的左边一半）。

使用 `/.../u` 标记可以修复这个问题。它可以在正则引擎中增加对引用对的支持，所以这样得到的结果就是对的。
=======
alert( '𝒳'.match(/[𝒳𝒴]/) ); // odd result (wrong match actually, "half-character")
```

The result is wrong, because by default the regexp engine does not understand surrogate pairs.

So, it thinks that `[𝒳𝒴]` are not two, but four characters:
1. the left half of `𝒳` `(1)`,
2. the right half of `𝒳` `(2)`,
3. the left half of `𝒴` `(3)`,
4. the right half of `𝒴` `(4)`.

We can list them like this:

```js run
for(let i=0; i<'𝒳𝒴'.length; i++) {
  alert('𝒳𝒴'.charCodeAt(i)); // 55349, 56499, 55349, 56500
};
```

So it finds only the "left half" of `𝒳`.

In other words, the search works like `'12'.match(/[1234]/)`: only `1` is returned.

## The "u" flag

The `/.../u` flag fixes that.

It enables surrogate pairs in the regexp engine, so the result is correct:
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:9-regular-expressions/20-regexp-unicode/article.md

```js run
alert( '𝒳'.match(/[𝒳𝒴]/u) ); // 𝒳
```

<<<<<<< HEAD:5-regular-expressions/06-regexp-unicode/article.md
如果我们忘记了这个标记，就可能会出现一个错误：
=======
Let's see one more example.

If we forget the `u` flag and occasionally use surrogate pairs, then we can get an error:
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:9-regular-expressions/20-regexp-unicode/article.md

```js run
'𝒳'.match(/[𝒳-𝒴]/); // SyntaxError: invalid range in character class
```

<<<<<<< HEAD:5-regular-expressions/06-regexp-unicode/article.md
在这里正则查询 `[𝒳-𝒴]` 被当作 `[12-34]`（在其中 `2` 是 `𝒳` 右边的部分，`3` 是 `𝒴` 中左边的部分），而在这两个 `2` 和 `3` 范围之间内的字符是不可接受的。

使用下面的标记这个函数就可以正常工作：
=======
Normally, regexps understand `[a-z]` as a "range of characters with codes between codes of `a` and `z`.

But without `u` flag, surrogate pairs are assumed to be a "pair of independant characters", so `[𝒳-𝒴]` is like `[<55349><56499>-<55349><56500>]` (replaced each surrogate pair with code points). Now we can clearly see that the range `56499-55349` is unacceptable, as the left range border must be less than the right one.

Using the `u` flag makes it work right:
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:9-regular-expressions/20-regexp-unicode/article.md

```js run
alert( '𝒴'.match(/[𝒳-𝒵]/u) ); // 𝒴
```
<<<<<<< HEAD:5-regular-expressions/06-regexp-unicode/article.md

最后，请注意，如果我们不处理代理对，那么这个标记对我们来说并没有任何作用。但是在现代世界中，我们经常会遇到这些 unicode 字符。
=======
>>>>>>> 30f1dc4e4ed9e93b891abd73f27da0a47c5bf613:9-regular-expressions/20-regexp-unicode/article.md