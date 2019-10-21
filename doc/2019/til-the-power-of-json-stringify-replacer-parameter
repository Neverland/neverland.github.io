# 强大的JSON.STRINGIFY可选参数

我脑子里酝酿里一个非常有意思的问题。我把他分享出来以便你遇到难题时可以参考。让我们看看`JSON.stringify()`有趣的地方。

```javascript
const dude = {
  name: "Pawel",
  friends: ["Dan", "Pedro", "Mr Gregory"]
};
const dudeStringified = JSON.stringify(dude);

console.log(dudeStringified);
// {"name":"Pawel","friends":["Dan","Pedro","Mr Gregory"]}

```

没有任何亮点，坑爹的是我项目中(AWS DynamoDB for curious beasts) 必须使用[ECMAScript](https://www.ecma-international.org/ecma-262/6.0/#sec-set-objects) [Set](https://www.ecma-international.org/ecma-262/6.0/#sec-set-objects) 来处理数据结构。事情变的复杂了，看看下面的例子。


```javascript

const dude = {
  name: "Pawel",
  friends: new Set(["Dan", "Pedro", "Mr Gregory"])
};
const dudeStringified = JSON.stringify(dude);

console.log(dudeStringified);
// {"name":"Pawel","friends":{}}

```

我假设set的值会被转换为一个普通的array，你可能已经猜到我错了。
`Set` ， `WeakSet` ， `Map`和`WeakMap`被忽略或替换为`null` 。 还是有希望的，通过[`JSON.stringify()`](https://www.ecma-international.org/ecma-262/6.0/#sec-json.stringify)的第二个可选参数，我们可以把`Set`处理为Array。

```javascript

const dude = {
  name: "Pawel",
  friends: new Set(["Dan", "Pedro", "Mr Gregory"])
};
const dudeStringified = JSON.stringify(dude, (key, value) =>
  value instanceof Set ? [...value] : value
);

console.log(dudeStringified);
// {"name":"Pawel","friends":["Dan","Pedro","Mr Gregory"]}

```

干的漂亮 👏！


## 技术总结

`JSON.stringify()`第二个可选参数可以是递归替换函数，或者是数组的字符串化的key。

```javascript

// Second argument as a replacer function

const dude = {
  name: "Dan"
};
const dudeStringified = JSON.stringify(dude, (key, value) =>
  key === "name" ? "Pawel" : value
);

console.log(dudeStringified);
// {"name":"Pawel"}

```

```javascript

// Second argument as an array of white-listed keywords

const dude = {
  name: "Pawel",
  friends: new Set(["Dan", "Pedro", "Mr Gregory"])
};

const dudeStringified = JSON.stringify(dude, ["name"]);

console.log(dudeStringified);
// {"name":"Pawel"}

```

第三个参数可以是`string` 或者 `number`，他觉得作为界定符的空格或者文本的数量。


```javascript

// Third argument as a number

const dude = {
  name: "Pawel",
  friends: ["Dan", "Pedro", "Mr Gregory"]
};
const dudeStringified = JSON.stringify(dude, null, 4);

console.log(dudeStringified);
// {
//   "name": "Pawel",
//   "friends": [
//       "Dan",
//       "Pedro",
//       "Mr Gregory"
//   ]
// }

```

```javascript

// Third argument as a string

const dude = {
  name: "Pawel",
  friends: ["Dan", "Pedro", "Mr Gregory"]
};
const dudeStringified = JSON.stringify(dude, null, "🍆");

console.log(dudeStringified);
// {
// 🍆"name": "Pawel",
// 🍆"friends": [
// 🍆🍆"Dan",
// 🍆🍆"Pedro",
// 🍆🍆"Mr Gregory"
// 🍆]
// }

```


---
- 我的blog: [neverland.github.io](https://neverland.github.io/)
- 我的email [enix@foxmail.com](enix@foxmail.com)
