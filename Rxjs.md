# 1. 常见创建类操作符

## 1.1 from

from可以把数组、Promise、以及Iterable转化为Observable

```typescript
import { Observable } from 'rxjs/Observable';
import 'rxjs/add/observable/from'
import 'rxjs';

let arr = ['张三', '李四', '王五', '赵六'];
let arr$ = Observable.from(arr);
arr$.subscribe(value => console.log(value));
```

## 1.2 fromEvent

fromEvent可以把事件转化为Observable

```typescript
import { Observable } from 'rxjs/Observable';
import 'rxjs/add/observable/fromEvent';
import 'rxjs';

let width$ = Observable
    .fromEvent($('#width')[0], 'keyup');
width$.subscribe(value => console.log(value));
```

## 1.3 of

of接收一系列的数据，并把它们emit出去。
```typescript
import { Observable } from 'rxjs/Observable';
import 'rxjs/add/observable/of';
import 'rxjs';

Observable.of(1, 2, 3).subscribe(value => console.log(value));
```

# 2. 常见转换类操作符

## 2.1 map

![avatar](https://raw.githubusercontent.com/hsk287416/AngularNote/master/img/2018-06-23_112531.png)