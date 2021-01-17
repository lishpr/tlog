---
layout: post
title:  "Understanding Redis: 3"
author: "Shori"
comments: false
tags: Docs
---

# Redis datatypes

After spending some seemingly bewildering time of introducing the basic data structures in Redis. We are getting into the Redis datatype system.

> "Wait, haven't we *already* been talking about datatypes?"

Well, yes and no.

In the previous chapters, we introduced ```sds```, skip lists, ziplists, ```intset```, etc. Those are the *implementations* of the datatypes that *users* interacts with, such as strings, lists, and sets. We have ```sds``` and ```int``` for strings, skip lists, ziplists, and ```intset``` for list and sets, etc.

Therefore, there needs to be a way of keep tracking of the datatypes for each key, in spite of their implementation. And as a result, users should be able to call methods, for example ```ZADD```, on values regardless of the internal data structure, which could be dicts or ```intset```.

Yeah, that's called polymorphism. As C is not an OO language, Redis implemented its way of handling "objects", type-checks, explicit polymorphism, and object life-cycles.

## 3.1 Redis objects

### 3.1.1 Implementation
```c
// server.h
typedef struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    unsigned lru:LRU_BITS; /* LRU time (relative to global lru_clock) or
                            * LFU data (least significant 8 bits frequency
                            * and most significant 16 bits access time). */
    int refcount;
    void *ptr;
} robj;
```
As we could see, the struct ```redisObject``` documents the ```type```, ```encoding```, and ```ptr``` of the object. Where ```type``` is the types as users understands it, ```encoding``` is the actual implementation of the types, and ```ptr``` is a pointer to the data structure that keeps the value.

```c
// TYPE
#define OBJ_STRING 0    /* String object. */
#define OBJ_LIST 1      /* List object. */
#define OBJ_SET 2       /* Set object. */
#define OBJ_ZSET 3      /* Sorted set object. */
#define OBJ_HASH 4      /* Hash object. */

// ENCODING
#define OBJ_ENCODING_RAW 0     /* Raw representation */
#define OBJ_ENCODING_INT 1     /* Encoded as integer */
#define OBJ_ENCODING_HT 2      /* Encoded as hash table */
#define OBJ_ENCODING_ZIPMAP 3  /* Encoded as zipmap */
#define OBJ_ENCODING_LINKEDLIST 4 /* No longer used: old list encoding. */
// And more ...
```

### 3.1.2 Type-checks and polymorphism

Because of the ```redisObject``` struct, performing type-checks and polymorphism is much easier. When the user is performing an operation, Redis will:

1. Find the ```redisObject``` corresponds to the key in the database dictionary. If not found, return ```NULL```.
2. Check the ```type``` of the ```redisObject```. If the type does not permit the operation that the user intends to perform, return type error.
3. Calls the method that corresponds to the operation for the ```encoding```.
4. Returns the return value of the method called in 3.

### 3.1.3 Object sharing

There are objects that are commonplace in Redis, such as some return values like ```OK```, ```ERROR``` strings, and some small integers.