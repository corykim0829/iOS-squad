# Advanced Memory Management Programming Guide

## About Memory Management

**어플리케이션의 메모리 관리**는 프로그램의 런타임동안의 메모리 할당, 메모리 사용, 사용을 마친 메모리 해제의 프로세스라고 할 수 있다. 잘짜여진 프로그램은 메모리를 가능한 적게 사용한다. Objective-C에서는, 많은 데이터와 코드들 중에서 한정된 메모리 자원의 소유권을 분산시키는 것으로 보일 수 있다. 이 가이드를 끝마치면, 객체의 생명 주기를 관리하고 더 이상 필요는 객체를 해제하는 것을 명확히 하는 것과 같은 앱의 메모리를 관리해야하는 정보를 얻게 될 것이다. 

비록 메모리 관리는 보 객체간의 레벨에서 고려해야하지만, 우리의 목표는 사실 **object graphs**를 관리하는 것이다. 우리는 실제로 필요한 객체보다 더 많은 객체가 메모리에 있는 것을 원하지 않는다.

<img src="https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Art/memory_management_2x.png">

<br>

### 한눈에 보기

Objective-C는 앱의 메모리 관리를 위해 두가지 메소드를 제공한다.

1. 이 가이드에 명시된 메소드에 따르면, **"수동 유지-해제"(manual retain-release)** 또는 줄여서 **MRR** 메소드는, 우리가 소유한 객체를 계속 추적하는 방식으로 명시적으로 메모리를 관리한다.
2. **자동 참조 카운터(Automatic Reference Counting)**, 또는 줄여서 **ARC**는, MRR과 동일한 방식의 참조 카운팅 체계를 사용하지만, 컴파일시 적절한 메모리 관리 메소드 호출을 사용한다. 새로운 프로젝트를 만들 때에 ARC를 사용하도록 강력히 권장된다. ARC를 사용하면, 몇가지 상황에서는 도움이 될 수 있지만, 이 문서에 명시된 구현내용을 알필요가 없다. ARC에서 더 알고싶다면, [Transitioning to ARC Release Notes](https://developer.apple.com/library/archive/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011226) 이 문서를 보면 된다.

<br>

### Good Practices Prevent Memory-Related Problems

There are two main kinds of problem that result from incorrect memory management:

- Freeing or overwriting data that is still in use

  This causes memory corruption, and typically results in your application crashing, or worse, corrupted user data.

- Not freeing data that is no longer in use causes memory leaks

  A memory leak is where allocated memory is not freed, even though it is never used again. Leaks cause your application to use ever-increasing amounts of memory, which in turn may result in poor system performance or your application being terminated.

Thinking about memory management from the perspective of reference counting, however, is frequently counterproductive, because you tend to consider memory management in terms of the implementation details rather than in terms of your actual goals. Instead, you should think of memory management from the perspective of object ownership and object graphs.

Cocoa uses a straightforward naming convention to indicate when you own an object returned by a method.

See [Memory Management Policy](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmRules.html#//apple_ref/doc/uid/20000994-BAJHFBGH).

Although the basic policy is straightforward, there are some practical steps you can take to make managing memory easier, and to help to ensure your program remains reliable and robust while at the same time minimizing its resource requirements.

See [Practical Memory Management](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html#//apple_ref/doc/uid/TP40004447-SW1).

Autorelease pool blocks provide a mechanism whereby you can send an object a “deferred” `release` message. This is useful in situations where you want to relinquish ownership of an object, but want to avoid the possibility of it being deallocated immediately (such as when you return an object from a method). There are occasions when you might use your own autorelease pool blocks.

See [Using Autorelease Pool Blocks](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmAutoreleasePools.html#//apple_ref/doc/uid/20000047-CJBFBEDI).

<br>

### References

- [Advanced Memory Management Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html)