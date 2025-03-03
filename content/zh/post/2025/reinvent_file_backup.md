+++
title = "重新造轮子系列(二)：文件备份"
date = 2025-03-02T11:57:00-08:00
lastmod = 2025-03-02T21:24:24-08:00
tags = ["reinvent"]
categories = ["ReInvent: 重新造轮子系列"]
draft = false
toc = true
+++

## <span class="section-num">1</span> 前言 {#前言}

既然我们已经有[单元测试]({{< relref "reinvent_unit_test" >}})框架来测试软件了，我们肯定不想已经写好的代码丢失掉。

对于重要的文件，一个必不可少的功能肯定是备份, 这样在丢失文件之后可以重新恢复。

今天我们就来写个简单的文件备份软件，类似 Git 这样的版本系统可以当作是高级版本的文件系统，因为它还支持切换到不同版本，对比版本间的差异等等功能，而我们不打算实现一个版本管理系统，只实现基础的文件备份功能。


## <span class="section-num">2</span> 实现思路 {#实现思路}

{{< figure src="/ox-hugo/reinvent_file_backup_design.png" >}}


### <span class="section-num">2.1</span> 校验文件是否变更 {#校验文件是否变更}

我们不可能备份都将所有的文件备份一次，这样做效率太低了，我们应该只备份发生变更的文件，那么如何高效地判断文件是否发生变更呢？

最简单粗暴的方式是把文件读取出来，然后与以备份的文件作对比，但是这样的效率太低，并且算法复杂度是: O(N), 即运行时间是随着文件内容增长而增长的，文件越长，对比越慢。

最优算法的复杂度是 `O(1)`, 我们希望可以通过常数时间内比较完文件内容。

我们可以使用 [密码哈希算法(Cryptographic hash algorithms)](https://en.wikipedia.org/wiki/Cryptographic_hash_function), 来实现判断文件是否发生变更，它有两个显著的特征:

1.  hash 函数的结果是定长，不会因输入变化而增加或减少
2.  只要输入的任意bit生成变更， hash 函数生成的结果都会不一样

因此我们可以将文件的内容使用密码哈希函数如 `sha1` 来hash, 通过比较两次的哈希结果是否一致来判断文件是否发生变更。


### <span class="section-num">2.2</span> 判断文件是否被备份 {#判断文件是否被备份}

判断文件是否被备份就很直接了，只需要看下当前文件是否在目标路径存在。

再结合上文提到的，只备份内容发生变更的文件，那么我们可以使用哈希函数的结果作为目标路径的备份文件名。

假设有文件 `src/a.txt`, 它的文件内容的哈希结果是 `86f7e437faa5a7fce15d1ddcb9eaeaea377667b8`, 那么我们使用哈希值作为文件名备份到 `dst`, 即 `dst/86f7e437faa5a7fce15d1ddcb9eaeaea377667b8`.

对于文件 `a.txt`, 只需要判断 `dst` 是否存在 `86f7e437faa5a7fce15d1ddcb9eaeaea377667b8`, 就知道 `a.txt` 是否被备份;

更巧妙的是，如果的 `a.txt` 文件内容发生变化，那么它的哈希值就一定不再会是 `86f7e437faa5a7fce15d1ddcb9eaeaea377667b8` 那么查找文件不存在，也可以当作是未备份，直接重新备份。

下面的序列图就是low level design:

{{< figure src="/ox-hugo/reinvent_file_backup_lowlevel_design.png" >}}


### <span class="section-num">2.3</span> 性能优化 {#性能优化}

备份涉及到非常多的文件IO操作，而IO恰恰就是 Nodejs 最擅长的领域, 毕竟曾经的 NodeJS 还有个项目叫做 `io.js`.

NodeJS 的异步IO是基于 [libuv](https://github.com/libuv/libuv), 但是我们不需要支持使用 `libuv` 的API, 只需要把文件相关的操作封装在 `Promise` 里面，NodeJS就会帮我们在处理底层的 IO 调度, 尽可能地并发处理IO, 避免阻塞.

```js
export const hashExisting = (rootDir: string): Promise<PathHashPair[]> => {
    const pattern = `${rootDir}/**/*`;
    return new Promise((resolve, reject) => {
        glob(pattern)
            .then(matches => Promise.all(
                matches.map(path => statPath(path))
            ))
            .then((pairs: PathStatPair[]) => pairs.filter(
                ([path, stat]) => stat.isFile()))
            .then((pairs: PathStatPair[]) => Promise.all(
                pairs.map(([path, stat]) => readPath(path))))
            .then((pairs: PathContentPair[]) => Promise.all(
                pairs.map(([path, content]) => hashPath(path, content))
            ))
            .then((pairs: PathHashPair[]) => resolve(pairs))
            .catch(err => reject(err))
    })
}
```

更多关于 `Promise` 的内容，可以查看[这本书](https://javascript.info/async)，它的解释非常到位.


### <span class="section-num">2.4</span> 测试文件系统 {#测试文件系统}

备份文件的设计我们已经分析和实现完了，接下来肯定是需要编写单元测试来测试我们的函数的，我们的文件备份涉及到非常多的文件操作，免不了要和文件系统打交道，包括创建文件，查找文件等等。

单元测试的其中一个原则就是要尽量屏蔽掉外部系统的依赖，以保证我们只聚焦在测试功能本身，文件系统的读写更像是集成测试需要做的事情, 各种操作也很容易把文件目录结构给搞乱，导致单元测试失败。

所以我们希望可以使用一个 mock object 来把文件系统 mock 掉，[`mock-fs`](https://github.com/tschaub/mock-fs) 这个库做的就是这样的事情, 它可以把程序中的文件操作都 mock 掉，实际操作的是内存对象而非文件系统.

{{< figure src="/ox-hugo/reivent_file_backup_mock_fs.jpg" >}}

我们就可以在每个单元测试运行时，任意构造任何想要的文件目录，并且保证文件操作都是在操纵内存对象，
而不会直接作用到文件系统，保证单元测试的相互隔离。

```js
import mock from 'mock-fs'

describe('checks for pre-existing hashes using mock filesystem', () => {
    beforeEach(() => {
        mock({
            'bck-0-csv-0': {},
            'bck-1-csv-1': {
                '0001.csv': 'alpha.js,abcd1234',
                'abcd1234.bck': 'alpha.js content'
            },
            'bck-4-csv-2': {
                '0001.csv': ['alpha.js,abcd1234',
                             'beta.txt,bcde2345'].join('\n'),
                '3024.csv': ['alpha.js,abcd1234',
                             'gamma.png,3456cdef',
                             'subdir/renamed.txt,bcde2345'].join('\n'),
                '3456cdef.bck': 'gamma.png content',
                'abcd1234.bck': 'alpha content',
                'bcde2345.bck': 'beta.txt became subdir/renamed.txt'
            }
        })
    })

    afterEach(() => {
        mock.restore()
    })
})
```

上面的代码就构造出下如下的文件目录：

```sh
├── bck-0-csv-0
├── bck-1-csv-1
│   ├── 0001.csv
│   └── abcd1234.bck
└── bck-4-csv-2
├── 0001.csv
├── 3028.csv
├── 3456cdef.bck
├── abcd1234.bck
└── bcde2345.bck
```


## <span class="section-num">3</span> 使用示例 {#使用示例}

```sh
> tree .
.
├── backup.ts
├── check-existing-files.ts
├── hash-existing-promise.ts
├── main.ts
├── manifest.ts
├── reinvent_file_backup.org
├── run-hash-existing-promise.ts
├── stream-copy.ts
└── test
    ├── bck-0-csv-0
    ├── bck-1-csv-1
    │   ├── 0001.csv
    │   └── abcd1234.bck
    ├── bck-4-csv-2
    │   ├── 0001.csv
    │   ├── 3028.csv
    │   ├── 3456cdef.bck
    │   ├── abcd1234.bck
    │   └── bcde2345.bck
    ├── test-backup.js
    ├── test-find-mock.js
    └── test-find.js

5 directories, 18 files

> npx tsx main.ts -s . -d /tmp/backup -f json -v
[INFO] Destination directory ensured: /tmp/backup
[INFO] Starting backup from '.' to '/tmp/backup'
[INFO] Copied 8 files from /Users/ramsayleung/code/javascript/reinvent/file_backup to /tmp/backup
Backup completed in: 15.96ms
Backup completed successfully!

> ls -alrt /tmp/backup
total 88
drwxrwxrwt  23 root         wheel   736  2 Mar 17:06 ..
-rw-r--r--@  1 ramsayleung  wheel  1056  2 Mar 21:02 6bd385393bd0e4a4f9a3b68eea500b88165033b1.bck
-rw-r--r--@  1 ramsayleung  wheel  1649  2 Mar 21:02 8b0bc65c42ca2ae9095bb1c340844080f2f054da.bck
-rw-r--r--@  1 ramsayleung  wheel  9795  2 Mar 21:02 464240b6ef1f03652fefc56152039c0f8d105cfe.bck
-rw-r--r--@  1 ramsayleung  wheel   636  2 Mar 21:02 d0f548d134e99f1fcc2d1c81e1371f48d9f3ca0c.bck
-rw-r--r--@  1 ramsayleung  wheel   182  2 Mar 21:02 7fa1b33f68d734b406ddb58e3f85f199851393db.bck
-rw-r--r--@  1 ramsayleung  wheel   666  2 Mar 21:02 369034de6e5b7ee0e867c6cfca66eab59f834447.bck
-rw-r--r--@  1 ramsayleung  wheel  2533  2 Mar 21:02 02d5c238d29f9e49d2a1f525e7db5f420a654a3f.bck
-rw-r--r--@  1 ramsayleung  wheel  3512  2 Mar 21:02 964c0245a5d8cb217d64d794952c80ddf2aecca8.bck
drwxr-xr-x@ 11 ramsayleung  wheel   352  2 Mar 21:02 .
-rw-r--r--@  1 ramsayleung  wheel  1030  2 Mar 21:02 0000000000.json
```

为什么 `file_backup` 目录里面有 18 个文件，只备份了8个文件呢？因为 `test` 目录里面所有的文件都是空的，所以备份时就跳过了。


## <span class="section-num">4</span> 总结 {#总结}

我们就完成了一个文件备份软件的开发，功能当然还非常简单，还有非常多优化的空间，比如现在 `src` 目录的所有文件都会被平铺到 `dst` 目录，如果我们可以保存目录结构，那么就更好用了。

另外，使用哈希函数值作为文件名的确很巧妙，但是对于用户而已，如果不逐个打开文件，根本不知道哪个文件是对应哪个源文件等等。

如果想要实现一个更健壮易用的备份文件，可以参考下关于这 [rsync 系列的文章](https://michael.stapelberg.ch/posts/2022-06-18-rsync-overview/) , `rsync` 是Linux 上非常流行的增量备份的文件，不仅可以备份本地文件，更可以把文件备份把远程服务器，非常强大。


## <span class="section-num">5</span> 参考 {#参考}

-   <https://third-bit.com/sdxjs/file-backup/>
-   <https://en.wikipedia.org/wiki/Cryptographic_hash_function>
-   <https://michael.stapelberg.ch/posts/2022-06-18-rsync-overview/>
-   <https://javascript.info/async>
