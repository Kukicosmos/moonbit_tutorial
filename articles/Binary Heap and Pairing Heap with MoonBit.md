# How to Implement Binary Heap and Pairing Heap with MoonBit?

**The concept of binary heap was introduced in 1964 by J.W.J Williams' *Algorithm 232 - Heapsort*.** Over the following decades, people discovered that beyond general sorting (such as priority task scheduling, graph algorithms, etc.), this data structure can be applied to more settings, and various variants have been extended. This article will explore the principles of binary heaps implemented using arrays and pairing heaps implemented using lists, and how to implement them using MoonBit (a Rust-like language and toolchain optimized for WebAssembly experience).

(website here: [https://www.moonbitlang.com/](https://www.moonbitlang.com/))

## 01 **Basic Structure and Usage of Heap**

**What comes to mind when talking about "heap"?**

The name "heap" is often related to a discontinuous program data storage area contrasting with stacks, but just like stacks, in the realm of data structures, "heap" holds a radically different definition. According to the earliest literature on “heap” from 1964, it is mentioned right at the beginning that heap sort is an improved version of tree sort, so **the logical structure of a heap is a tree**.

![image](https://github.com/Kukicosmos/moonbit_tutorial/blob/main/image/image_binary/heap.png)

https://www.happycoders.eu/algorithms/heapsort/

In a heap, the value stored in any node is greater or smaller than the values of its child nodes, ensuring that any subtree in the heap also meets this condition. For users, using a heap is akin to using a "queue".

```rust
let h = MinHeap::new()
```

Users can add elements to the heap using the `insert` method:

```rust
h.insert(6)
h.insert(4)
h.insert(13)
```

You can also use `pop` to remove an element, but each time it is the smallest element in the current heap that pops from the heap, not necessarily in the order they were inserted.

```rust
h.pop() // the result should be 4.
```

In these examples, the elements stored in the heap are integers. However, in reality, the heap only requires elements to be comparable. In MoonBit, the comparison operators such as `>` and `<` are methods under the `Compare` interface. Any type that implements this interface can be used to store elements in the heap.

## 02 **Array Implementation of Binary Heap**

Here, we are implementing a max heap, where the root element of the heap is the largest.

**The logical structure of a binary heap is a nearly complete binary tree (commonly termed as a "complete binary tree").**

Suppose there is an n-level binary heap, where the first n-1 levels are full, and the nodes on the nth level tend to fill from left to right. The diagram below illustrates a binary heap `arr` with 6 elements, which can be stored in an array of length 6. The positions of each node in the corresponding array are marked in the diagram.

![image](https://github.com/Kukicosmos/moonbit_tutorial/blob/main/image/image_binary/arr.png)

The online visualization display page of a binary heap can provide a clearer demonstration: [https://visualgo.net/en/heap?slide=1](https://visualgo.net/en/heap?slide=1)

Due to the property mentioned earlier, each node can be assigned a specific identifier (used as an array index), and by using this identifier, we can find its left and right child nodes. Consequently, the entire binary heap can be directly stored in an array.

On an array indexed starting from 1, the left child node of node i is at position 2i, the right child node is at position 2i+1, and the parent node is at position i/2 rounded down.

```rust
fn parent(self : Int) -> Int {
    self / 2
  }
  
  fn left(self : Int) -> Int {
    self * 2
  }
  
  fn right(self : Int) -> Int {
    self * 2 + 1
  }
  
  // Assuming you want to obtain the array index corresponding to the parent node of node i, you can use i.parent().
```

In MoonBit, arrays are indexed starting from 0. To maintain formal consistency, the position initially indexed as 0 is left unused, and in heap-related operations, 1 is used as the index start.

In the `BHeap::new` function for creating a heap, to ensure that the actual heap capacity matches the parameter, the array length is incremented by one when creating the array. Consequently, the `capacity()` function, which retrieves the heap's capacity, subtracts 1 from the array length.

> The built-in Array[T] in MoonBit is of fixed length, so a struct with an actual element count and an array needs to be established.
> 

```rust
struct BHeap[T] {
    mut data : Array[T]
    mut count : Int
    min_value : T
  }
  
  fn BHeap::new[T : Compare](capacity : Int, minValue : T) -> BHeap[T] {
    { data: Array::make(capacity + 1, minValue), count: 0, min_value: minValue }
  }
  
  fn capacity[T](self : BHeap[T]) -> Int {
    self.data.length() - 1
  }
```

> Creating an array requires default values, and it is recommended to fill it with the minimum value of type T.
> 

**The next step is to implement two related operations of the binary heap: insertion and popping.**

Inserting elements into an empty binary heap is straightforward; just place the element at `data[1]`. However, when inserting more elements, how do we find the appropriate position by comparing their sizes?

A widely adopted approach is to first place the new element to be inserted at the end of the array - essentially finding the leftmost available position in the last level of the binary tree and designating it as a leaf node. If the bottom level is already full, a new level is started. The diagram below illustrates inserting the new element 20 into the right child node:

![image]https://github.com/Kukicosmos/moonbit_tutorial/blob/main/image/image_binary/tree_0.png

```rust
fn insert[T : Compare](self : BHeap[T], x : T) {
    ......
    self.data[self.count + 1] = x
    self.count = self.count + 1
    self.heap_fix(self.count)
    ......
}
```

This action has a high probability of violating the heap property. We have no idea about the size relationship between this new element and the randomly assigned parent node. However, although the heap property is disrupted, the previous heap elements still adhere to the rule of having the maximum element at the parent node. Repairing it based on this rule is not so difficult.

Suppose the array index corresponding to the new element is stored in the variable `i`. Then:

- First, check if `i` is equal to 1. If it is, do nothing - there is no parent node. Since we are not using index 0, `i` not being equal to 1 can also be expressed as `i > 1`.
- If `i` is not equal to 1, compare it with its parent node. If it is greater than the value of the parent node, `exchange` it with the parent node. In the diagram below, the newly inserted element 20 is greater than its parent node element 17, so 17 is moved downwards to allow 20 to "float up."
- Assign `i` to `i.parent()` because this is the new position of the element.
- Repeat the above process until the parent node at the new position of the element is greater than it or until it reaches the top of the heap.

![image](https://github.com/Kukicosmos/moonbit_tutorial/blob/main/image/image_binary/tree.png)

```rust
fn heap_fix[T : Compare](self : BHeap[T], i : Int) {
    var i = i
    while i > 1 && self.data[i] > self.data[i.parent()] {
      self.data.exchange(i, i.parent()) // Exchange the positions of the elements.
      i = i.parent() // Mark the index corresponding to the inserted element now.
    }
  }
  
  fn exchange[T](self : Array[T], x : Int, y : Int) {
    let tmp = self[x]
    self[x] = self[y]
    self[y] = tmp
  }
```

**Another important operation is popping the top element of the heap.** Following the strategy of altering first and then rebalancing, we first swap the first and last elements of the array, then decrement the `count` variable by one. In the diagram below, the original top element of the heap is 45, and the element at the heap's end is 9. Now, by swapping their positions and decrementing the `count` , we remove 45.

![image](https://github.com/Kukicosmos/moonbit_tutorial/blob/main/image/image_binary/tree_2.png)

The process of deletion can be viewed as gradually "sinking" the element that has been swapped to the top of the heap to a suitable position. This step is completed through the `heapify` function.

Suppose the index of the element swapped to the top of the heap is stored in the variable `i`, and each comparison is done using the variable `s`, which initially holds the index of the node with the maximum value.

- First, find the indices of the left and right child nodes, named `l` and `r`.
- Check if they are out of bounds - the indices calculated by `left` and `right` might not actually store elements or even exceed the current heap capacity.
- If they are within bounds, compare the values of `data[l]` and `data[s]`, assigning the index of the larger value to `s`.
- If they are within bounds, compare the values of `data[r]` and `data[s]`, assigning the index of the larger value to `s` (as `l` and `i` have already been compared; at this point, `s` should hold the index of the larger value between `l` and `i`).
- Check if `s` is equal to `i`. If so, it means that `data[i]` is larger than both `data[l]` and `data[r]`. Considering the property of the subheap hasn't been violated, `data[i]` is already larger than all the elements in the subtree, so terminate the loop and exit.
- If `s` is not equal to `i`, swap the values of `s` and `i`, because the original element's position has shifted downward. Then assign `i` to `s` and continue the loop. In the example below, after comparing the newly inserted element 9 with its left and right child nodes, the largest value 36 is swapped to the top of the heap.

![image](https://github.com/Kukicosmos/moonbit_tutorial/blob/main/image/image_binary/tree_3.png)

The MoonBit code implementation corresponding to the process described above is as follows:

```
fn pop[T : Compare](self : BHeap[T]) -> Option[T] {
  if self.count == 0 {
    return None
  }
  let x = self.data[1] // Save the top element of the heap.
  let n = self.count
  self.data.exchange(1, n)
  self.count = self.count - 1 // Remove the original top element of the heap exchanged to the end of the array.
  if self.count > 0 {
    self.heapify(1)
  }
  return Some(x)
}

fn heapify[T : Compare](self : BHeap[T], index : Int) {
  let n = self.count
  var i = index
  var s = i
  while true {
    let l = i.left()
    let r = i.right()
    if l <= n && self.data[l] > self.data[s] {
      s = l
    }
    if r <= n && self.data[r] > self.data[s] {
      s = r
    }
    if s != i {
      self.data.exchange(s, i)
      i = s
    } else {
      break
    }
  }
}
```

With these two basic operations, it becomes quite straightforward to implement generic array sorting and selecting the largest k elements. However, some boundary condition handling, such as resizing when capacity is exhausted, is not explicitly written in the `insert` and `delete`和`delete` functions. The complete code is shared here: [try.moonbitlang.com/#421122d5](https://try.moonbitlang.com/#421122d5).

In situations with small data sets and frequent element insertions and deletions, binary heaps perform quite well, as mentioned by Poul-Henning Kamp in his article "You’re Doing It Wrong: Think you’ve mastered the art of server performance? Think again."

> However, when dealing with larger data sets and memory constraints, using a multi-way heap would be preferable.
> 

![image](https://github.com/Kukicosmos/moonbit_tutorial/blob/main/image/image_binary/Kamp.png)

The binary heap implemented above exhibits a clear imperative style, while MoonBit, as a multi-paradigm language, also provides good support for functional programming. Next, I will use immutable data structures such as List and enumerations in MoonBit to implement a pairing heap.

## 03 Pairing Heap

Implemented here is a min-heap.

Pairing heap is a type of heap based on a multiway tree. It is simple to implement and offers superior performance. It is used to implement priority queues in the GNU libstdc++ library. The complexity analysis of its operations presents challenges, which will not be covered in this article.

![image](https://github.com/Kukicosmos/moonbit_tutorial/blob/main/image/image_binary/pairing_heap.png)

[https://brilliant.org/wiki/pairing-heap/](https://brilliant.org/wiki/pairing-heap/)

Since the structure of a pairing heap is a more general multiway tree, here we use an `enum` to define `PHeap`. A `PHeap` is either an empty tree or a node containing one element and N subtrees. This concept can be naturally expressed through an enumeration type.

```rust
enum PHeap[T] {
    Empty
    Node(T, List[PHeap[T]])
  }
```

In the context of pairing heaps, it will be convenient to define the merge operation for two heaps first. By pattern matching, we can clearly describe the logic of merging:

- If one of h1 and h2 is empty, then keep the other one.
- If both are not empty, then compare the top elements of each heap and put the heap with the larger element into the list of the other heap.

```rust
fn merge[T : Compare](h1 : PHeap[T], h2 : PHeap[T]) -> PHeap[T] {
    match (h1, h2) {
      (Empty, h2) => h2
      (h1, Empty) => h1
      (Node(x, ts1), Node(y, ts2)) => if x < y {
        Node(x, Cons(Node(y, ts2), ts1))
      } else {
        Node(y, Cons(Node(x, ts1), ts2))
      }
    }
  }
  
```

Insertion can be seen as merging a single-element heap into the original heap:

```rust
fn insert[T : Compare](self : PHeap[T], x : T) -> PHeap[T] {
    merge(self, Node(x, Nil))
  }
```

Popping the top element from the heap requires folding the entire list of heaps using `merge`, employing the `consolidate` function. It's worth noting that the  `consolidate`  function effectively implements a two-stage consolidation through recursion. First, it merges the pairs of heaps in the list, and then it merges these newly generated heaps together. This is manifested in the nested `merge` function calls in the code.

```rust
fn consolidate[T : Compare](ts : List[PHeap[T]]) -> PHeap[T] {
    match ts {
      Nil => Empty
      Cons(t, Nil) => t
      Cons(t1, Cons(t2, ts)) => merge(merge(t1, t2), consolidate(ts))
    }
  }
  
  fn pop[T : Compare](self : PHeap[T]) -> Option[PHeap[T]] {
    match self {
      Empty => None
      Node(_, ts) => Some(consolidate(ts))
    }
  }
```

The use of `match` and enumeration types makes the operations related to pairing heaps very concise. The complete code can be found here: [try.moonbitlang.com/#a2f1dd62](https://try.moonbitlang.com/#a2f1dd62).

From the definition of pairing heaps and their operations, it can be observed that they do not extensively retain additional information such as tree size, depth, or rank in their structure. This allows them to avoid the poor constants associated with data structures like Fibonacci heaps, earning them a reputation for efficiency, simplicity, and flexibility in practice.
