# WeakMap and WeakSet monkey patch / shim

This repository contains both a WeakMap/WeakSet monkey patch for modern JS (IE11 thru ES2025) and a WeakMap/WeakSet shim for old browsers (ES3 thru ES5). They are independent files that work independently.

## Monkey Patch

This monkey patches `WeakMap` and `WeakSet` constructors so that they can accept iterables and unregistered symbols (E.g. `Symbol.keyFor(symbol)==undefined`). All prototype methods are also monkey patched. This also fixes non-standard behaviors notorious of Firefox. It works in strict mode and non-browser environments. This is intended for environments where `WeakMap` is already natively defined which gives all operations O(1) time. Polyfills `WeakSet` and all prototypal methods where not already native.

This is for environments that pre-date holding weak references to unregistered symbols. Therefore there a no actual weak references to unregistered symbols here.

Small file size <2K

## Shim

This `WeakSet` and `WeakMap` shim is for ES3 and ES5 browsers such as IE5-10 that do not already support `WeakMap` natively. It has O(1) insertion time on set() and add() operations, and O(n) search time on get() has() and delete() operations. However search time is reduced to O(1) for objects of type `String`, `Number`, `Date`, `RegExp`, and `Function` in all browsers, and `Error` in IE8+.

For example if every new object is created with `Object(Math.random())` each inserted object will be a unique Number wrapper. That uniqueness will give them O(1) time complexity on all operations without adding any unwanted properties to them.

It is designed to closely mimic real WeakSets and WeakMaps as much as possible given the tools available to old browsers while keeping its minified file size under 2K. Like a real WeakMap or WeakSet, one can do operations from the prototype:

    WeakMap.prototype.set.call(weakMap, key, value);// Throws if not a WeakMap instance or key is not an object

To instantiate use `new WeakSet(iterable)` or `new WeakMap(iterable)`. The iterable argument is optional and can be either an array-like with indexed elements or an iterable i.e. an object where [Symbol.iterator]() returns an iterator. The constructor also supports no or nullish argument.

Note that `delete` is an ES3 reserved word and must be quoted to work in most ES3 browsers such as IE 6 thru IE 8.

    new WeakSet([Date,Array])['delete'](Date)// returns true

Search times are O(n) for plain objects. But the most recently searched key is always moved to the start of the linked list. That helps to reduce search times by allowing unused keys to go to the end. That also means that an insertion of a key immediately after searching that same key will result in an overwrite since it can do that while maintaining O(1) insertion time.

Delete has an optional non-standard second boolean parameter which when truthy indicates to stop searching after the first mapping or instance of `key` which is useful only when the data structure contains no duplicates. I added this because I had a use case where there are no duplicates and where the deleted item is at the start of the linked list. In such a case it is wasteful to keep searching the linked list after deleting the first match.

    weakMap['delete'](key, true)// don't delete beyond the first match

Remember O(n) search times causes poor performance. That's why it's always better to add any information associated with an object directly to the object itself. This shim is only useful when that isn't possible.
