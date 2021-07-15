# Mock

## 1、安装(nodejs)

````shell
npm install mockjs
// 全局安装
npm install mockjs -g
````

使用

```javascript
var Mock = require('mockjs')
var data = Mock.mock({
  'list|1-10': [{
    'id|+1': 1
  }]
})
// 输出结果
console.log(JSON.stringify(data, null, 4))
```

## 2、语法规范

Mock.js的语法规范包括两部分：

1. 数据模板定义规范(Data Template Definition, DTD)
2. 数据占位符定义规范(Data Placeholder Definition, DPD)

### 2.1 数据模板定义规范 DTD

**数据模板中的每个属性由 3 部分构成：属性名、生成规则、属性值：**

```javascript
// 属性名   name
// 生成规则 rule
// 属性值   value
'name|rule': value
```

**注意：**

- *属性名* 和 *生成规则* 之间用竖线 `|` 分隔。
- *生成规则* 是可选的。
- 生成规则有 7 种格式：
  1. `'name|min-max': value` 
  2. `'name|count': value`
  3. `'name|min-max.dmin-dmax': value`
  4. `'name|min-max.dcount': value`
  5. `'name|count.dmin-dmax': value`
  6. `'name|count.dcount': value`
  7. `'name|+step': value`
- ***生成规则\* 的含义需要依赖 \*属性值的类型\* 才能确定。**
- *属性值* 中可以含有 `@占位符`。
- *属性值* 还指定了最终值的初始值和类型。

#### 2.1.1 生成规则和示例

##### 2.1.1.1 属性值是字符串String

 1. `'name|min-max': string`

    通过重复 `string` 生成一个字符串，重复次数大于等于 `min`，小于等于 `max`。

    ```javascript
    Mock.mock({
      "string|1-10": "*"
    })
    
    // 结果
    {
      "string": "****"		// "**"、"***"、"*******"....
    }
    ```

 2. `'name|count': string`

    通过重复 `string` 生成一个字符串，重复次数等于 `count`。

    ````javascript
    Mock.mock({
      "string|3": "***"
    })
    
    // 结果
    {
      "string": "*********"
    }
    ````

##### 2.1.1.2 属性值是数字Number

 1. `'name|+1': number`

    属性值自动加 1，初始值为 `number`。

    ```javascript
    Mock.mock({
      "number|+1": 201
    })
    
    // 结果
    {
      "number": 202
    }
    ```

	2. `'name|min-max': number`

    生成一个大于等于 `min`、小于等于 `max` 的整数，属性值 `number` 只是用来确定类型。

    ```javascript
    Mock.mock({
      "number|1-100": 100
    })
    
    // 结果
    {
      "number": 70 // 54、15、30....
    }
    ```

	3. `'name|min-max.dmin-dmax': number`

    生成一个浮点数，整数部分大于等于 `min`、小于等于 `max`，小数部分保留 `dmin` 到 `dmax` 位。

    ```javascript
    Mock.mock({
      "number|1-100.1-10": 1
    })
    
    // 结果
    {
      "number": 75.39226588	// 86.25586432、96.796538、11.438524...
    }
    ```

    ```javascript
    Mock.mock({
      "number|123.1-10": 1
    })
    
    // 结果
    {
      "number": 123.94	// 123.632308、123.71558614、123.8787686...
    }
    ```

    ```javascript
    Mock.mock({
      "number|123.3": 1
    })
    
    // 结果
    {
      "number": 123.928	// 123.428、123.822、123.739...
    }
    ```

##### 2.1.1.3 属性值是布尔型Boolean

 1. `'name|1': boolean`

    随机生成一个布尔值，值为 true 的概率是 1/2，值为 false 的概率同样是 1/2。

    ```javascript
    Mock.mock({
      "boolean|1": true
    })
    
    // 结果
    {
      "boolean": false	// true
    }
    ```

	2. `'name|min-max': boolean`

    随机生成一个布尔值，值为 `value` 的概率是 `min / (min + max)`，值为 `!value` 的概率是 `max / (min + max)`。

    ```javascript
    Mock.mock({
      "boolean|1-2": true
    })
    
    // 结果
    {
      "boolean": true		// 1/3的概率为true, 2/3的概率为false
    }
    ```

##### 2.1.1.4 属性值是布尔型Object

 1. `'name|count': object`

    从属性值 `object` 中随机选取 `count` 个属性。

    ````javascript
    Mock.mock({
      "object|2": {
        "310000": "上海市",
        "320000": "江苏省",
        "330000": "浙江省",
        "340000": "安徽省"
      }
    })
    
    // 结果
    {
      "object": {
        "310000": "上海市",
        "320000": "江苏省"
      }
    }
    ````

	2. `'name|min-max': object`

    从属性值 `object` 中随机选取 `min` 到 `max` 个属性

    ```javascript
    Mock.mock({
      "object|2-4": {
        "110000": "北京市",
        "120000": "天津市",
        "130000": "河北省",
        "140000": "山西省"
      }
    })
    
    // 结果
    {
      "object": {
        "110000": "北京市",
        "120000": "天津市",
        "130000": "河北省",
        "140000": "山西省"
      }
    }
    /*
    {
      "object": {
        "110000": "北京市",
        "120000": "天津市",
        "130000": "河北省"
      }
    }
    */
    ```

##### 2.1.1.5 属性值是布尔型Array

 1. `'name|1': array`

    从属性值 `array` 中随机选取 1 个元素，作为最终值。

    ```javascript
    Mock.mock({
      "array|1": [
        "AMD",
        "CMD",
        "UMD"
      ]
    })
    
    // 结果
    {
      "array": "CMD"
    }
    /*
    {
      "array": "UMD"
    }
    {
      "array": "AMD"
    }
    */
    ```

 2. `'name|+1': array`

    从属性值 `array` 中顺序选取 1 个元素，作为最终值。

    ```javascript
    Mock.mock({
      "array|+1": [
        "AMD",
        "CMD",
        "UMD"
      ]
    })
    
    // 结果
    // 第一次
    {
      "array": "AMD"
    }
    // 第二次
    {
      "array": "CMD"
    }
    // 第三次
    {
      "array": "UMD"
    }
    // 第四次
    {
      "array": "AMD"
    }
    ....
    ```

	3. `'name|min-max': array`

    通过重复属性值 `array` 生成一个新数组，重复次数大于等于 `min`，小于等于 `max`。

    ```javascript
    Mock.mock({
      "array|1-10": [
        {
          "name|+1": [
            "Hello",
            "Mock.js",
            "!"
          ]
        }
      ]
    })
    
    // 结果
    {
      "array": [
        {
          "name": "Hello"
        },
        {
          "name": "Mock.js"
        },
        {
          "name": "!"
        }
      ]
    }
    /*
    {
      "array": [
        {
          "name": "Hello"
        },
        {
          "name": "Mock.js"
        }
      ]
    }
    {
      "array": [
        {
          "name": "!"
        },
        {
          "name": "Hello"
        },
        {
          "name": "Mock.js"
        },
        {
          "name": "!"
        },
        {
          "name": "Hello"
        }
      ]
    }
    */
    ```

    ````javascript
    Mock.mock({
      "array|1-10": [
        "Hello",
        "Mock.js",
        "!"
      ]
    })
    
    // 结果
    {
      "array": [
        "Hello",
        "Mock.js",
        "!",
        "Hello",
        "Mock.js",
        "!",
        "Hello",
        "Mock.js",
        "!",
        "Hello",
        "Mock.js",
        "!",
        "Hello",
        "Mock.js",
        "!"
      ]
    }
    /*
    {
      "array": [
        "Hello",
        "Mock.js",
        "!",
        "Hello",
        "Mock.js",
        "!",
        "Hello",
        "Mock.js",
        "!"
      ]
    }
    */
    ````

	4. `'name|count': array`

    通过重复属性值 `array` 生成一个新数组，重复次数为 `count`

    ```javascript
    Mock.mock({
      "array|3": [
        "Mock.js"
      ]
    })
    
    // 结果
    {
      "array": [
        "Mock.js",
        "Mock.js",
        "Mock.js"
      ]
    }
    ```

##### 2.1.1.6 属性值是布尔型Function

 1. `'name': function`

    执行函数 `function`，取其返回值作为最终的属性值，函数的上下文为属性 `'name'` 所在的对象。

    ```javascript
    Mock.mock({
      'foo': 'Syntax Demo',
      'name': function() {
        return this.foo
      }
    })
    
    // 结果
    {
      "foo": "Syntax Demo",
      "name": "Syntax Demo"
    }
    ```

##### 2.1.1.7 属性值是布尔型RegExp

1. `'name': regexp`

   根据正则表达式 `regexp` 反向生成可以匹配它的字符串。用于生成自定义格式的字符串。

```javascript
Mock.mock({
  'regexp': /[a-z][A-Z][0-9]/
})

// 结果
{
  "regexp": "qM1"	// "lB6"、"cL7"、"oV7"...
}
```

```javascript
Mock.mock({
  'regexp': /\w\W\s\S\d\D/
})

// 结果
{
  "regexp": "E$\rQ0i"	// "l#\fe6O"、"C*\f14P"、"e#\tP8H"...
}
```

```javascript
Mock.mock({
  'regexp': /\d{5,10}/
})
 
// 结果
{
  "regexp": "81077"
}
/*
{
  "regexp": "35618475"
}
*/
```

```javascript
Mock.mock({
  'regexp|3': /\d{5,10}\-/
})

// 结果
{
  "regexp": "49623341-6356568-2421816-"
}
/*
{
  "regexp": "655528145-64505014-472678487-"
}
*/
```

```javascript
Mock.mock({
  'regexp|1-5': /\d{5,10}\-/
})

// 结果
{
  "regexp": "5645777804-83666119-"
}
/*
{
  "regexp": "8864630896-"
}
{
  "regexp": "459424985-72120208-6415538-552313197-"
}
*/
```

##### 2.1.1.8 Path

1. Absolute Path

   ```javascript
   Mock.mock({
     "foo": "Hello",
     "nested": {
       "a": {
         "b": {
           "c": "Mock.js"
         }
       }
     },
     "absolutePath": "@/foo @/nested/a/b/c"
   })
   
   // 结果
   {
     "foo": "Hello",
     "nested": {
       "a": {
         "b": {
           "c": "Mock.js"
         }
       }
     },
     "absolutePath": "Hello Mock.js"
   }
   ```

2. Relative Path

   ````javascript
   Mock.mock({
     "foo": "Hello",
     "nested": {
       "a": {
         "b": {
           "c": "Mock.js"
         }
       }
     },
     "relativePath": {
       "a": {
         "b": {
           "c": "@../../../foo @../../../nested/a/b/c"
         }
       }
     }
   })
   
   // 结果
   {
     "foo": "Hello",
     "nested": {
       "a": {
         "b": {
           "c": "Mock.js"
         }
       }
     },
     "relativePath": {
       "a": {
         "b": {
           "c": "Hello Mock.js"
         }
       }
     }
   }
   ````

### 2.2 数据占位符定义规范 DPD

*占位符* 只是在属性值字符串中占个位置，并不出现在最终的属性值中。

*占位符* 的格式为：

```
@占位符
@占位符(参数 [, 参数])
```

**注意：**

1. 用 `@` 来标识其后的字符串是 *占位符*。
2. *占位符* 引用的是 `Mock.Random` 中的方法。
3. 通过 `Mock.Random.extend()` 来扩展自定义占位符。
4. *占位符* 也可以引用 *数据模板* 中的属性。
5. *占位符* 会优先引用 *数据模板* 中的属性。
6. *占位符* 支持 *相对路径* 和 *绝对路径*。

## 3、Mock.mock()

### Mock.mock( rurl?, rtype?, template|function( options ) )

根据数据模板生成模拟数据。

**Mock.mock( template )**
根据数据模板生成模拟数据。

**Mock.mock( rurl, template )**
记录数据模板。当拦截到匹配 `rurl` 的 Ajax 请求时，将根据数据模板 `template` 生成模拟数据，并作为响应数据返回。

**Mock.mock( rurl, function( options ) )**
记录用于生成响应数据的函数。当拦截到匹配 `rurl` 的 Ajax 请求时，函数 `function(options)` 将被执行，并把执行结果作为响应数据返回。

**Mock.mock( rurl, rtype, template )**
记录数据模板。当拦截到匹配 `rurl` 和 `rtype` 的 Ajax 请求时，将根据数据模板 `template` 生成模拟数据，并作为响应数据返回。

**Mock.mock( rurl, rtype, function( options ) )**
记录用于生成响应数据的函数。当拦截到匹配 `rurl` 和 `rtype` 的 Ajax 请求时，函数 `function(options)` 将被执行，并把执行结果作为响应数据返回。

**rurl**  可选。
表示需要拦截的 URL，可以是 URL 字符串或 URL 正则。例如 `/\/domain\/list\.json/`、`'/domian/list.json'`。
**rtype** 可选。
表示需要拦截的 Ajax 请求类型。例如 `GET`、`POST`、`PUT`、`DELETE` 等。

**template**  可选。
表示数据模板，可以是对象或字符串。例如 `{ 'data|1-10':[{}] }`、`'@EMAIL'`。

**function(options)** 可选。
表示用于生成响应数据的函数。

**options**
指向本次请求的 Ajax 选项集，含有 `url`、`type` 和 `body` 三个属性，参见 [XMLHttpRequest 规范](https://xhr.spec.whatwg.org/)。



## 引用

官网：https://github.com/nuysoft/Mock/wiki/Getting-Started