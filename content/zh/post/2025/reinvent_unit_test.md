+++
title = "重新造轮子系列(一)：单元测试框架"
date = 2025-02-16T22:27:00-08:00
lastmod = 2025-03-02T21:15:29-08:00
tags = ["reinvent"]
categories = ["ReInvent: 重新造轮子系列"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 前言 {#前言}

单元测试的重要性无须多言，它是保证项目质量的基石.

如果没有单元测试，根本没有信心说自己开发的功能是符合要求的，更没法在没有测试的保证进行项目的重构。

既然单元测试如此重要，今天就用Typescript来写一个简单但五脏俱全的单元测试框架。


## <span class="section-num">2</span> 历史 {#历史}

Javascript 比较流行的测试框架是 [Mocha](https://mochajs.org/) 和 [Jest](https://jestjs.io/) , Java 具有统治地位的单元测试框架就是 [JUnit](https://junit.org/junit5/), 现在做单元测试的框架, 一般称为 xUnit 家族, 而 xUnit 家族最早的成员, 不是 JUnit, 而是 SUnit(Smalltalk Unit), SUnit 的历史比 Junit 悠久得多, 大约在1994年的时候, [Kent Beck](https://en.wikipedia.org/wiki/Kent_Beck), 也就是 Junit 的作者之一, 写了 [SUnit](https://sunit.sourceforge.net/), 而后才有了 JUnit (1998).

所以, 在 [SUnit](https://sunit.sourceforge.net/) 的网站上, 极其显摆的写着”一切单元测试框架之母” (The mother of all unit testing frameworks).

事实上这是大实话 — 所有单元测试框架里面的名词术语, 都从 Sunit 来的, 如 TestCase, Fixture 等等.


## <span class="section-num">3</span> 实现 {#实现}


### <span class="section-num">3.1</span> 需求 {#需求}

先定义需求, 一个单元测试框架应该可以做到下面的事:

1.  找到包含测试的文件
2.  找到上述文件的测试 case
3.  运行测试case
4.  捕获测试运行结果，并输出所有的测试的运行总结


### <span class="section-num">3.2</span> 原型 {#原型}

一条 `assert` 语句就可以看作是最简单的测试 case, 对于测试case, 我们会有以下三种结果：

-   Pass: 运行成功, 测试结果与预期一致
-   Fail: 运行失败, 测试结果与预期不一致
-   Error: 运行测试过程中出现错误，我们不确定测试结果是否与预期一致

我们用以下的状态机来判断测试的结果:

{{< figure src="/ox-hugo/unit_test_result_state.png" >}}

我们把要实现的单元测试框架命名为 `Hope`, 根据上面的状态机，我们很快就可以写出一个原型：

单元测试用例接收一个函数作为参数，然后又集中运行所有的测试用例，并根据是否抛出异常以及异常的类型来判断结果:

```javascript
import assert from 'assert';

const HopeTests: [string, () => void][] = [];
let HopePass = 0;
let HopeFail = 0;
let HopeError = 0;

// Record a single test for running later.
const hopeThat = (message: string, callback: () => void) => {
    HopeTests.push([message, callback]);
}

const main = () => {
    HopeTests.forEach(([message, test]) => {
        try {
            test();
            HopePass += 1;
        } catch (e) {
            if (e instanceof assert.AssertionError) {
                HopeFail += 1;
            } else {
                HopeError += 1;
            }
        }
    });

    console.log(`pass ${HopePass}`);
    console.log(`fail ${HopeFail}`);
    console.log(`error ${HopeError}`);
}
```

让我们编写点代码来测试下我们的「单元测试框架」:

```javascript
// Something to test(doesn't handle zero properly)

const sign = (value: number) => {
    if (value < 0) {
        return -1;
    } else {
        return 1;
    }
}

// These two should pass
hopeThat('Sign of negative is -1', () => assert(sign(-3) === -1));
hopeThat('Sign of positive is 1', () => assert(sign(10) === 1));

// This one should fail.
hopeThat('Sign of zero is 0', () => assert(sign(0) === 0));

// This one is an error.
hopeThat('Sign mispelled is erorr', () => assert(sign(sgn(1) === 1)));

// Call the main driver
main()
```

输出的结果是:

```sh
-> npx tsx dry_run.ts
pass 2
fail 1
error 1
```

我们的第一版单元测试框架 `Hope` 能正常运行了，不过它有几个问题：

1.  它只是输出结果，但没有告诉我们是哪个单元测试成功了，哪个失败了，哪个报错，没法 debug
2.  可变全局变量通常是有很大副作用的，我们应该把它封装起来
3.  如果我们要测的函数里面，预期是要抛出 `assert.AssertionError`, 那么这个函数对应的测试用例就会被识别成失败的测试用例，也就是意味着我们不应该依赖 `assert.AssertError` 来作运行结果判断。


### <span class="section-num">3.3</span> 单例版本 {#单例版本}

我们可以将上面的测试代码地址封装在一个类里，然后通过单例设计模式([Singleton pattern](https://refactoring.guru/design-patterns/singleton))来确保只初始化出一个实例，这样就可以模拟出全局变量的效果，以此来解决前面的两个问题。

```javascript
import assert from "assert";
import caller from 'caller';

class Hope {
  private todo: [string, () => void][] = []; // 记录所有需要运行的测试case.
  private passes: string[] = [];
  private fails: string[] = [];
  private errors: string[] = [];
  constructor() {
    this.todo = [];
    this.passes = [];
    this.fails = [];
    this.errors = [];
  }

  test(comment: string, callback: () => void) {
      // 通过caller 获取单元测试用例对应的文件名
    this.todo.push([`${caller()}::${comment}`, callback]);
  }

  run() {
    this.todo.forEach(([comment, test]) => {
      try {
        test();
        this.passes.push(comment);
      } catch (e) {
        if (e instanceof assert.AssertionError) {
          this.fails.push(comment);
        } else {
          this.errors.push(comment);
        }
      }
    })
  }
}
export default new Hope()
```

上面的代码又是如何实现单例模式的呢？依靠的是 Node 的两个运行机制:

1.  在加载一个 `module` 的时候, 它就会解释并执行 `module` 的代码，这意味着它会运行 `new Hope()` 并且导出新创建的实例
2.  那么是否意味着，每个 `import` 语句都会运行一下 `new Hope()` 呢? 并不是，Node会缓存导入的 `module` ，也就是说无论一个 `module` 被导入多少次, 它也只会执行一次代码。

只要导入 `hope.ts` 之后, 就可以使用 `hope.test()` 会注册单元测试用例，以便后续执行:

{{< figure src="/ox-hugo/unit_test_hope_structure.svg" >}}

最后， 我们只需要再实现下输出测试结果的功能，既支持输出一行的简短结果，又可以支持详尽的输出. 如果需要的话，后续还可以支持输出JSON, CSV, 或者HTML 格式的结果:

```javascript
terse() {
  return this.cases()
    .map(([title, results]) => `${title}: ${results.length}`)
    .join(' ');
}

verbose() {
  let report = '';
  let prefix = '';
  for (const [title, results] of this.cases()) {
    report += `${prefix}${title}:`;
    prefix = '\n';
    for (const r of results) {
      report += `${prefix} ${r}`
    }
  }
  return report;
}

cases() {
  return [
    ['passes', this.passes],
    ['fails', this.fails],
    ['errors', this.errors]
  ]
}
```

万事具备，接下来就让我们写个函数验证下 `Hope` 框架:

```javascript
import assert from "assert";
import hope from "./hope";

hope.test('Sum of 1 and 2', () => assert((1 + 2) === 3));
```

看起来挺不错，但是要怎么运行这个测试case 呢? 总不能每个测试文件都调用下 `hope.run()` 嘛? 人家 `Jest` 都可以自动扫描并运行测试用例。

让我们参考 Jest, 实现一个 `Runner`, 也实现动态加载测试文件.

`import` 不仅可以用来导入其他的模块，它可以当作是一个 async 函数，加载指定路径的文件, 如:

```js
await import(module_path);
```

为了更好地控制我们的单元测试, 我们可以给 `Hope` 框架增加上一些命令行参数以控制其行为, CLI + Runner 的实现如下:

```js
import minimist from 'minimist';
import { glob } from 'glob';
import hope from './hope';
import { fileURLToPath } from 'url';

const parse = (args: string[]) => {
  const parsed = minimist(args)

  return {
    // Default root directory is current directory if not specified
    root: parsed.root || '.',

    // Output format can be 'terse' or 'verbose' (default)
    output: parsed.output || 'verbose',

    // Array of test filenames if explicitly provided
    filenames: parsed._ || []
  }
}

const main = async (args: Array<string>) => {
  const options = parse(args);
  if (options.filenames.length == 0) {
    options.filenames = await glob(`${options.root}/**/test*.{ts,js}`);
  }

  for (const f of options.filenames) {
    const absolutePath = fileURLToPath(new URL(f, import.meta.url));
    await import(absolutePath);
  }
  hope.run()
  const result = (options.output === 'terse') ? hope.terse() : hope.verbose();
  console.log(result);
}

main(process.argv.slice(2))
```

我们默认会匹配所有以 `test` 为前缀的 ts 和 js 文件, 然后通过 `import` 导入, 因为 `hope` 是单例模式，所以所有的测试文件用的都是同一个实例, `hope.run` 就将注册的所有单元测试运行.

整个框架的工作流程如下:

{{< figure src="/ox-hugo/unit_test_workflow.png" >}}

大功告成，现在就来运行下我们的单元测试:

```sh
> npx tsx pray.ts
passes:
 file:///private/tmp/reinvent/unit_test/test_add.ts::Sum of 1 and 2
fails:
errors:
```


### <span class="section-num">3.4</span> 优化 {#优化}


#### <span class="section-num">3.4.1</span> 增加运行时间 {#增加运行时间}

我们还可以记录每个测试用例的运行时间, 纳秒有点太小了，就精确到微秒即可:

```js
run() {
  this.todo.forEach(([comment, test]) => {
    try {
      const now = process.hrtime.bigint();
      test();
      const elapsedInMicro = (process.hrtime.bigint() - now) / (BigInt(1000));
      this.passes.push(comment + `, execution time: ${elapsedInMicro}us`);
    } catch (e) {
      if (e instanceof assert.AssertionError) {
        this.fails.push(comment);
      } else {
        this.errors.push(comment);
      }
    }
  })
}
```

```sh
> npx tsx pray.ts
passes:
 file:///private/tmp/reinvent/unit_test/test_add.ts::Sum of 1 and 2, execution time: 5us
fails:
errors:
```


#### <span class="section-num">3.4.2</span> 增加 assert 函数 {#增加-assert-函数}

内置的 `assert` 函数只支持比较输入值是否为 True, 现代的测试框架都有很多的 `helper` 函数来简化 `assert` 语句，就让我们来实现下 `assertEqual`, `assertThrows`, `assertMapEqual`, `assertSetEqual`, `assertArraySame` 这几个函数:

```js
/**
 * assert 抛出指定的异常
 */
export function assertThrows<T extends Error>(expectedType: new (...args: any[]) => T, func: () => void) {
    try {

        // expected to throw exception
        func();
        // unreachable
        assert(false, `Expected function to throw ${expectedType.name} but it did not throw`);
    } catch (error) {
        assert(error instanceof expectedType, `Expected function to throw ${expectedType.name} but it threw ${error instanceof Error ? error.constructor.name : typeof error}`);
    }
}

/**
 * assert 两个元素相等
 */
export function assertEqual<T>(actual: T, expected: T, message: string) {
    assert(actual === expected, message);
}

/**
 * assert 两个 Set 相同
 */
export function assertSetEqual<T>(actual: Set<T>, expected: Set<T>, message: string) {
    assert(actual.size == expected.size, message);
    for (const element of actual) {
        assert(expected.has(element), message);
    }
}

/**
 * assert 两个 Map 相同
 */
export function assertMapEqual<K extends string | number | symbol, V>(actual: Record<K, V>, expected: Record<K, V>, message: string) {
    const actualKeys = Object.keys(actual) as K[];
    const expectedKeys = Object.keys(expected) as K[];

    assert(actualKeys.length === expectedKeys.length, message);
    for (const actualKey of actualKeys) {
        assert(expected[actualKey] && actual[actualKey] == expected[actualKey], message);
    }
}

/**
 * assert两个列举的值相等，如元素相等，但是顺序不同也被视为相同
 */
export function assertArraySame<T>(actual: Array<T>, expected: Array<T>, message: string) {
    assert(actual.length === expected.length, message);
    assertSetEqual(new Set(actual), new Set(expected), message);
}
```

针对上述函数的测试:

```js
import assert from "assert";
import hope, { assertArraySame, assertMapEqual, assertSetEqual, assertThrows } from "./hope";

hope.test('test assertSetEqual happy path', () => {
  const setA = new Set([1, 2, 3, 4, 5]);
  const setB = new Set([5, 1, 2, 4, 3]);
  assertSetEqual(setA, setB, 'Set supposed to be equal');

  assertSetEqual(new Set([]), new Set([]), 'Empty Set');
});

hope.test('test assertMapEqual unhappy path', () => {
  assertThrows(assert.AssertionError, () => {
    const setA = new Set([1, 2, 3, 4, 5]);
    const setB = new Set([1, 2, 4, 3]);
    assertSetEqual(setA, setB, 'Set supposed to be equal');
  })
});

hope.test('test assertMapEqual happy path', () => {
  const mapA = {
    'a': 1,
    'b': 2,
  };
  const mapB = {
    'b': 2,
    'a': 1
  };
  assertMapEqual(mapA, mapB, 'Map supposed to be map');
});

hope.test('test assertMapEqual unhappy path', () => {
  const mapA = {
    'a': 1,
    'b': 3
  };
  const mapB = {
    'b': 2,
    'a': 1
  };
  assertThrows(assert.AssertionError, () => {
    assertMapEqual(mapA, mapB, 'Map supposed to be map');
  });
});


hope.test('test assertArraySame happy path', () => {
  const arr1 = [1, 2, 3, 2];
  const arr2 = [2, 1, 2, 3];
  assertArraySame(arr1, arr2, "Arrays should have same elements"); // Passe
});

hope.test('test assertArraySame unhappy path', () => {
  const arr1 = [1, 2, 3, 2];
  const arr2 = [2, 1, 2, 4];

  assertThrows(assert.AssertionError, () => {
    assertArraySame(arr1, arr2, "Arrays should have same elements"); // Passe
  });
});
```


#### <span class="section-num">3.4.3</span> 增加 -s/--select 参数指定测试文件 {#增加-s-select-参数指定测试文件}

我们的 `Runner` 默认匹配的是以 `test` 为前缀的测试文件, 我们可以增加一个 `-s/--select` 参数，用来指定需要匹配的测试文件名：

```js
const parse = (args: string[]) => {
  const parsed = minimist(args)

  return {
    ...
    select: parsed.select || parsed.s // 增加select 参数
  }
}

const main = async (args: Array<string>) => {
  const options = parse(args);
  if (options.filenames.length == 0) {
    const namePattern = options.select ?? 'test*'; // 使用传入的模式
    options.filenames = await glob(`${options.root}/**/${namePattern}.{ts,js}`);
  }

  ...
}
```

运行结果:

```sh
> ls -al test*
-rw-r--r--@ 1 ramsayleung  wheel   115 17 Feb 10:01 test_add.ts
-rw-r--r--@ 1 ramsayleung  wheel   762 17 Feb 10:01 test_approx_equal.ts
-rw-r--r--@ 1 ramsayleung  wheel  1536 17 Feb 10:38 test_assert.ts
-rw-r--r--@ 1 ramsayleung  wheel   187 17 Feb 10:38 test_async.ts
-rw-r--r--@ 1 ramsayleung  wheel   275 17 Feb 10:38 test_setup_teardown.ts
-rw-r--r--@ 1 ramsayleung  wheel   140 17 Feb 10:38 test_tag.ts

> npx tsx pray.ts -s "test_a*"
passes:
 file:///private/tmp/reinvent/unit_test/test_async.ts::delayed test, execution time: 412us
 file:///private/tmp/reinvent/unit_test/test_assert.ts::test assertSetEqual happy path, execution time: 31us
 file:///private/tmp/reinvent/unit_test/test_assert.ts::test assertMapEqual unhappy path, execution time: 1175us
 file:///private/tmp/reinvent/unit_test/test_assert.ts::test assertMapEqual happy path, execution time: 32us
 file:///private/tmp/reinvent/unit_test/test_assert.ts::test assertMapEqual unhappy path, execution time: 85us
 file:///private/tmp/reinvent/unit_test/test_assert.ts::test assertArraySame happy path, execution time: 17us
 file:///private/tmp/reinvent/unit_test/test_assert.ts::test assertArraySame unhappy path, execution time: 54us
 file:///private/tmp/reinvent/unit_test/test_approx_equal.ts::Default margin throws exception, execution time: 111us
 file:///private/tmp/reinvent/unit_test/test_approx_equal.ts::Large margin not throws exception, execution time: 6us
 file:///private/tmp/reinvent/unit_test/test_approx_equal.ts::Relative error throw exception, execution time: 51us
 file:///private/tmp/reinvent/unit_test/test_approx_equal.ts::Default Relative error not throw exception: , execution time: 5us
 file:///private/tmp/reinvent/unit_test/test_add.ts::Sum of 1 and 2, execution time: 4us
fails:
errors:
```


#### <span class="section-num">3.4.4</span> 增加 -t/--tag 参数按标签运行测试case {#增加-t-tag-参数按标签运行测试case}

对于 `hope.test` 函数，我们还可以提供一个额外的参数，用于给这个test case 打标签:

```js
hope.test('Difference of 1 and 2',
          () => assert((1 - 2) === -1),
          ['math', 'fast'])
```

然后通过 `-t/--tag` 按指定的tag来运行测试用例, 实现起来很容易:

```js
test(comment: string, callback: () => void, tags: Array<string> = []) {
    this.todo.push([`${caller()}::${comment}`, callback, tags]);
}

run(tag: string = '') {
    this.todo
        .filter(([comment, test, tags]) => {
            if (tag.length === 0) { return true; }
            return tags.indexOf(tag) > - 1;
        })
        .forEach(([comment, test, tags]) => {
            // run the test, nothing change
        })
}
```

```js
const parse = (args: string[]) => {
    const parsed = minimist(args)

    return {
        ...
            tag: parsed.tag || parsed.t
    }

    const main = async (args: Array<string>) => {
        ...
            hope.run(options.tag);
        ...
    }
```

`test_tag.ts`:

```js
import assert from "assert";
import hope from "./hope";
hope.test('Differene of 1 and 2', () => assert((1 - 2) === -1), ['math', 'fast']);
```

```sh
> npx tsx pray.ts -t "math"
passes:
 file:///private/tmp/reinvent/unit_test/test_tag.ts::Differene of 1 and 2, execution time: 5us
fails:
errors:
```


#### <span class="section-num">3.4.5</span> setup与teardown {#setup与teardown}

正常的测试框架都是有 `setup` 与 `teardown` 函数的，可以指定在每个测试case 运行之前或之后的函数，比如运行测试case 前的数据准备，以为运行结束时的数据清理，我们的测试框架也可以支持这个功能：

```js
type CallbackType = () => void;
class Hope {
  ...
  private setupFn: CallbackType | null = null;
  private teardownFn: CallbackType | null = null;

  setup(setupFn: CallbackType) {
    this.setupFn = setupFn;
  }

  teardown(teardownFn: CallbackType) {
    this.teardownFn = teardownFn;
  }

  run(tag: string = '') {
    this.todo
      .filter(([comment, test, tags]) => {
        if (tag.length === 0) { return true; }
        return tags.indexOf(tag) > - 1;
      })
      .forEach(([comment, test, tags]) => {
        try {
          if (this.setupFn) {
            this.setupFn();
          }

          const now = microtime.now();
          test();
          const elapsedInMicro = microtime.now() - now;
          this.passes.push(comment + `, execution time: ${elapsedInMicro}us`);

          if (this.teardownFn) {
            this.teardownFn();
          }
        } catch (e) {
          if (e instanceof assert.AssertionError) {
            this.fails.push(comment);
          } else {
            this.errors.push(comment);
          }
        }
      })
  }
}
```

针对上述函数的测试:

```js
import hope, { assertEqual } from "./hope";

let x = 0;

const createFixtures = () => {
  x = 1;
}

hope.setup(createFixtures);
hope.test('Validate x should be 1', () => {
  assertEqual(x, 1, 'X should be 1');
});

const cleanUp = () => {
  x = 0;
}

hope.teardown(cleanUp);
```


#### <span class="section-num">3.4.6</span> 增加对 async 测试case 的支持 {#增加对-async-测试case-的支持}

目前我们的test case 都只支持同步的函数, 我们可以增加上对 `Promise` 的支持, 这样我们可以使用以下的语法:

```js
hope.test('delayed test', async () => {...})
```

实现方式也很直接: 一种就是判断传入函数的类型, 如果是同步函数则直接调用，如果是 async 函数, 那么就加上 `await`:

```js
type SyncCallbackType = () => void;
type AsyncCallbackType = () => Promise<void>;
type CallbackType = SyncCallbackType | AsyncCallbackType;

class Hope {
    private todo: [string, CallbackType, Array<string>][] = [];
    private setupFn: CallbackType | null = null;
    private teardownFn: CallbackType | null = null;

    setup(setupFn: CallbackType) {
        this.setupFn = setupFn;
    }

    teardown(teardownFn: CallbackType) {
        this.teardownFn = teardownFn;
    }

    test(comment: string, callback: () => void, tags: Array<string> = []) {
        this.todo.push([`${caller()}::${comment}`, callback, tags]);
    }

    private async runTest(comment: string, test: CallbackType, tags: string[]) {
        try {
            if (this.setupFn) {
                if (this.isAsync(this.setupFn)) {
                    await this.setupFn();
                } else {
                    this.setupFn();
                }
            }

            const now = process.hrtime.bigint()
            if (this.isAsync(test)) {
                await test();
            } else {
                test();
            }

            const elapsedInMicro = (process.hrtime.bigint() - now) / (BigInt(1000));
            this.passes.push(comment + `, execution time: ${elapsedInMicro}us`);

            if (this.teardownFn) {
                if (this.isAsync(this.teardownFn)) {
                    await this.teardownFn();
                } else {
                    this.teardownFn();
                }
            }
        } catch (e) {
            if (e instanceof assert.AssertionError) {
                this.fails.push(comment);
            } else {
                this.errors.push(comment);
            }
        }
    }

    async run(tag: string = '') {
        const tests = this.todo
              .filter(([comment, test, tags]) => {
                  if (tag.length === 0) { return true; }
                  return tags.indexOf(tag) > - 1;
              });


        for (const [comment, test, tags] of tests) {
            await this.runTest(comment, test, tags);
        }
    }

    private isAsync(fn: CallbackType): fn is AsyncCallbackType {
        return fn.constructor.name === 'AsyncFunction';
    }
}
```

`pray.ts`:

```js
const main = async (args: Array<string>) => {
  const options = parse(args);
  if (options.filenames.length == 0) {
    const namePattern = options.select ?? 'test*';
    options.filenames = await glob(`${options.root}/**/${namePattern}.{ts,js}`);
  }

  for (const f of options.filenames) {
    const absolutePath = fileURLToPath(new URL(f, import.meta.url));
    await import(absolutePath);
  }

  await hope.run(options.tag); // 增加上await
  const result = (options.output === 'terse') ? hope.terse() : hope.verbose();
  console.log(result);
}await hope.run(options.tag);
```


## <span class="section-num">4</span> 参考 {#参考}

-   <https://third-bit.com/sdxjs/unit-test/>
-   <https://blog.youxu.info/2008/11/30/pearl-in-smalltal/>
