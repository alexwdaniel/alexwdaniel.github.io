---
layout: post
title: Swift Dictionaries
---

The dictionary is one of the most common and flexible data structure used across almost all programming languages. It is an abstract data type representing the mapping between two sets, consisting of a collection of key-value pairs. This associative array can have many different implementations, including binary search trees, skip lists, graphs, and hash tables, each with their own performance and memory utilization tradeoffs. Dictionaries are commonly referred to as hashes, alluding to the most prevalent implementation using a hash table, offering the best performance and memory utilization for insertion, retrieval, and removal.

<!--more-->

Coming from Ruby, I am used to seeing hashes used in a wide variety of contexts, from storing default values, to manipulating JSON data, and even as lightweight alternatives to defining custom classes. Swift introduced a new kind of dictionary with a shortened syntax, updated implementation, and type safety guarantees; however, NSDictionary will not be disappearing anytime soon. This article is indented to highlight some of the differences between the Swift library’s dictionary and the Foundation NSDictionary class. The principal distinction between Swift dictionaries and NSDictionary must begin with a discussion of the differences between value and reference types.

####Value & Reference Types

Value types, as their name implies, are copied when passed to functions as arguments or assigned to variables. References types, by contrast, are passed by reference; the data they encapsulate is not copied and only exists at one dynamically allocated location in the heap.

Reference type objects are generally considered to be the default programming unit in object oriented languages, and rightly so. Objects offer many more features compared to structures, including class inheritance, reflection, de/initializers, as well as the ability to have multiple owners. It is precisely these powerful features that make classes less suited for implementing foundational data structures.

In Swift, dictionaries are implemented as value types using structs, while instances of the NSDictionary class are reference types. Dictionaries are copied when they are assigned to a new variable or passed to a function or method. Modifications to that data only alter the local copy, not the original source of information. Copying potentially large amounts of data naturally introduces performance considerations; however, Swift intelligently manages the actual physical copying of data and does so only when necessary. In contrast to the immutability benefits of localized data, modifying a reference type does alter the source. Subsequent accesses of the data encapsulated by the reference type will return the updated information with no knowledge of the original value.

Swift dictionary are mutable in that it is possible to change its contents, but because it is a value type and copied when passed to another function, subsequent mutations do not alter the values of the calling functions local copy.

Value types are better suited for data structures as they generally do not have behavior that manipulates their internal state. Though Swift structs can define methods to provide functionality, the functionality typically does not alter the internal data but rather computes and returns a distinct value constructed from the internal data. Objects encapsulate behavior and communicate with other objects by sending and receiving messages that manipulate its internal data thereby altering internal state.

Reference types are easily shared across multiple locations in a program, with all of these locations pointing to the same data. Any of these locations can access the object’s data either directly manipulating data or by calling methods that mutate its state. These disparate locations might also try and access the same reference at the same time causing concurrency issues.

Using value types makes code easier to understand by minimizing the number of dependencies and reducing the interaction between objects in the system.

####Creating a Dictionary

*Swift Dictionary*

{% highlight swift %}

let myDict : Dictionary<Int, String> = [1: “One”, 2: “Two”]
let myDictShort : [Int: String] = [1: “One”, 2: “Two”]
let myDictShorter = [1: “One”, 2: “Two”]

{% endhighlight %}

*NSDictionary*

{% highlight swift %}

let myEmptyNSDict: NSDictionary = NSDictionary()
let myShortNSDict = NSDictionary(object: “value1”, forKey: “key1”)

let dictKeys = ["key1", "key2", "key3", "key4"]
let dictValues = ["value1", "value2", "value3", "value4"]
let myNSDict = NSDictionary(objects: dictValues, forKeys: dictKeys)

{% endhighlight %}

####Mutability

Swift brings mutability to the forefront through the use of constants (let) and variables (var). Both must be declared before they can be used, but constants cannot be changed once they are set; they are immutable. This language feature necessitates some care when using dictionaries. As we have already seen, Swift dictionaries are value types while both NSDictionary and NSMutableDictionary are reference types. For reference types, immutability means once an instance of a class is assigned to a variable, another instance cannot be assigned to that same variable. This does not mean that the data contained in the object is immutable. For instance, it is perfectly legal in Swift to declare an NSMutableDictionary as a constant, yet still append, remove, or modify it’s values. The same cannot be said for a standard Swift dictionary; once it is declared as a constant, it’s values cannot be changed. The immutable NSDictionary behaves more like a value type.

{% highlight swift %}

let dict = NSMutableDictionary()
dict.setValue(“value”, forKey: “key”) //ok

var dict = NSDictionary()
dict.setValue(“value”, forKey: “key”) //error

let dict = [“key1” : “value1”]
dict[“key2”] = “value2” //error

{% endhighlight %}

####Type Safety

Swift dictionaries are type safe, meaning you cannot add keys or values with respectively different types to a dictionary. NSDictionary is not type safe and therefore can have keys and values of differing types.

{% highlight swift %}

let mismatchedDict: [String : String] = ["stringKey1" : "stringValue1", "stringKey2" : 55, NSNumber(double: 3.14) : "stringValue3"]

{% endhighlight %}

As we have tried to initialize a dictionary of type [String : String] with elements of differing types, the compiler complains. To fix this, we must either remove the non-string type elements or declare the dictionary as one of more generic types. In this case, this dictionary would need to be [NSObject : NSObject].

####Nil Values

Swift dictionaries allow nil values while NSDictionary does not.

{% highlight swift %}

var nilDict = [“key1”: “value1”, “key2”: “value2”]
nilDict[“key2”] = nil

{% endhighlight %}

Setting a value for a key to nil removes the key-value pair from the dictionary. With an NSDictionary, you can still represent null values with the NSNull singleton object but the key-value pair will still exist in the dictionary.

{% highlight swift %}

let nilNSDict = NSMutableDictionary(objects: ["value1", "value2"], forKeys: ["key1", "key2"])
nilNSDict.setValue(NSNull(), forKey: "key1")
nilNSDict.allKeys // [“key1”, “key2”]

{% endhighlight %}

####Acceptable Keys

Both the Swift dictionary and NSDictionary are implemented using hash tables.

A hash table consists of an array and a hash function. The hash function applied to the key determines the index in the array where the key-value pair resides. Ideally, the function would uniformly distribute values across all indices, where each index consisted of a single key-value pair. Perfect hashing rarely occurs and is dependent upon the chosen keys. As a result, collisions occur when multiple keys map to the same array index. The most common strategy for resolving collisions is separate chaining. Separate chaining can be implemented with a variety of data structures; however, a linked list is most common. Instead of storing the key-value pair in the array itself, the index location points to a list of pairs with the same hash value. Key-value pairs are then appended to the end the list.

Returning the correct value requires iterating over the list of key-value pairs until a match is found. Instead of having to iterate over the entire array of key-value pairs, the hash function serves to reduce the search space and yields an average case complexity of O(1). A poor hash function, where all keys map to the same index, results in a single list of values and a worst case complexity of O(n).

Custom Swift objects must conform to the Hashable protocol in order to be used as keys in a dictionary. The Hashable protocol requires the object to have a hashValue property. The Hashable protocol is derived from the Equatable protocol and therefore requires an implementation of ==. The custom equality function overloads the == operator and therefore must be defined in the global scope. It takes two arguments of the same class and returns a boolean value indicating whether or not they are equal. Two equal objects must have the same hash value; however, they need not be equal in order for their hash values to be equal.

{% highlight swift %}

class MyClass: Hashable {
	var id: Int
	var hashValue: Int {
		return self.id or something
	}

	init() {}
}

func ==(arg1: MyClass, arg2: MyClass) {
	return arg1.id == arg2.id
}

{% endhighlight %}

This custom class can now be used as a key in Swift’s dictionary.

A potential cause of error occurs when using reference types as keys in a Swift dictionary. Objects used as keys are not copied and can result in values being unreachable should their data mutate.

{% highlight swift %}

let key1 = NSMutableString(string: "key1")
let key2 = NSMutableString(string: "key2")
let dictionaryWithMutableKeys = [key1 : "value1", key2 : “value2”]

dictionaryWithMutableKeys[key1] // value1

key1.appendString("somemoretext")

dictionaryWithMutableKeys[key1] // nil
dictionaryWithMutableKeys[NSMutableString(string: "key1")] // nil

dictionaryWithMutableKeys[key2] // value2

{% endhighlight %}

*NSDictionary*

Keys for an NSDictionary must be a subclass of NSObject that overrides the hash and isEqual methods and must also adopt the NSCopying protocol. The hash and isEqual implementation requirement corresponds to the Hashable protocol required for Swift dictionaries. An important difference between NSDictionary and the Swift dictionary is the stipulation that NSDictionary keys also conform to the NSCopying protocol. The NSCopying protocol declares a single required method, copyWithZone. This prevents the problem seen in the Swift dictionary that occurs when mutable keys are altered. The updated key no longer provides access to the value in its pair. NSDictionary solves this problem by making its own copy of the key. It is even intelligent enough to only copy mutable keys. By default, objects that are values of an NSDictionary are stored as a strong reference and are not coped, though there is an optional initializer NSDictionary(dictionary:[NSObject : AnyObject], copyItems: Bool).

{% highlight swift %}

let key1 = NSMutableString(string: "key1")
let key2 = NSMutableString(string: "key2")
let dictionary = NSDictionary(objects: ["value1", "value2"], forKeys: [key1, key2])

dictionary[key1] // value1

key1.appendString("somemoretext")

dictionary[key1] // nil
dictionary[NSMutableString(string: "key1")] // value1

dictionaryWithMutableKeys[key2] // value2

{% endhighlight %}

---

While looking at the documentation for the NSCopying protocol I did some reading about NSZone. It is a Foundation data type for managing memory zones. A quick Google and I came across [this](http://stackoverflow.com/questions/6097007/what-is-nszone-what-are-the-advantages-of-using-initwithzone) great explanation.

---

####Accessing Dictionary Elements

Inserts, updates, and deletes from Swift dictionaries are fairly straightforward using square bracket subscripting. The return value is an optional as the specified key may not exist.

{% highlight swift %}

var dictionary = ["key1" : "value1", "key2" : "value2", "key3" : "value3"]
let elem : String? = dictionary[“key2"] // value2
let missingElem : String? = dictionary[“missingKey"] // nil

dictionary["addedKey"] = "addedValue"
dictionary.updateValue("updatedValue", forKey: “addedKey")

dictionary.removeValueForKey("addedKey")
let removedElem : String? = dictionary[“addedKey"] // nil

{% endhighlight %}

The main difficultly of working with NSDictionary is dealing with AnyObject. While NSDictionary can contain keys and values of different types, Swift dictionaries cannot. Therefore, choosing which dictionary to use depends on whether the added flexibility of holding objects of mismatched type outweighs the slight annoyance of having to optionally cast objects on retrieval.

When converting from an NSDictionary to a Swift dictionary the result will be of type [NSObject: AnyObject] because the dictionary needs to be of more generic type to function like an NSDictionary. Once you have a Swift dictionary, you can downcast it to one of a more specific type.

####Iterating Over Dictionaries

Tuples in Swift make iteration even easier:

{% highlight swift %}

for (key, value) in myDictionary {
	//do things
}

{% endhighlight %}

*NSDictionary*

{% highlight swift %}

myDictionary.enumerateKeysAndObjectsUsingBlock {
	(key, value, pointer) in
	//do more things
}

{% endhighlight %}

###Additional Resources

- [Apple Developer Documentation](https://developer.apple.com/library/mac/documentation/Swift/Conceptual/BuildingCocoaApps/WorkingWithCocoaDataTypes.html#//apple_ref/doc/uid/TP40014216-CH6-XID_46){:target="_blank"}
- [NSDictionary](http://ciechanowski.me/blog/2014/04/08/exposing-nsdictionary/){:target="_blank"}
- [Objc.io](http://www.objc.io/issue-16/swift-classes-vs-structs.html){:target="_blank"}
- [Apple Cocoa Conceptual Documentation](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Collections/Articles/Dictionaries.html){:target="_blank"}
