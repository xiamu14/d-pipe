<p align="center">
<img width="200" src="https://img.icons8.com/clouds/500/000000/synchronize.png"/>
</P>
 <p align="center">
 <img src="https://img.shields.io/badge/coverage-100%25-brightgreen">
 <img src="https://img.shields.io/badge/min%20size-1%20kb-blue">
 <img src="https://img.shields.io/npm/dt/data-matcher.svg?colorB=ff69b4">
 <!-- <img src="https://img.shields.io/npm/dt/d-pipe.svg?colorB=ff69b4"> -->
 </p>

## 数据管道转换工具

## 用途

在数据处理过程中，不同环节所储存或反馈的数据字段、类型总会存在差异。

服务端 api 数据和前端数据的差异，前端组件和第三方组件数据的差异，数据在不同环节的差异等。

如果基于同一领域，同一场景，同一产品的认知，这种差异往往只是数据表象的差异，比如：key 命名不一致，value 数据类型不同，数据原始或复合状态不同。

通过开发经验归纳，大致有下面三种差异：

1. 初始数据和使用时数据的复合形态不同：初始数据应该包含产品所需的全部内容，但在使用时却需要不同的复合形态，比如：select 选项初始数据包含 {id,name}，第三方组件使用时需要 {key,value,label}。[add ，delete 方法组合使用可抹平差异]
2. key 命名不一致 [editKey 方法可修改 key]
3. value 数据类型不一致 [editValue 方法可修改 value]

当然，实际开发中还存在着更复杂的差异，那样的情状建议特例化处理。

## 安装

```bash
yarn add d-pipe
```

```bash
pnpm install d-pipe
```

## 使用

```js
const data = { a: 'a', b: 'b' };
const pipe = new Pipe(data);
pipe
  .add('c', () => 'c')
  .delete(['b'])
  .editValue('a', () => 'aa');
pipe.data; // { a: 'aa', c: 'c' }
```

> Tips：传入的数据必须是非空对象和非空数组，{} 和 [] 都是不允许的数据。开发者必须在业务中明确判断数据是非空的，才能传递给 Pipe。
>
> 否则 Pipe 将抛出一个类型错误： The dataSet must be an Object or Array,and cannot be an empty object or empty array.

## 方法

### add (增加数据)

- 定义

  ```js
  public add(key: DataItemKey, valueFn: (data: any) => any)
  ```

- 参数
  | 参数 | 描述 | 默认值 | 类型 |
  | ------ | ----------- | ------ | ------ |
  | key | 键 | -- | string 或 Symbol |
  | valueFn | 值的生成函数 | -- | (data: DataItem) => any |

- 示例
  ```js
  const data = { startTime: '2019/09/12', endTime: '2019/09/30' };
  const pipe = new Pipe(data);
  pipe.add('dateRange', (data) => `${data.startTime}-${data.endTime}`);
  pipe.data; // {startTime: '2019/09/12', endTime: '2019/09/30', dateRange:'2019/09/12-2019/09/30'}
  ```

### pick (取出指定数据)

- 定义

  ```js
  public pick(keys: DataItemKey[])
  ```

- 参数
  | 参数 | 描述 | 默认值 | 类型 |
  | ------ | ----------- | ------ | ------ |
  | keys | 键数组 | -- | 包含 string 或 Symbol 的数组 |

- 示例

  ```js
  const data = { key: '1', label: 'apple', value: 'apple' };
  const pipe = new Pipe(data);
  pipe.pick(['label']);
  pipe.data; // {label: 'apple',}
  ```

- 场景
  用于从表格中编辑某些字段时，只取出一条数据里的部分进行编辑操作。虽然 delete 也可以实现，但偶尔会出现以后增加统计数据的情况，为避免每次增加数据都要去 delete ，最好使用 pick 方法来明确指定编辑的数据。

> 注意，pick 和 delete 时互相冲突的，使用 pick 的话，delete 将无效。优先保留指定数据而非删除数据。

### delete (删除指定数据)

- 定义

  ```js
  public delete(keys: DataItemKey[])
  ```

- 参数
  | 参数 | 描述 | 默认值 | 类型 |
  | ------ | ----------- | ------ | ------ |
  | keys | 键数组 | -- | 包含 string 或 Symbol 的数组 |

- 示例
  ```js
  const data = { key: '1', label: 'apple', value: 'apple' };
  const pipe = new Pipe(data);
  pipe.delete(['label']);
  pipe.data; // {key:'1',value:'apple'}
  ```

### editValue (修改 value)

- 定义

  ```js
  public editValue(
    key: DataItemKey,
    valueFn: (value: any, data: DataItem) => any,
  )
  ```

- 参数
  | 参数 | 描述 | 默认值 | 类型 |
  | ------ | ----------- | ------ | ------ |
  | key | 键 | -- | string 或 Symbol |
  | valueFn | 值的生成函数 | -- | (key:any, data: DataItem) => any |

- 示例
  ```js
  const data = { price: 1, createAt: 1632833413149 }; // 价格服务端存储单位[分]
  const pipe = new Pipe(data);
  pipe
    .editValue('price', (value) => value / 100)
    .editValue('createAt', (value) => dayjs(value).format('YYYY/MM/DD'));
  pipe.data; // {price:0.01, createAt:'2021/09/28'}
  ```

### editKey (修改 key)

- 定义

  ```js
  public editKey(keyMap: Record<DataItemKey, DataItemKey>)
  ```

- 参数
  | 参数 | 描述 | 默认值 | 类型 |
  | ------ | ----------- | ------ | ------ |
  | keyMap | 新旧 key 对象 | -- | {旧 key：新 key} |

- 示例
  ```js
  const data = { id: '1' };
  const matcher = new Pipe(data);
  matcher.editKey({ id: 'key' });
  marcher.data; // {key:'1'}
  ```

### clone (拷贝数据)

- 定义

  ```js
  public clone(keyMap: Record<DataItemKey, DataItemKey>)
  ```

- 参数
  | 参数 | 描述 | 默认值 | 类型 |
  | ------ | ----------- | ------ | ------ |
  | keyMap | key 对象 | -- | {元 key：新 key} |

- 示例
  ```js
  const data = { a: 'a', b: { bb: 'bb' } };
  const pipe = new Pipe(data);
  pipe.clone({ a: 'a/' });
  pipe.data; // {a:'a', b:{bb:'bb'}, 'a/':'a'}
  ```

### clean (清除无意义的数据)

- 定义

  ```js
  public clean(invalidValues: MayBeInvalidType[])
  ```

- 参数
  | 参数 | 描述 | 默认值 | 类型 |
  | ------ | ----------- | ------ | ------ |
  | invalidValues | 无意义的值集合 | -- | null 、 undefined 、 '' 、 'null' 、 'undefined' 、 0 |

- 场景
  某些情况，数据里因为初始值等原因会出现 null 等值，但实际使用时需要去除值为 null 的字段

- 示例

  ```js
  const data = { price: 100, projectId: undefined };
  const pipe = new Pipe(data);
  pipe.clean([undefined]);
  pipe.data; // {price:100}
  ```

### when (根据条件组合操作)

- 定义

  ```js
  public when(
    condition: boolean,
    whenTruthy: ((that: Matcher) => void) | null,
    whenFalsy: ((that: Matcher) => void) | null,
  )
  ```

- 参数
  | 参数 | 描述 | 默认值 | 类型 |
  | ------ | ----------- | ------ | ------ |
  | condition | 条件值 | -- | boolean | true or false |

- 示例
  ```js
  const data = { a: 'a', b: { bb: 'bb' } };
  const pipe = new Pipe(data);
  pipe.when(data.a === 'a', (that) => that.add('aa', () => 'aa'), null);
  pipe.data; // {a:'a', b:{bb:'bb'}, 'aa':'aa'}
  ```

## 特性

1. immutable：内部使用 cloneDeep 函数，对传入的数据进行深拷贝，所有修改不影响原始数据
   ```js
   constructor(data: any) {
    this.originalData = deepClone(data);
    this.result = deepClone(data);
   }
   ```
2. 方法调用顺序无关：可以自由使用链式调用方法，内部使用固定的方法执行顺序（add[增加数据]->pickOrDelete[保留或删除数据]->editValue[修改 value] -> editKey[修改 key] -> clean[清除数据]）调用对应的方法来执行，确保数据操作的正确性
   ```js
   // 源码测试用例
   test('valueDelivery', () => {
     const data = { a: 'a', b: 'b' };
     const pipe = new Pipe(data);
     pipe
       .add('c', () => 'c')
       .delete(['b'])
       .editValue('a', () => 'aa');
     expect(pipe.data).toEqual({ a: 'aa', c: 'c' }); // pass
     // 顺序无关
     pipe
       .editValue('a', () => 'aa')
       .add('c', () => 'c')
       .delete(['b']);
     expect(pipe.data).toEqual({ a: 'aa', c: 'c' }); // pass
   });
   ```
3. 归纳操作：Pipe 内部会收集所有调用方法，以便于对数组数据只使用一次遍历完成数据操作
4. 链式调用：使用链式调用方法，对数据的操作代码更有组织性
