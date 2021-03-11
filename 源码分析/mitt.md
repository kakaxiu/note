# midd

`midd`是vue3官方推荐的全局事件总成库。代码非常简单，只有200b，85行代码

## 实现
```js
import mitt from 'mitt'

const emitter = mitt()

// listen to an event
emitter.on('foo', e => console.log('foo', e) )

// listen to all events
emitter.on('*', (type, e) => console.log(type, e) )

// fire an event
emitter.emit('foo', { a: 'b' })

// clearing all events
emitter.all.clear()

// working with handler references:
function onFoo() {}
emitter.on('foo', onFoo)   // listen
emitter.off('foo', onFoo)  // unlisten
```

## 功能
* 实现一个包含事件总线`all`的闭包函数对象
* `on`方法注册事件
* `emit`方法触发事件
* `off`方法注销事件
* `all.clear`方法清除所有注册事件

## 源码
```ts
// 事件名/标识
export type EventType = string | symbol;

// 定义事件句柄类型
// An event handler can take an optional event argument
// and should not return a value
export type Handler<T = any> = (event?: T) => void;
// 占位符句柄，与普通句柄的区别是监听*事件，传参多了一个
export type WildcardHandler = (type: EventType, event?: any) => void;

// 句柄数组（事件注册数组，多个函数可以顺序执行）
// An array of all currently registered event handlers for a type
export type EventHandlerList = Array<Handler>;
export type WildCardEventHandlerList = Array<WildcardHandler>;

// 基于Map的事件总线
// A map of event types and their corresponding event handlers.
export type EventHandlerMap = Map<EventType, EventHandlerList | WildCardEventHandlerList>;

// 方法对象类型定义
export interface Emitter {
	all: EventHandlerMap;

	on<T = any>(type: EventType, handler: Handler<T>): void;
	on(type: '*', handler: WildcardHandler): void;

	off<T = any>(type: EventType, handler: Handler<T>): void;
	off(type: '*', handler: WildcardHandler): void;

	emit<T = any>(type: EventType, event?: T): void;
	emit(type: '*', event?: any): void;
}

/**
 * Mitt: Tiny (~200b) functional event emitter / pubsub.
 * @name mitt
 * @returns {Mitt}
 */
export default function mitt(all?: EventHandlerMap): Emitter {
	all = all || new Map();

	return {

		/**
		 * A Map of event names to registered handler functions.
		 */
		all,

		/**
		 * Register an event handler for the given type.
		 * @param {string|symbol} type Type of event to listen for, or `"*"` for all events
		 * @param {Function} handler Function to call in response to given event
		 * @memberOf mitt
		 */
		on<T = any>(type: EventType, handler: Handler<T>) {
			const handlers = all.get(type);
			const added = handlers && handlers.push(handler);
			if (!added) {
				all.set(type, [handler]);
			}
		},

		/**
		 * Remove an event handler for the given type.
		 * @param {string|symbol} type Type of event to unregister `handler` from, or `"*"`
		 * @param {Function} handler Handler function to remove
		 * @memberOf mitt
		 */
		off<T = any>(type: EventType, handler: Handler<T>) {
			const handlers = all.get(type);
			if (handlers) {
				handlers.splice(handlers.indexOf(handler) >>> 0, 1);
			}
		},

		/**
		 * Invoke all handlers for the given type.
		 * If present, `"*"` handlers are invoked after type-matched handlers.
		 *
		 * Note: Manually firing "*" handlers is not supported.
		 *
		 * @param {string|symbol} type The event type to invoke
		 * @param {Any} [evt] Any value (object is recommended and powerful), passed to each handler
		 * @memberOf mitt
		 */
		emit<T = any>(type: EventType, evt: T) {
			((all.get(type) || []) as EventHandlerList).slice().map((handler) => { handler(evt); });
			((all.get('*') || []) as WildCardEventHandlerList).slice().map((handler) => { handler(type, evt); });
		}
	};
}
```