---
title: View的事件拦截机制分析
---

# 基本 

## 流程 

分发 dispatchTouchEvent ——拦截 onlnterceptTouchEvent ——消费(onTouchEvent)

## 方法 

### 分发 dispatchTouchEvent 

1. true——被当前视图消费，不再分发
2. super.dispatchTouchEvent ——表示继续分发该事件

### 拦截 onlnterceptTouchEvent

**只在ViewGroup及其子类才存在**

1.true——拦截这个事件——自身调用onTouchEvent消费该事件
2.false/super——**不拦截**——传递给子视图

### 消费  onTouchEvent

1. true——消费该事件
2. false——不消费——向上传递给**父视图**的 onTouchEvent


# 一些结论 

- 触摸事件会按照**嵌套顺序**从外向内层传递，到达最内层的view时，如果能消费，返回true；否则返回false，这时事件会重新向外层传递；
- View事件顺序是**先执行onTouch**，最后执行 onclick；如果onTouch返回true，则不会执行 onclick；返回false，则会执行onclick