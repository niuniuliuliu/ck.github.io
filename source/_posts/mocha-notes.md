---
title: Mocha笔记
date: 2021-03-15 16:35:16
categories:
- web前端
tags:
- 测试
---
# Mocha笔记

## 测试脚本
mocha默认加载test目录下的test.js，也可以通过参数指定需要运行的测试文件，如运行test目录下的test.math.js。

```
mocha test/test.math.js
```
mocha的测试脚本
```
describe('test of math', function () {
    it('should return 2 when 1 + 1', function () {
        assert.equal(math.add(1, 1), 2);
    });
});
```
* describe: 表示一个测试套件
* it： 表示一个测试用例

<!-- more -->

上面使用的是nodejs的assert断言库，也可以安装其他断言库，如
* should.js
* expect.js 
* chai - expect()
* better-assert 
* unexpected 

## mocha参数
mocha允许在test目录下通过mocha.opts文件，进行参数配置。
mocha.opts
```
--timeout 5000
--reporter mochawesome
```

## 测试报表
控制台显示的测试结果：
```
test of math
    ✓ should return 2 when 1 + 1
    ✓ use expect assertion: should return 2 when 1 + 1
    ✓ asyncAdd, should return 2 when 1 + 1 (2004ms)
    - a pending test


  3 passing (2s)
  1 pending
```
--reporter参数可以配置使用的测试报表名称，如spec, dot, nyan, mochawesome

mochawesome需要安装
```
npm install --save-dev mochawesome
```
报表生成在mochawesome-report目录下，为html文件。

## 代码覆盖率测试
Istanbul
```
npm install --save-dev nyc
```
在mocha命令前增加nyc
```
nyc mocha test/test.math.js
```
运行改命令后：
```
----------|----------|----------|----------|----------|-------------------|
File      |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
----------|----------|----------|----------|----------|-------------------|
All files |      100 |      100 |      100 |      100 |                   |
 math.js  |      100 |      100 |      100 |      100 |                   |
----------|----------|----------|----------|----------|-------------------|
```
如需要生成代码覆盖率的报告，可以修改命令为：
```
nyc --reporter=html mocha test/test.math.js
```
运行命令后，覆盖率报表会生成在coverage目录下。

## test.math.js
```
var assert = require('assert');
var expect = require('expect.js');
var Math = require('../math');

var math;

describe('test of math', function () {
    //hooks
    before(function () {
        // runs before all tests in this block
        math = new Math();
    });

    after(function () {
        // runs after all tests in this block
    });

    beforeEach(function () {
        // runs before each test in this block
    });

    afterEach(function () {
        // runs after each test in this block
    });

    it('should return 2 when 1 + 1', function () {
        assert.equal(math.add(1, 1), 2);
    });

    it('should return 1 when 1 * 1', function () {
        assert.equal(math.mutiply(1, 1), 1);
    });

    it('should return max value 9 of [1,3,6,9,0]', function () {
        assert.equal(math.max([1, 3, 6, 9, 0]), 9);
    });

    it('should return 2 when 1 + 1', function () {
        assert.equal(math.add(1, 1), 2);
    });

    it('use expect assertion: should return 2 when 1 + 1', function () {
        expect(math.add(1, 1)).to.be(2);
    });

    it('asyncAdd, should return 2 when 1 + 1', function (done) {
        math.asyncAdd(1, 1).then(function (result) {
            assert.equal(result, 2);
            done();
        }, function (err) {
            assert.fail(err);
            done();
        });
    });

    it('a pending test');
});
```