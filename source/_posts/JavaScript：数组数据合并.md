---
title: JavaScript：数组数据合并
date: 2022-05-22 19:47:01
tags:
    - JavaScript
    - 数组
categories:
    - JavaScript进阶
---

# 假设业务场景

现在需要一个两个数组的高效合并方案

![5-8-1](5-8-1.png)

-   第一次请求返回一个列表信息
-   第二次请求根据列表信息中的每一个发送一次请求，返回第二个列表数据

# 模拟场景

-   将第二组数据根据 uid 合并至第一组数据

```javascript
export const usersInfo = Array.from({ length: 200 }, (val, index) => {
    return {
        uid: `${index + 1}`,
        name: `user-name-${index}`,
        age: index + 10,
        avatar: `http://www.my-avatar.com/${index + 1}`
    }
})

export const scoresInfo = Array.from({ length: 180 }, (val, index) => {
    return {
        uid: `${index + 191}`,
        score: ~~(Math.random() * 10000),
        comments: ~~(Math.random() * 10000),
        stars: ~~(Math.random() * 1000)
    }
})
```

---

# 数据合并 - 基础版本

-   方案：两层 for 循环，通过 key 关联
-   耗时：> 3.8ms

```javascript
import * as data from './data.js'

const { usersInfo, scoresInfo } = data

for (let i = 0; i < usersInfo.length; i++) {
    var user: any = usersInfo[i]
    for (let j = 0; j < scoresInfo.length; j++) {
        var score = scoresInfo[j]
        if (user.uid == score.uid) {
            user.score = score.score
            user.comments = score.comments
            user.stars = score.stars
        }
    }
}
```

---

# 数据合并 - 对象基础版

-   方案：数组转换为对象，数组查找变为属性查找
-   耗时：0.1ms - 0.7ms

```javascript
import * as data from './data.js'

const { usersInfo, scoresInfo } = data

const scoreMap = scoresInfo.reduce((obj, cur) => {
    obj[cur.uid] = cur
    return obj
}, Object.create(null))

for (let i = 0; i < usersInfo.length; i++) {
    const user: any = usersInfo[i]
    const score = scoreMap[user.uid]

    if (score != null) {
        user.score = score.score
        user.comments = score.comments
        user.stars = score.stars
    }
}
```

---

# 数据合并 - 跳出

-   统计已被合并的数据条数，当已合并条数大于等于需要被合并的数据总条数时进行跳出

```javascript
import * as data from './data.js'

const { usersInfo, scoresInfo } = data

const scoreMap = scoresInfo.reduce((obj, cur) => {
    obj[cur.uid] = cur
    return obj
}, Object.create(null))

// 被合并数据的条数
const len = scoresInfo.length
// 已合并的条数
let count = 0
// 已遍历的次数
let walkCount = 0
for (let i = 0; i < usersInfo.length; i++) {
    const user: any = usersInfo[i]
    const score = scoreMap[user.uid]

    walkCount++
    if (score != null) {
        count++
        user.score = score.score
        user.comments = score.comments
        user.stars = score.stars

        if (count >= len) {
            break
        }
    }
}
```

---

# 合并数据 - 倒序合并

-   方案：在跳出版的基础上，一个是从前向后，一个是从后向前
-   适用场景：分页拉取数据，先数组添加在最后，倒序更快

```javascript
import * as data from './data.js'

const { usersInfo, scoresInfo } = data

const scoreMap = scoresInfo.reduce((obj, cur) => {
    obj[cur.uid] = cur
    return obj
}, Object.create(null))

const len = scoresInfo.length
let count = 0
let walkCount = 0
for (let i = usersInfo.length - 1; i >= 0; i--) {
    const user: any = usersInfo[i]
    const score = scoreMap[user.uid]

    walkCount++
    if (score != null) {
        count++
        user.score = score.score
        user.comments = score.comments
        user.stars = score.stars

        if (count >= len) {
            break
        }
    }
}
```

---

# 数组合并 - 通用方案

-   顺序问题
    -   倒序遍历
    -   顺序遍历
-   合并多级属性
-   学习优秀开源库的解决方法， 例如 underscore 和 lodash

## 多级属性合并

> 来自 lodash

-   stringToPath：路径转换成数组
-   getProperty：读属性
-   setProperty：设属性

```javascript
// https://github.com/lodash/lodash/blob/master/.internal/stringToPath.js

const charCodeOfDot = '.'.charCodeAt(0);
const reEscapeChar = /\\(\\)?/g;
const rePropName = RegExp(
    // Match anything that isn't a dot or bracket.
    '[^.[\\]]+' + '|' +
    // Or match property names within brackets.
    '\\[(?:' +
    // Match a non-string expression.
    '([^"\'][^[]*)' + '|' +
    // Or match strings (supports escaping characters).
    '(["\'])((?:(?!\\2)[^\\\\]|\\\\.)*?)\\2' +
    ')\\]' + '|' +
    // Or match "" as the space between consecutive dots or empty brackets.
    '(?=(?:\\.|\\[\\])(?:\\.|\\[\\]|$))'
    , 'g');

/**
 * Converts `string` to a property path array.
 *
 * @private
 * @param {string} string The string to convert.
 * @returns {Array} Returns the property path array.
 */
const stringToPath = (string: string) => {
    const result = [];
    if (string.charCodeAt(0) === charCodeOfDot) {
        result.push('');
    }
    string.replace(rePropName, ((match, expression, quote, subString) => {
        let key = match;
        if (quote) {
            key = subString.replace(reEscapeChar, '$1');
        }
        else if (expression) {
            key = expression.trim();
        }
        result.push(key);
    }) as any);
    return result;
};
```

```javascript
/**
 * 获取对象属性
 * https://github.com/lodash/lodash/blob/master/.internal/baseGet.js
 * @param obj
 * @param key
 * @param defaultValue
 * @returns
 */
function getProperty(obj: Object, key: string, defaultValue: any = undefined) {
    if (!isObject(obj)) {
        return defaultValue
    }

    const path = stringToPath(key)

    let index = 0
    const length = path.length

    while (obj != null && index < length) {
        obj = obj[path[index++]]
    }
    return index && index == length ? obj : undefined || defaultValue
}

/**
 * 设置属性值
 * https://github.com/lodash/lodash/blob/master/.internal/baseSet.js
 * @param obj
 * @param path
 * @param value
 * @returns
 */
```

```javascript
function setProperty(obj: Object, path: string, value: any = undefined) {
    if (!isObject(obj)) {
        return obj
    }
    const keys = stringToPath(path)

    const length = keys.length
    const lastIndex = length - 1

    let index = -1
    let nested = obj

    while (nested != null && ++index < length) {
        const key = keys[index]
        let newValue = value

        if (index != lastIndex) {
            const objValue = nested[key]
            newValue = undefined
            if (newValue === undefined) {
                newValue = isObject(objValue) ? objValue : isIndexLike[keys[index + 1]] ? [] : {}
            }
        }
        nested[key] = newValue
        nested = nested[key]
    }
    return obj
}
```

## 正序和倒序（迭代器）

-   传统方式：if/else + 两个 for，while，数组的 forEach，reduce 等
-   弊端：逻辑判断倒序和逆序，可读性差
-   解决方案：实现迭代器：hasNext 和 current

```javascript
/**
 * 循环迭代器
 * TODO:: 对象形式
 * @param min
 * @param max
 * @param desc
 * @returns
 */
function getStepIter(min: number, max: number, desc: boolean) {
    let [start, end, step] = desc ? [max, min, -1] : [min, max, 1]
    const hasNext = () => (desc ? start >= end : end >= start)
    return {
        hasNext() {
            return hasNext()
        },
        get current() {
            return start
        },
        next() {
            return (start += step)
        }
    }
}
```

## 对象合并

-   创建空对象
-   将被合并和需要合并的两个对象的属性抽离处理添加到新的空对象上

```javascript
/**
 * 合并对象生成新的对象
 * // TODO:: 无限合并
 * @param obj1
 * @param obj2
 * @param ob1KMap
 * @param ob2KMap
 * @returns
 */
export function mergeObject<T = any, S = any, R = any>(
    obj1: T,
    obj2: S,
    ob1KMap: Record<string, string> = null,
    ob2KMap: Record<string, string> = null
): R {
    const ret = Object.create(null)

    Object.assign(ret, extractObject(obj1, ob1KMap))

    Object.assign(ret, extractObject(obj2, ob2KMap))

    return ret
}
```

## 通用方案

```javascript
/**
 * 合并数组生成新的数组
 * @param targetArr 目标数组
 * @param sourceArr 需要被合并的数组
 * @param options.desc  是否是从后往前遍历
 * @param options.soureKey  源数组对象的key
 * @param options.targetKey  目标数组对象的key
 * @param options.sKMap  源复制map关系
 * @returns
 */
export function mergeArray<S = any, T = any, R = any>(targetArr: T[] = [], sourceArr: S[] = [], options: MergeOptions<S, T> = DEFAULT_MERGE_OPTIONS): R[] {

    // 有一个不是数组
    if (!Array.isArray(sourceArr) || !Array.isArray(targetArr)) {
        // return [...targetArr] as any[];
        return targetArr as any;
    }

    const opt: MergeOptions = { ...DEFAULT_MERGE_OPTIONS, ...options };

    // 判断sourceKey和sourceProperties
    if (typeof opt.sourceKey !== "string") {
        console.error("无效的soureKey");
        return targetArr as any[];
    }

    const wTypes = ["string", "number"];

    // TODO:: 更安全的检查
    const getSKeyFn = typeof opt.sourceKey === "function" ? opt.sourceKey : (s: S) => s[opt.sourceKey as string];

    let { targetKey } = opt;
    if (targetKey == null) {
        targetKey = opt.sourceKey;
    }

    const getTKeyFn = typeof targetKey === "function" ? targetKey : (t: T) => t[opt.targetKey as string];

    const objMap: Record<string, S> = sourceArr.reduce((obj: Record<string, S>, cur: S) => {
        const key = getSKeyFn(cur);
        if (wTypes.includes(typeof key)) {
            obj[cur[key]] = cur
        }
        return obj;
    }, Object.create(null));

    const { desc, sourceKey, sKMap = null } = opt;

    const sourceLen = sourceArr.length;
    let hitCounts = 0;
    let walkCounts = 0;

    let resultArr = Array.from(targetArr);
    const targetLen = targetArr.length;
    let tempTObj, keyValue, tempSObj;

    const stepIter = getStepIter(0, targetLen - 1, desc);

    while (stepIter.hasNext()) {

        const index = stepIter.current;

        walkCounts++

        if (walkCounts > MAX_WLAK_COUNT) {
            console.error(`mergeArray 遍历次数超过最大遍历次数 ${MAX_WLAK_COUNT}, 终止遍历，请检查程序逻辑`);
            break;
        }

        tempTObj = resultArr[index];

        // 目标比对的键值
        keyValue = tempTObj[getTKeyFn(tempTObj)];
        if (keyValue == null || (tempSObj = objMap[keyValue]) == null || tempSObj[sourceKey] != keyValue) {
            stepIter.next()
            continue;
        }

        resultArr[index] = mergeObject(tempTObj, tempSObj, undefined, sKMap);
        hitCounts++
        if (hitCounts >= sourceLen) {
            break;
        }

        stepIter.next()
    }

    console.log(`mergeArray:: sourceArr(${sourceLen}), 统计：遍历次数${walkCounts}, 命中次数${hitCounts}`);
    return resultArr as any[];
}
```

```javascript
const arr = mergeArray(usersInfo, scoresInfo, {
    sourceKey: 'uid',
    targetKey: 'uid',
    sKMap: {
        score: 'score.score',
        comments: 'score.comments',
        stars: 'stars'
    }
})
```
