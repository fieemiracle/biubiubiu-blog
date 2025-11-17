---
title: 枚举
tags:
  - 预览
  - 枚举
createTime: 2025/09/10 16:45:50
permalink: /article/wtq3sk25/
---

## enum

当你需要定义一些固定值的集合的场景时，你通常都是如何考虑的?例如对于一个订单状态集合，未支付、支付成功、支付失败、已取消、已发货、已签收、已退款，对应的数字分别是:1、2、3、4、5、6、7。

### type

你会选择定义一个单纯的类型，简洁直观，无运行开销吗?
```typescript
type OrderStatus = 1 | 2 | 3 | 4 | 5 | 6 | 7
```
实际上这并不是我想要的，显然不是集合，无法直接通过"键"访问值，必须手写字符串，要是换成字符串，还有拼写风险，当然写TS是会帮你校验的。

### 普通对象
最初我使用普通对象去满足这项要求:
```typescript
const OrderStatus = {
  UNPAID: 1,
  PAID_SUCCESS: 2,
  PAID_FAILED: 3,
  CANCELLED: 4,
  SHIPPED: 5,
  SIGNED: 6,
  REFUNDED: 7,
}
```
可以直接通过"键"访问值，例如`OrderStatus.UNPAID`，这是我想要的。
但是这种方式也有一些不直观的地方，类型松散，TS会自动推断该对象的类型为`{ UNPAID: number, PAID_SUCCESS: number, PAID_FAILED: number, CANCELLED: number, SHIPPED: number, SIGNED: number, REFUNDED: number }`，这显然不是我想要的。
另外，虽然枚举类型规定了1-7，但是我直接赋值其他number类型数字，也是没啥问题的，TS也不会报错。

当然这个也有解决方式，就是使用`as const`断言，将对象的所有属性都变成只读的字面量类型。
```typescript
const OrderStatus = {
  UNPAID: 1,
  PAID_SUCCESS: 2,
  PAID_FAILED: 3,
  CANCELLED: 4,
  SHIPPED: 5,
  SIGNED: 6,
  REFUNDED: 7,
} as const
```
这样，TS就会自动推断该对象的类型为`{ readonly UNPAID: 1, readonly PAID_SUCCESS: 2, readonly PAID_FAILED: 3, readonly CANCELLED: 4, readonly SHIPPED: 5, readonly SIGNED: 6, readonly REFUNDED: 7 }`，这显然是我想要的。
兼具了普通对象的"键"访问值的便利性，同时也具备了枚举类型的"值"访问键的便利性。
但是它又不能当作类型去使用，需要手动使用`typeof OrderStatus`来提取它的联合类型。

```typescript
type OrderStatusT = typeof OrderStatus[keyof typeof OrderStatus]
```
所以要兼具普通对象的"键"访问值的便利性，同时也具备了枚举类型的"值"访问键的便利性，以及使用联合类型。那么只能另辟蹊径了。

### enum
最后我选择了使用枚举类型去满足这项要求:
```typescript
enum OrderStatus {
  UNPAID = 1,
  PAID_SUCCESS = 2,
  PAID_FAILED = 3,
  CANCELLED = 4,
  SHIPPED = 5,
  SIGNED = 6,
  REFUNDED = 7,
}
```