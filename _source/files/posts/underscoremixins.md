---
template: post.html
title: "Underscore mixins"
date: 2014-05-07
tags: [Underscore]
author: David
excerpt: Some methods I usually mix into Underscore
---

Here are some methods I usually mix into Underscore:

### `ensureArray(val,emptyel)`

This method always gives you back an array. There are four different scenarios:

```javascript
// when passed an array, we get that array back:
_.ensureArray([1,2,3]); // => [1,2,3]

// when passed a truthy value that isn't an array,
// you get that value wrapped in an array:
_.ensureArray(1); // => [1]

// when passed a falsy value, we get an empty array:
_.ensureArray(undefvar); // => []

// when passed a falsy value and the optional `emptyel` parameter,
// we get that parameter in an array:
_.ensureArray(undefvar,x); // => [x]
```


The point of the method is of course to safely use an API that expects an array when you're not sure exactly what you've got. Here's the source code:

```
function(o,e){ return o ? [].concat(o) : e ? [e] : [];}

```

A [pull request](https://github.com/jashkenas/underscore/pull/816) to add this into Underscore proper was shot down by Ashkenas for being too special case and because there are other object types too with the same problem. I agree with the first sentiment but not the second; We can easily ensure booleans (`!!x`), strings (`""+x`) and numbers (`+x`), but arrays alone require logic! Converting to objects will always be domain-specific so they don't really play into this discussion.

### `mapObj(obj,iterator,context)`

The `mapObj` follows the filosophy of `map`, but returns an object instead of an array. Here's a silly example:

```
var obj = {a:1,b:2}, fn = function(i){return i*3;};

mapObj(obj,fn); // => {a:3,b:6}
```

There's been lots of pull requests and issues asking for this feature, but as it can be nicely composed and is considered to be a 
rare use case it has been denied. I however find myself very frequently needing to map objects, so being able to do that in a simple
function call cleans up the codebase nicely. Here's the source:

```javascript
function(obj,iterator,context){
	var keys = _.keys(obj);
	return _.reduce(_.map(obj,iterator,context),function(memo,val,i){
		return _.extend(_.object([keys[i]],[val]),memo);
	},{});
}
```

If you're using Lo-Dash then there is already a function called [`mapValues`](http://lodash.com/docs#mapValues) that does all of the above and more!

### `extendProp(obj,propname,src)`

This is a utility method for exending an object which is the property of another object, which works even if 
the property doesn't exist. Doing this:

```javascript
_.extend(obj[propname],src);
```

...would fail if the `propname` property is undefined.Here's the source:

```
function(obj,propname,src){
	obj[propname] = _.extend(obj[propname]||{},source);
	return obj;
}
```

As the parent object is returned we can chain more operations on the target object.


### Mixing it in

The methods are added to underscore through the `mixin` method:

```
_.mixin({
	ensureArray: function(o,e){ return o ? [].concat(o) : e ? [e] : [];},
	mapObj: function(obj,iterator,context){
		var keys = _.keys(obj);
		return _.reduce(_.map(obj,iterator,context),function(memo,val,i){
			return _.extend(_.object([keys[i]],[val]),memo);
		},{});
	},
	extendProp: function(obj,propname,src){ obj[propname] = _.extend(obj[propname]||{},src) },
});
```
