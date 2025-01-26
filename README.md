This is a __WeakSet__ and __WeakMap__ shim for ES3 and ES5 browsers such as IE5-10 that do not already support `WeakMap` natively.

This shim is intended for code that cannot tolerate adding properties to inserted objects which alas means O(n) search times for most objects. However insertion time is always O(1) and search time is also O(1) for objects of type `String`, `Number`, `Date`, `RegExp`, and `Function` in all browsers, and `Error` in IE8+.

For example if every new object is created with `Object(Math.random())` each inserted object will be a unique Number wrapper. That uniqueness will give them O(1) time complexity on all operations without adding any unwanted properties to them.

It is designed to closely mimic real WeakSets and WeakMaps as much as possible given the tools available to old browsers. Like a real WeakMap or WeakSet, one can do operations from the prototype:

    WeakMap.prototype.set.call(weakMap, key, value);// Throws if weakMap is not a WeakMap instance or key is not an object

To instantiate use `new WeakSet(iterable)` or `new WeakMap(iterable)`. The iterable argument is optional and only requires a separate iterables shim if it is not an array. The constructor also supports no or nullish argument.

Note that `delete` is an ES3 reserved word and must be quoted to work in most ES3 browsers such as IE 6 thru IE 8.

    new WeakSet([Date,Array])['delete'](Date)// returns true

Search times are O(n) for plain objects. But the most recently searched key is always moved to the start of the linked list. That helps to reduce search times by allowing unused keys to go to the end. That also means that an insertion of a key immediately after searching that same key will result in an overwrite since it can do that while maintaining O(1) insertion time.

Delete has an optional non-standard second boolean parameter which when truthy indicates to stop searching after the first mapping or instance of `key` which is useful only when the data structure contains no duplicates. I added this because I had a use case where there are no duplicates and where the deleted item is at the start of the linked list. In such a case it is wasteful to keep searching the linked list after deleting the first match.

    weakMap['delete'](key, true)// don't delete beyond the first match

Remember O(n) search times causes poor performance. That's why it's always better to add any information associated with an object directly to the object itself. This shim is only useful when that isn't possible.
