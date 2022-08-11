实用类型

Partical<T>:将对象的属性都变成可选

> type MyPartical<T> = { [p in keyof T]?: T[p]  } 

Required<T>: 将对象的属性都变成必选

> type MyRequired<T> = {[p in keyof T] : T[p]}

Readonly<T>: 将对象的属性都变成不可更改

> type MyReadyonly<T> = {readonly [p in keyof T] : T[p]}