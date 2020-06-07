# Window对象的各种测试方法总结

## 前言

在平时前端项目开发中有很多需要对`window`对象进行进行操作，比如改变`window.location.href`的值，在前端页面中，这会使浏览器发生页面跳转，还有如`window.location.replace()`, 那么在测试中，虽然 jest 会有部分初始值，但有时候我们需要明确给定值，让测试更明确。

## Window 对象测试分析

window 对象有属性(`property`)和方法(`method`), 在此我们以`href`, `window.location.href` + hash 和 `postMessage`几个特性来测试， 对每个类型(property/method)将使用至少三种方法来展示，测试方法可以分为如下四类：

* delete - 在原对象window上进行测试
* Object.defineProperty - 重新定义属性
* mockfile - mock 整个被测文件
* spyOn - 对特定方法进行mock

## 被测文件

在这可以不用遵循`TDD`的流程，直接给出被测文件内容, 也可在github

```typescript
export const changeHref = (value: string) => {
  window.location.href = value
}

export const addHash = (hash: string): string => {
  return window.location.href + `#${hash}`
}

export const sendMessage = (message) => {
  window.top.postMessage({
    type: 'message',
    data: message,
  }, '*' )
}

```

## 测试

### property - changeHref

#### delete

```typescript
import { changeHref } from '../src/attribute'

describe('attribute', () => {
  const { location } = window
  beforeEach(() => {
    delete window.location

  })
  afterEach(() => {
    window.location = location
  })
  it('should change href to http://test.com when newURL is http://test.com', () => {
    const newURL = "http://test.com"
    window.location = {
      ...location,
      href: ''
    }
    changeHref(newURL)
    expect(window.location.href).toBe(newURL)
  })
})
```

#### Object.defineProperty

```typescript
import { changeHref } from '../src/attribute'

describe('attribute', () => {
  let windowSpy;
  beforeEach(() => {
    windowSpy= jest.spyOn(window, 'location', 'get')
  })
  afterEach(() =>{
    windowSpy.mockRestore()
  })

  it('jest.spyOn', () => {
    expect(window.location.href).toBe('http://localhost/')
    const newURL = "http://test.com"
    windowSpy.mockImplementation(() => ({
        href: ''
    }))
    changeHref(newURL)
    expect(windowSpy).toHaveBeenCalled()
  })
})
```

#### spyOn

```typescript
import { changeHref } from '../src/attribute'

describe('attribute', () => {
  let windowSpy;
  beforeEach(() => {
    windowSpy= jest.spyOn(window, 'location', 'get')
  })
  afterEach(() =>{
    windowSpy.mockRestore()
  })

  it('jest.spyOn', () => {
    expect(window.location.href).toBe('http://localhost/')
    const newURL = "http://test.com"
    windowSpy.mockImplementation(() => ({
        href: ''
    }))
    changeHref(newURL)
    expect(windowSpy).toHaveBeenCalled()
  })
})
```

### method - addHash

#### delete

```typescript
import { addHash } from '../src/attribute'

describe('method', () => {
  const { location } = window
  beforeEach(() => {
    delete window.location;
    window.location = {
      ...location,
      href: 'http://href.com'
    }
  })
  afterEach(() => {
    window.location = location
  })

  it("should return http://href.com#123 when give 123", () => {
    expect(addHash('123')).toEqual('http://href.com#123')
  })
})
```

#### Object.defineProperty

```typescript
import { addHash } from '../src/attribute'
describe('method', () => {
  const { location } = window
  beforeEach(() => {
    Object.defineProperty(window, 'location', {
      value: {
        ...location,
        href: 'http://href.com',
      },
    })
  })
  afterEach(() => {
    Object.defineProperty(window, 'location', location)
  })
  it("should return http://href.com#123 when give 123", () => {
      expect(addHash('123')).toEqual('http://href.com#123')
    })
})
```

#### mockFile

```typescript
import * as attribute from '../src/attribute'
jest.mock('../src/attribute', () => {
  return {
    __esModule: true,
    addHash: jest.fn(),
  };
});

beforeEach( () => {
  jest.resetModules();
})
describe('method', () => {
  it('mocks `addHash`', () => {
    expect(jest.isMockFunction(attribute.addHash)).toBe(true);
  });
  it('verify method has been invoked', () => {
    expect(attribute.addHash).not.toHaveBeenCalled();
    // will failed
    // expect(attribute.addHash('test')).toEqual('http://localhost/#test')
    attribute.addHash('234')
    expect(attribute.addHash).toHaveBeenCalled()
    expect(attribute.addHash).toBeCalledTimes(1)
    expect(attribute.addHash).toBeCalledWith('234')
  })
})
```

#### spyOn

```typescript
import { addHash } from '../src/attribute'

describe('method', () => {
  let windowSpy
  beforeEach(() => {
    windowSpy = jest.spyOn(window, 'location', 'get')
  })
  afterEach(() => {
    windowSpy.mockRestore()
  })
  it('mocks `addHash`', () => {
    expect(jest.isMockFunction(windowSpy)).toBe(true)
  });
  it('spyOn for addHash', () => {
    windowSpy.mockImplementation(() => ({
      href: 'http://test.com',
    }))
    expect(windowSpy).not.toHaveBeenCalled()
    expect(addHash('123')).toEqual('http://test.com#123')
    expect(windowSpy).toHaveBeenCalled();
  })
})
```

### method - postMessage

```typescript
import {sendMessage} from '../src/attribute'

describe('multiple', () => {
  it('sendMessage test with multiple test method', () => {
    Object.defineProperty(window, 'top', {
      value: window,
      writable: true,
      enumerable: true,
      configurable: true,
    })
    Object.defineProperty(window, 'postMessage', {
      writable: true,
      value: jest.fn(),
    })
    sendMessage('message')
    expect(window.parent.postMessage).toHaveBeenCalled()
    expect(window.parent.postMessage).toBeCalledTimes(1)
  })
})
```

## 总结

> 总结就是整理自己，方便自己，如若能方便他人，那就是意外了。

源代码：[https://github.com/AndorLab/test-window-object](https://github.com/AndorLab/test-window-object)

## Reference

* [1.博客:https://guzhongren.github.io/](https://guzhongren.github.io/)
* [2.图床:https://sm.ms/](https://sm.ms/)
* [3.mock-window-location](https://remarkablemark.org/blog/2018/11/17/mock-window-location/)
* [4.jest-how-to-mock-window-location-href](https://wildwolf.name/jest-how-to-mock-window-location-href/)
* [5.Global Object defineProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
* [6.test double](https://www.jyt0532.com/2018/01/24/jinyong-test-double/)

----
![微信公众号](https://s1.ax1x.com/2020/05/23/Yx1I5q.png)


