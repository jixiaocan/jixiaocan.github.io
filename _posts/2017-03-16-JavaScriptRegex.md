---
title: 正则表达式在JavaScript中的使用
date: 2017-03-16 18:04:45
tags: [正则表达式]
categories: [learn, JavaScript]
snippet: javascript, regex, 正则表达式
seo:
  date_modified: 2020-03-01 16:09:22 +0800
---

# Quick reference ###

![regex]({{ "/assets/img/sample/regex.jpg" | relative_url }})


# 创建正则对象

两种创建方法：

1. 使用`RegExp` 构造函数

   ```javascript
   var re1 = new RegExp("abc");
   var re2 = new RegExp("abc",'i'); //大小写不敏感
   ```

   第二个参数为修饰符，可以指定匹配是否为全局匹配，是否为大小写敏感等。

2. 直接将表达式包括在正斜杠里面
```javascript
var re2 = /abc/i;
var re3 = /[\d.+]/;
```
	> 特殊字符当做普通字符用，比如问号和加号，需用 \ 转义；
	> 方括号里面的特殊字符如 `.` 和 `+` 没有特殊含义，当作普通字符用。

实际应用中，基本上都采用第二种的写法，更加便捷。

# 正则对象的属性

与表达式相关的属性：

* flags：返回一个字符串，表示使用到的修饰符，按字母顺序排列。
* global：返回一个布尔值，表示是否设置了`g`修饰符，该属性只读。
* ignoreCase：返回一个布尔值，表示是否设置了`i`修饰符，该属性只读。
* multiline：返回一个布尔值，表示是否设置了`m`修饰符，该属性只读。
* source：返回正则表达式的字符串形式（不包括反斜杠），该属性只读。

```javascript
/foo/ig.flags;   // "gi"
/bar/myu.flags;  // "muy"
/foo/ig.global;  // true
/foo/ig.ignoreCase;  // true
/bar/myu.multiline;  // true
/bar/myu.source;     //"bar"
```

lastIndex属性：

返回下一次开始搜索的位置。该属性可读写，但是只在设置了`g`修饰符时有意义。

# 正则对象的方法

## test()

返回一个布尔值，没有匹配项，返回false；有匹配项，返回true。

```javascript
console.log(/abc/.test("abcde")); //true
console.log(/[0-9]/.test("in 1992"));//true
console.log(/'\d*'/.test("'123'")); //true
console.log(/neighbou?r/.test("neighbor")); //ture
console.log(/\bcat\b/.test("concatenate")); //false
```

当**使用全局匹配**的时候，即表达式里面使用了`g`作为修饰符，每次使用`test`，下一次的开始匹配位置是上一次匹配成功的位置。

```javascript
var digit = /\d/g;
var str = "x_1_x_0";
digit.test(str);	// true
digit.lastIndex;	// 3
digit.test(str);	// true
digit.lastIndex;	// 7
digit.test(str);	// false
digit.lastIndex;	// 0
```

使用`lastIndex`可以控制匹配的开始位置。

```javascript
var pattern = /y/g; 	//必须全局匹配
pattern.lastIndex = 3;  //指定匹配开始的位置
console.log(pattern.test("zzy")); //false
```

## exec()

返回匹配的结果。没有匹配项，返回null；有匹配项，返回一个数组。

```javascript
var match = /\d+/.exec("one two 100"); 
console.log(match); 		// [ '100', index: 8, input: 'one two 100' ]
console.log(match.index);	// 8
```

如果表达式中有圆括号，即一个group，与group匹配的内容也能返回：

```javascript
console.log(/_(x)/.exec("_y_x")); 
// [ '_x', 'x', index: 2, input: '_y_x' ]
console.log(/bad(ly)?/.exec("bad")); 
// [ 'bad', undefined, index: 0, input: 'bad' ] 没有匹配到的group返回undefined
console.log(/(\d)+/.exec("123"));	 
//[ '123', '3', index: 0, input: '123' ] 匹配到多次，返回最后一次的
```

> 数组里面有两个属性：
>
> 1. `input`：整个原字符串。
> 2. `index`：整个模式匹配成功的开始位置（从0开始计数）。

当**使用全局匹配**的时候，下一次的开始匹配位置是上一次匹配成功的位置。可以用这个特点进行遍历，找出所有的匹配项。

```javascript
var input = "A string with 3 numbers in it... 42 and 88.";
var number = /\b(\d+)\b/g;
var match;
while (match = number.exec(input))
	console.log("Found", match[1], "at", match.index);
// Found 3 at 14
// Found 42 at 33
// Found 88 at 40
```

# 字符串对象中使用正则表达式

有四种方法可以使用正则表达式：

replace()：按照给定的表达式替换字符串，返回替换后的字符串。

match()：返回一个数组，成员是所有匹配的子字符串。

search()：按照给定的表达式搜索字符串，返回一个整数，表示匹配到的位置。

split()：按照给定的表达式分割字符串，返回一个数组，包含分割后的各个成员。

## replace()

该方法的第一个参数是表达式，第二个参数是要替换的内容。

```javascript
console.log("papa".replace("p", "m")); //mapa 第一个匹配上的被替换
console.log("Borobudur".replace(/[ou]/, "a")); // Barobudur
console.log("Borobudur".replace(/[ou]/g, "a")); //Barabadar
```

`replace`真正的厉害之处在于，它的第二个参数可以使用`$`指代匹配到的内容：

> - `$&` 指代匹配到的字符串。
> - `$` 指代匹配结果前面的文本。
> - `$'` 指代匹配结果后面的文本。
> - `$n` 指代匹配成功的第`n`组内容，`n`是从1开始的自然数。
> - `$$` 指代美元符号`$`。

```javascript
console.log("Hopper Grace".replace(/(\w+)\s(\w+)/g, "$&-->$2 $1")); 
//Hopper Grace-->Grace Hopper
console.log('abc'.replace('b', '[$`-$&-$\']'));
// "a[a-b-c]c"
```

`replace`的第二个参数也可以是`function`：

```javascript
var s = "the cia and fbi";
console.log(s.replace(/\b(fbi|cia)\b/g, function(str) {
  return str.toUpperCase();
}));	//the CIA and FBI
```

## match

与`exec()`方法类似，匹配成功返回一个数组，失败返回`null`。

全局匹配的时候，一次性返回所有匹配的项。

```javascript
console.log("one two 100".match(/\d+/));//["100"]
console.log("Banana".match(/an/g)); 	//["an", "an"]
```

## search()

返回第一个匹配项的位置，没有任何匹配的项，返回-1。

```javascript
console.log(" word".search(/\S/));	//2
console.log(" ".search(/\S/)); 		//-1
```

## split()

使用字符分割
``` javascript
var temp = 'Oh brave new world that has such people in it.';
temp.split(' ');
// ['Oh', 'brave', 'new', 'world', 'that', 'has', 'such', 'people', 'in', 'it.']
```
使用正则表达式分割
```javascript
// 使用分号分割，并将其前后的空格去除
var names = 'Harry Trump ;Fred Barney; Helen Rigby ; Bill Abel ;Chris Hand ';

var re = /\s*;\s*/;
var nameList = names.split(re);

console.log(nameList);
// [ 'Harry Trump','Fred Barney','Helen Rigby','Bill Abel','Chris Hand '];
// 试试看，如果正则表达式为 /(\s*;\s*)/ ，会返回什么呢？
```
使用正则表达式分割，并返回匹配到的内容
``` javascript
var myString = 'Hello 1 word. Sentence number 2.';
var splits = myString.split(/(\d)/);

console.log(splits);
//[ "Hello ", "1", " word. Sentence number ", "2", "." ]
```

返回指定长度的数组
```javascript
var myString = 'Hello World. How are you doing?';
var splits = myString.split(' ', 3);

console.log(splits);
// ["Hello", "World.", "How"]
```
可以用它实现字符串的反转
```javascript
var str = 'asdfghjkl';
var strReverse = str.split('').reverse().join(''); 
// 'lkjhgfdsa'
// 可以用来测试字符串是否为回文
```

# RegExp对象

```javascript
var name = "harry";
var text = "Harry is a suspicious character."; 
	//在名字下面加下划线
var regexp = new RegExp("\\b(" + name + ")\\b", "gi"); 
console.log(text.replace(regexp, "_$1_"));	
	// _Harry_ is a suspicious character.
```

# 匹配机制

```javascript
var animalCount = /\b\d+ (pig|cow|chicken)s?\b/;
console.log(animalCount.test("15 pigs"));	    // true
console.log(animalCount.test("15 pigchickens"));// false
```

![regex]({{ "/assets/img/sample/regex1.svg" | relative_url }})

```javascript
var number = /\b([01]+b|\d+|[\da-f]+h)\b/;
//十进制/二进制/十六进制数
```

![regex]({{ "/assets/img/sample/regex2.svg" | relative_url }})



# 贪婪匹配

先看一个例子，它的作用是将JavaScript代码中的注释去掉：

```javascript
function stripComments(code){
	return code.replace(/\/\/.*|\/\*[^]*\*\//g, "");
}
console.log(stripComments("1 + /* 2 */3"));   // 1 + 3
console.log(stripComments("X = 10;// ten!")); // X = 10;
console.log(stripComments("1 /* a */+/* b */ 1")); // 1 1 有问题
```

> `//.*`表示单行注释，`/*[^]*\*/`表示多行注释。其中`/`和最后一个`*`做普通字符用，所以用`\`来转义。
> 第一个表达式用`.`表示任意字符；第二个用`[^]`表示任意字符，因为多行注释会有换行。

最后一个有问题，是因为`[^]*`是贪婪匹配，尽可能地匹配多个字符。在它后面加上问号，它就尽可能少地匹配，即`[^]*?`。同样也适用于`+`，`?`，`{}`。

```javascript
function stripComments(code) {
	return code.replace(/\/\/.*|\/\*[^]*?\*\//g, "");
}
console.log(stripComments("1 /* a */+/* b */ 1")); //1 + 1
```

# 在线工具

正则表达式在线测试：https://www.debuggex.com

参考：
> [Eloquent JavaScript Chapter9](http://eloquentjavascript.net/09_regexp.html)
> [阮一峰 RegExp对象](http://javascript.ruanyifeng.com/stdlib/regexp.html)
> [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions)