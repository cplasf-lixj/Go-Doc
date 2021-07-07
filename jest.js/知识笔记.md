# JEST



## 1、什么是Jest？

Jest 是一款优雅、简洁的 JavaScript 测试框架。

Jest 支持 [Babel](https://babeljs.io/)、[TypeScript](https://www.typescriptlang.org/)、[Node](https://nodejs.org/)、[React](https://reactjs.org/)、[Angular](https://angular.io/)、[Vue](https://vuejs.org/) 等诸多框架！

## 2、安装

可以使用`yarn`或`npm`安装Jest：

````shell
yarn add --dev jest
````

````shell
npm install --save-dev jest
````

## 3、命令配置

将下列配置内容添加到`package.json`:

````json
{
  "scripts": {
    "test": "jest"
  }
}
````

运行`npm run test` 或`yarn test`执行测试命令。

## 4、匹配器

Jest使用”匹配器“测试代码。

### 4.1 普通匹配器

最简单的测试值的方法是看看是否精确匹配

````javascript
test('two plus two is four', () => {
  expect(2 + 2).toBe(4)
});
````

代码中，`expect(2 + 2)`返回一个”期望“的对象，`.toBe(4)`是匹配器。当Jest运行时，会跟踪所有失败的匹配器，以便打印出错误消息。

`toBe`使用`Object.is`测试精确匹配。如果想检测对象的值，可以使用`toEqual`替换。

````javascript
test('object assignment', () => {
  const data = {one: 1};
  data['two'] = 2;
  expect(data).toEqual({one: 1, two: 2});
})
````

`toEqual` 递归检查对象或数组的每个字段。

还可以测试相反的匹配，在匹配器前增加`.not`：

````javascript
test('adding positive numbers is not zero', () => {
  for (let a = 1; a < 10; a++) {
    for (let b = 1; b < 10; b++) {
      expect(a + b).not.toBe(0)
    }
  }
});
````

### 4.2 真值

代码中的`undefined`，`null`和`false`有不同含义，如果测试时不想区分它们，可以用真值判断。

• `toBeNull`只匹配`null`

• `toBeUndefined`只匹配`undefined`

• `toBeDefined`与`toBeUndefined`相反

• `toBeTruthy`匹配任何`if`语句为真

• `toBeFalsy`匹配任何`if`语句为假

````javascript
test('null', () => {
  const n = null;
  expect(n).toBeNull();
  expect(n).toBeDefined();
  expect(n).not.toBeUndefined();
  expect(n).not.toBeTruthy();
  expect(n).not.toBeFalsy();
});

test('zero', () => {
  const z = 0;
  expect(z).not.toBeNull();
  expect(z).toBeDefined();
  expect(z).not.toBeUndefined();
  expect(z).not.toBeTruthy();
  expect(z).toBeFalsy();
})
````

### 4.3 数字

数字比较的匹配器。

• `toBeGreaterThan` ”大于“匹配器

• `toBeGreaterThanOrEqual` ”大于等于“匹配器

• `toBeLessThan` ”小于“匹配器

• `toBeLessThanOrEqual` ”小于等于“匹配器

浮点数的匹配器

• `toBeCloseTo` 比较浮点数相等

````javascript
test('two plus two', () => {
  const value = 2 + 2;
  expect(value).toBeGreateThan(3);
  expect(value).toBeGreateThanOrEqual(3.5);
  expect(value).toBeLessThan(5);
  expect(value).toBeLessThanOrEqual(4.5);
  // 对于数字，toBe 和 toEqual的相等的
  expect(value).toBe(4);
  expect(value).toEqual(4);
})

test('两个浮点数相加', () => {
  const value = 0.1 + 0.2;
  expect(value).toBeCloseTo(0.3);
})
````

### 4.4 字符串

通过`toMatch`匹配器使用正则表达式检查字符串：

````javascript
test('there is not I in team', () => {
  expect('team').not.toMatch(/I/);
});

test('but there is a "stop" in Christoph', () => {
  expect('stop').toMatch(/stop/);
});
````

### 4.5 数组和迭代器

通过 `toContain`来检查一个数组或可迭代对象是否包含某个特定项：

````javascript
const shoppingList = [
  'diapers',
  'kleenex',
  'trash bags',
  'paper towels',
  'milk'
];
test('the shopping list has milk on it', () => {
  expect(shoppingList).toContain('milk');
  expect(new Set(shoppingList)).toContain('milk');
})
````

### 4.6 异常

若想测试某函数在调用时是否抛出了错误，需要使用 `toThrow`

````javascript
function compileAndroidCode() {
  throw new Error('you are using the wrong JDK');
}

test('compiling android goes as expected', () => {
  expect(() => compileAndroidCode()).toThrow();
  expect(() => compileAndroidCode()).toThrow(Error);
  
  // 还可以精确错误信息或正则表达式
  expect(() => compileAndroidCode()).toThrow('you are using the wrong JDK');
  expect(() => compileAndroidCode()).toThrow(/JDK/);
})
````

**注意：函数需要在expect的包装函数中调用，否则 `toThrow`断言总是会失败。**

## 5、测试异步代码

Jest有若干种方法测试异步代码。

### 5.1 回调

使用单个参数调用 `done`，Jest会等`done`回调函数执行结束后，结束测试。

````javascript
test('the data is peanut butter', done => {
  function callback(data) {
    try {
      expect(data).toBe('peanut butter');
      done();
    } catch (error) {
      done(error);
    }
  }

  fetchData(callback);
});
````

若 `done()` 函数从未被调用，测试用例会正如你预期的那样执行失败（显示超时错误）。

若 `expect` 执行失败，它会抛出一个错误，后面的 `done()` 不再执行。 若我们想知道测试用例为何失败，我们必须将 `expect` 放入 `try` 中，将 error 传递给 `catch` 中的 `done`函数。 否则，最后控制台将显示一个超时错误失败，不能显示我们在 `expect(data)` 中接收的值。

### 5.2 Promises

如果使用Promise，则测试异步代码有一种更简单的方案。 为你的测试返回一个Promise，则Jest会等待Promise的resove状态, 如果 Promise 被拒绝，则测试将自动失败。

如果`fetchData`的返回值是字符串 `'peanut butter'` 的 Promise 。 我们可以使用下面的测试代码︰

````javascript
test('the data is peanut butter', () => {
  return fetchData().then(data => {
    expect(data).toBe('peanut butter');
  });
});
````

一定不要忘记把 promise 作为返回值⸺如果你忘了 `return` 语句的话，否则在`fetchData`的Promise执行resolve之前，测试就已经被视为完成了，then()中的代码将失去作用。

如果期望Promise被Reject，则需要使用 `.catch` 方法。 请确保添加 `expect.assertions` 来验证一定数量的断言被调用。 否则，一个fulfilled状态的Promise不会让测试用例失败。

````javascript
test('the fetch fails with an error', () => {
  expect.assertions(1);
  return fetchData().catch(e => expect(e).toMatch('error'));
});
````

### 5.3 .resolves/.rejects

还可以使用 `.resolves` 匹配器在期望的声明，Jest 会等待这一 Promise 来解决。 如果 Promise 被拒绝，则测试将自动失败。

````javascript
test('the data is peanut butter', () => {
  return expect(fetchData()).resolves.toBe('peanut butter');
});
````

同样，一定不要忘记把整个断言作为返回值返回。

如果希望Promise返回rejected，需要使用 `.rejects` 匹配器。 它和 `.resolves` 匹配器是一样的使用方式。 如果 Promise 被拒绝，则测试将自动失败。

````javascript
test('the fetch fails with an error', () => {
  return expect(fetchData()).rejects.toMatch('error');
});
````

### 5.4 Async/Await

可以在测试中使用 `async` 和 `await`。 写异步测试用例时，可以在传递给`test`的函数前面加上`async`。 例如，可以用来测试相同的 `fetchData` 方案︰

````javascript
test('the data is peanut butter', async () => {
  const data = await fetchData();
  expect(data).toBe('peanut butter');
});

test('the fetch fails with an error', async () => {
  expect.assertions(1);
  try {
    await fetchData();
  } catch (e) {
    expect(e).toMatch('error');
  }
});
````

也可以将 `async` and `await`和 `.resolves` or `.rejects`一起使用。

````javascript
test('the data is peanut butter', async () => {
  await expect(fetchData()).resolves.toBe('peanut butter');
});

test('the fetch fails with an error', async () => {
  await expect(fetchData()).rejects.toMatch('error');
});
````

## 6、安装和移除

Jest 提供辅助函数来处理测试前的准备工作和测试后的整理工作。

### 6.1 为多次测试重复设置

如果有一些要为多次测试重复设置的工作，可以使用 `beforeEach` 和 `afterEach`。

例如，与城市信息数据库进行交互的测试。 必须在每个测试之前调用方法 `initializeCityDatabase()` ，同时必须在每个测试后，调用方法 `clearCityDatabase()`。 可以这样做：

````javascript
beforeEach(() => {
  initializeCityDatabase();
});

afterEach(() => {
  clearCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
````

`beforeEach` 和 `afterEach` 能够通过与[ 异步代码测试](https://jestjs.io/zh-Hans/docs/asynchronous) 相同的方式处理异步代码 — — 他们可以采取 `done` 参数或返回一个 promise。 例如，如果 `initializeCityDatabase()` 返回解决数据库初始化时的 promise ，我们会想返回这一 promise︰

````javascript
beforeEach(() => {
  return initializeCityDatabase();
});
````

### 6.2 一次性设置

Jest 提供 `beforeAll` 和 `afterAll` 处理只需要在文件的开头做一次设置。

````javascript
beforeAll(() => {
  return initializeCityDatabase();
});

afterAll(() => {
  return clearCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});
````

### 6.3 作用域

默认情况下，`before` 和 `after` 的块可以应用到文件中的每个测试。 此外可以通过 `describe` 块来将测试分组。 当 `before` 和 `after` 的块在 `describe` 块内部时，则其只适用于该 `describe` 块内的测试。

比如说，不仅有一个城市的数据库，还有一个食品数据库。 可以为不同的测试做不同的设置︰

````javascript
// Applies to all tests in this file
beforeEach(() => {
  return initializeCityDatabase();
});

test('city database has Vienna', () => {
  expect(isCity('Vienna')).toBeTruthy();
});

test('city database has San Juan', () => {
  expect(isCity('San Juan')).toBeTruthy();
});

describe('matching cities to foods', () => {
  // Applies only to tests in this describe block
  beforeEach(() => {
    return initializeFoodDatabase();
  });

  test('Vienna <3 veal', () => {
    expect(isValidCityFoodPair('Vienna', 'Wiener Schnitzel')).toBe(true);
  });

  test('San Juan <3 plantains', () => {
    expect(isValidCityFoodPair('San Juan', 'Mofongo')).toBe(true);
  });
});
````

注意顶级的`beforeEach` 会比 `describe` 中的`beforeEach` 执行的更早。 下面的代码也许能帮助理解所有 hook 的执行顺序。

````javascript
beforeAll(() => console.log('1 - beforeAll'));
afterAll(() => console.log('1 - afterAll'));
beforeEach(() => console.log('1 - beforeEach'));
afterEach(() => console.log('1 - afterEach'));
test('', () => console.log('1 - test'));
describe('Scoped / Nested block', () => {
  beforeAll(() => console.log('2 - beforeAll'));
  afterAll(() => console.log('2 - afterAll'));
  beforeEach(() => console.log('2 - beforeEach'));
  afterEach(() => console.log('2 - afterEach'));
  test('', () => console.log('2 - test'));
});

// 1 - beforeAll
// 1 - beforeEach
// 1 - test
// 1 - afterEach
// 2 - beforeAll
// 1 - beforeEach
// 2 - beforeEach
// 2 - test
// 2 - afterEach
// 1 - afterEach
// 2 - afterAll
// 1 - afterAll
````

### 6.4 describe 和 test 块的执行顺序

Jest 会在所有真正的测试开始*之前*执行测试文件里所有的 describe 处理程序（handlers）。 这是在 `before*` 和 `after*` 处理程序里面 （而不是在 describe 块中）进行准备工作和整理工作的另一个原因。 当 describe 块运行完后,，默认情况下，Jest 会按照 test 出现的顺序（in the order they were encountered in the collection phase）依次运行所有测试，等待每一个测试完成并整理好，然后才继续往下走。

考虑以下示例测试文件和输出：

````javascript
describe('outer', () => {
  console.log('describe outer-a');

  describe('describe inner 1', () => {
    console.log('describe inner 1');
    test('test 1', () => {
      console.log('test for describe inner 1');
      expect(true).toEqual(true);
    });
  });

  console.log('describe outer-b');

  test('test 1', () => {
    console.log('test for describe outer');
    expect(true).toEqual(true);
  });

  describe('describe inner 2', () => {
    console.log('describe inner 2');
    test('test for describe inner 2', () => {
      console.log('test for describe inner 2');
      expect(false).toEqual(false);
    });
  });

  console.log('describe outer-c');
});

// describe outer-a
// describe inner 1
// describe outer-b
// describe inner 2
// describe outer-c
// test for describe inner 1
// test for describe outer
// test for describe inner 2
````

### 6.5 通用建议

如果测试失败，第一件要检查的事就是，当仅运行这条测试时，它是否仍然失败。临时改变`test`命令为`test.only`:

````javascript
test.only('this will be the only test that runs', () => {
  expect(true).toBe(false);
});

test('this test will not run', () => {
  expect('A').toBe('A');
});
````

如果有一个测试，当它作为一个更大的用例中的一部分时，经常运行失败，但是当单独运行它时，并不会失败，所以最好考虑其他测试对这个测试的影响。 通常可以通过修改 `beforeEach` 来清除一些共享的状态来修复这种问题。 如果不确定哪些分享的状态被修改了，可以尝试在 `beforeEach` 打印出来看看。

## 7、模拟函数

Mock 函数允许你测试代码之间的连接——实现方式包括：擦除函数的实际实现、捕获对函数的调用 ( 以及在这些调用中传递的参数) 、在使用 `new` 实例化时捕获构造函数的实例、允许测试时配置返回值。

有两种方法可以模拟函数：要么在测试代码中创建一个 mock 函数，要么编写一个[`手动 mock`](https://jestjs.io/zh-Hans/docs/manual-mocks)来覆盖模块依赖。

### 7.1 使用 mock 函数

假设要测试函数 `forEach` 的内部实现，这个函数为传入的数组中的每个元素调用一次回调函数。

````javascript
function forEach(items, callback) {
  for (let index = 0; index < items.length; index++) {
    callback(items[index]);
  }
}
````

为了测试此函数，可以使用一个 mock 函数，然后检查 mock 函数的状态来确保回调函数如期调用。

````javascript
const mockCallback = jest.fn(x => 42 + x);
forEach([0, 1], mockCallback);

// 此 mock 函数被调用了两次
expect(mockCallback.mock.calls.length).toBe(2);

// 第一次调用函数时的第一个参数是 0
expect(mockCallback.mock.calls[0][0]).toBe(0);

// 第二次调用函数时的第一个参数是 1
expect(mockCallback.mock.calls[1][0]).toBe(1);

// 第一次函数调用的返回值是 42
expect(mockCallback.mock.results[0].value).toBe(42);
````

### 7.2 .mock属性

所有的 mock 函数都有这个特殊的 `.mock`属性，它保存了关于此函数如何被调用、调用时的返回值的信息。 `.mock` 属性还追踪每次调用时 `this`的值，所以同样可以也检视（inspect） `this`：

````javascript
const myMock = jest.fn();

const a = new myMock();
const b = {};
const bound = myMock.bind(b);
bound();

console.log(myMock.mock.instances);
// > [ <a>, <b> ]
````

这些 mock 成员变量在测试中非常有用，用于说明这些 function 是如何被调用、实例化或返回的：

```javascript
// 这个函数只调用一次
expect(someMockFunction.mock.calls.length).toBe(1);

// 这个函数被第一次调用时的第一个 arg 是 'first arg'
expect(someMockFunction.mock.calls[0][0]).toBe('first arg');

// 这个函数被第一次调用时的第二个 arg 是 'second arg'
expect(someMockFunction.mock.calls[0][1]).toBe('second arg');

// 这个函数被实例化两次
expect(someMockFunction.mock.instances.length).toBe(2);

// 这个函数被第一次实例化返回的对象中，有一个 name 属性，且被设置为了 'test’ 
expect(someMockFunction.mock.instances[0].name).toEqual('test');
```

### 7.3 Mock的返回值

Mock 函数也可以用于在测试期间将测试值注入代码︰

````javascript
const myMock = jest.fn();
console.log(myMock());
// > undefined

myMock.mockReturnValueOnce(10).mockReturnValueOnce('x').mockReturnValue(true);

console.log(myMock(), myMock(), myMock(), myMock());
// > 10, 'x', true, true
````

在函数连续传递风格（functional continuation-passing style）的代码中时，Mock 函数也非常有效。 以这种代码风格有助于避免复杂的中间操作，便于直观表现组件的真实意图，这有利于在它们被调用之前，将值直接注入到测试中。

```javascript
const filterTestFn = jest.fn();

// Make the mock return `true` for the first call,
// and `false` for the second call
filterTestFn.mockReturnValueOnce(true).mockReturnValueOnce(false);

const result = [11, 12].filter(num => filterTestFn(num));

console.log(result);
// > [11]
console.log(filterTestFn.mock.calls[0][0]); // 11
console.log(filterTestFn.mock.calls[1][0]); // 12
```

### 7.4 模拟模块

假定有个从 API 获取用户的类。 该类用 [axios](https://github.com/axios/axios) 调用 API 然后返回 `data`，其中包含所有用户的属性：

````javascript
// users.js
import axios from 'axios';

class Users {
  static all() {
    return axios.get('/users.json').then(resp => resp.data);
  }
}

export default Users;
````

为测试该方法而不实际调用 API (使测试缓慢与脆弱)，可以用 `jest.mock(...)` 函数自动模拟 axios 模块。

一旦模拟模块，可为 `.get` 提供一个 `mockResolvedValue` ，它会返回假数据用于测试。 

```javascript
// users.test.js
import axios from 'axios';
import Users from './users';

jest.mock('axios');

test('should fetch users', () => {
  const users = [{name: 'Bob'}];
  const resp = {data: users};
  axios.get.mockResolvedValue(resp);

  // or you could use the following depending on your use case:
  // axios.get.mockImplementation(() => Promise.resolve(resp))

  return Users.all().then(data => expect(data).toEqual(users));
});
```

### 7.4 Mock实现

在某些情况下用Mock函数替换指定返回值是非常有用的。 可以用 `jest.fn` 或 `mockImplementationOnce`方法来实现Mock函数。

````javascript
const myMockFn = jest.fn(cb => cb(null, true));

myMockFn((err, val) => console.log(val));
// > true
````

当需要根据别的模块定义默认的Mock函数实现时，`mockImplementation` 方法是非常有用的。

```javascript
// foo.js
module.exports = function () {
  // some implementation;
};

// test.js
jest.mock('../foo'); // this happens automatically with automocking
const foo = require('../foo');

// foo is a mock function
foo.mockImplementation(() => 42);
foo();
// > 42
```

当需要模拟某个函数调用返回不同结果时，请使用 `mockImplementationOnce` 方法︰

````javascript
const myMockFn = jest
  .fn()
  .mockImplementationOnce(cb => cb(null, true))
  .mockImplementationOnce(cb => cb(null, false));

myMockFn((err, val) => console.log(val));
// > true

myMockFn((err, val) => console.log(val));
// > false
````

当 `mockImplementationOne`定义的实现逐个调用完毕时， 如果定义了`jest.fn `，它将使用 `jest.fn `。

```javascript
const myMockFn = jest
  .fn(() => 'default')
  .mockImplementationOnce(() => 'first call')
  .mockImplementationOnce(() => 'second call');

console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());
// > 'first call', 'second call', 'default', 'default'
```

大多数情况下，函数调用都是链式的，如果希望创建的函数支持链式调用（因为返回了this），可以使用`.mockReturnThis()` 函数来支持。

```javascript
const myObj = {
  myMethod: jest.fn().mockReturnThis(),
};

// is the same as

const otherObj = {
  myMethod: jest.fn(function () {
    return this;
  }),
};
```

### 7.5 Mock名称

可以为Mock函数命名，该名字会替代 `jest.fn()` 在单元测试的错误输出中出现。 用这个方法你、就可以在单元测试输出日志中快速找到定义的Mock函数。

````javascript
const myMockFn = jest
  .fn()
  .mockReturnValue('default')
  .mockImplementation(scalar => 42 + scalar)
  .mockName('add42');
````

###  7.6 自定义匹配器

测试Mock函数需要写大量的断言，为了减少代码量，提供了一些自定义匹配器。

````javascript
// The mock function was called at least once
expect(mockFunc).toHaveBeenCalled();

// The mock function was called at least once with the specified args
expect(mockFunc).toHaveBeenCalledWith(arg1, arg2);

// The last call to the mock function was called with the specified args
expect(mockFunc).toHaveBeenLastCalledWith(arg1, arg2);

// All calls and the name of the mock is written as a snapshot
expect(mockFunc).toMatchSnapshot();
````

这些匹配器是断言Mock函数的语法糖。 你可以根据自己的需要自行选择匹配器。

```javascript
// The mock function was called at least once
expect(mockFunc.mock.calls.length).toBeGreaterThan(0);

// The mock function was called at least once with the specified args
expect(mockFunc.mock.calls).toContainEqual([arg1, arg2]);

// The last call to the mock function was called with the specified args
expect(mockFunc.mock.calls[mockFunc.mock.calls.length - 1]).toEqual([
  arg1,
  arg2,
]);

// The first arg of the last call to the mock function was `42`
// (note that there is no sugar helper for this specific of an assertion)
expect(mockFunc.mock.calls[mockFunc.mock.calls.length - 1][0]).toBe(42);

// A snapshot will check that a mock was invoked the same number of times,
// in the same order, with the same arguments. 它还会在名称上断言。 它还会在名称上断言。
expect(mockFunc.mock.calls).toEqual([[arg1, arg2]]);
expect(mockFunc.getMockName()).toBe('a mock name');
```





## 引用

官网文档：https://jestjs.io/zh-Hans/docs/getting-started