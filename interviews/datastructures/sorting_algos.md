# Sorting Algorithms

Few Common Terms
1. **Stability** : Stable Algorithm maintains the relative order of the items with same (equal) sort keys.
2. **Adaptive** : An algorithm is adaptive if it takes advantage of already sorted elements. These algorithm takes minimum time over already sorted list. For example, if input is already (nearly) sorted then, go for Insertion sort as the time complexity will be O(n). 
	
	**_Adaptive Sorting Algorithms_**
	- InsertionSort
	- BubbleSort

	**_Non-Adaptive Sorting Algorithms_**
	- MergeSort
	- HeapSort
	- SelectionSort
	- QuickSort

### Insertion Sort
- Loop Invariant arr[1..i] is always sorted.
- Inserting an element into its correct position wrt 1..i such that arr[1..i] is always sorted.
```
for (int i=i;i<n;i++) {
	int k=i-1;
	int key=a[k];
	while (k>=0 && a[k]>a[k+1]) {
		swap(a,k+1, k);
		k--;
	}
	a[k+1]=key;
}
```

- [Animation](https://www.toptal.com/developers/sorting-algorithms/insertion-sort)

##### Complexity
**TimeComplexity** -> O(n^2). Used when data is nearly sorted or when data size is small.

##### Properties
- Stable
- O(1) extra space
- O(n^2) comparisons and swaps
- O(n) when nearly sorted
- Adaptive

### Quick Sort
- Divide and conquer algorithm
- Relies on partition algorithm which picks a pivot and puts it into its corresponding position. Quicksort then sorts left and right side of the paritions recurisively.

```
quickSort(a,start,end) {
   int pivot = partition(a,start,end);
   quickSort(a,start, pivot-1)
   quickSort(a, pivot+1, end);
}

partition(a, start, end) {
   int pivot=a[end-1]; // choosing the last element
   int low=start-1;
   for (int j=start;j<end-1;j++) {
   	if (a[j]<pivot) {
	   low++;
	   swap(a,low,j);
	}
   }
   low++;
   swap(a,low,end-1); // puts the pivot into its right position
   return low;
}
```

##### Complexity
**TimeComplexity** 
- Best Case: When partition is balanced i.e (n/2, n/2(n/2-1) O(nlogn)
- Worst Case: parition is totally unbalanced (1, n-1) -> O(n^2)

**Space Complexity**
- Worst Case: O(n) -> when partition is unbalanced happens when input is sorted.
- Average Case: O(logn)

##### Properties
- Not Stable
- In Place
- Not Adaptive

### Merge Sort
- Classic divide and conquer problem, split array into 2 recursively, merge them.
- Very predictable and only stable O(nlogn) algorithm.
- Takes between .5log(n) and log(n) comparisons and between log(n) and 1.5 log(n) swaps per element. 
- Minima for already sorted data and maxima for random data.
- If O(n) extra space is not a concern, this is an excellent choice and used mostly for external sort

```
mergeSort(a[], start, end) {
   if (end > start) {
     int mid = (start+end)/2;
     mergeSort(a, start, mid-1);
     mergeSort(a, mid, end);
     merge(a, start, mid, end);
   }
}

merge(a,start,mid,end) {
  int temp[end-start+1];
  i=start;j=mid+1;
  k=0;
  
  while(i<=mid && j<=end) {
  	if (a[i]<a[j]) {
		temp[k++]=a[i++]  
	} else {
		temp[k++]=a[j++]
	}
  }
  
  // copy remaining i.ss
  while(i<=mid) {
  	temp[k++]=a[i++];
  }
  // copy remaining j.ss
  while(j<=end) {
  	temp[k++]=a[j++];
  }
  
  //copy temp to a
  for (int i=0;i<end-start+1;i++) {
  	a[i]=temp[i];
  }
}
```

##### Complexity
**TimeComplexity**
- O(nlog(n)) -> min for already sorted data

**Space Complexity**
- O(n) - extra space
- Extra O(log(n)) - for linked list (recursions)

##### Useful
- Stability is needed
- Sorting of linked lists
- And when random access is much more expensive than sequential access (external sorting on tape)
- There do exist linear time in-place merge algorithms for the last step of the algorithm, but they are both expensive and complex. The complexity is justified for applications such as external sorting when Θ(n) extra space is not available.

##### Properties
- Stable
- O(n) extra space for arrays
- Requires O(logn) extra space for linked list (recursion)
- Not adaptive
- Does not require random access to data


### Heap Sort
- Heapify n/2 elements, highest element at the top of the heap
- A heap is a binary tree in which each node has a smaller key than its children; this property is called the heap property or heap invariant.


```
heapSort(a) {
	for (int i=a.length/2;i>=0;i--) {
		heapify(a,i);
	}
	
	for (int i=a.length-1;i>=0;i--) {
		swap(a,i,0);
		heapify(a,0);
	}
}

heapify(a, i) {
	left = 2*i+1;
	right = 2*i+2;
	largest = i;
	
	// if left child is greater than largest
	if (left < n && a[left]> a[largest]) {
		largest = left;
	}
	
	// if right child is greater
	if (right<n && a[right]>a[largest]) {
		largest = right;
	}
	
	if (i!=largest) {
		swap(a,i,largest)
		heapify(a,largest)
	}
}
```

##### Complexity
**Time Complexity**
-  O(nlogn) for all cases (best,worst, avg)

**Space Complexity**
-  O(lg(n)) space for the recursive call stack. However, the tail recursion in heapify() is easily converted to iteration, which yields the O(1)

#### Useful
- When smallest or largest value is needed instantly
- Finding the order in statistics, 
- Dealing with priority queues in MST algorithms - Prim’s algorithm and Huffman encoding or data compression
- Guaranteed O(nlogn) performance
- We get a partially sorted array if Heapsort is somehow stopped abruptly

##### Properties
- Not stable
- Not really adaptive (Slightly)
- In the nearly sorted case, the heapify phase destroys the original order. In the reversed case, the heapify phase is as fast as possible since the array starts in heap order, but then the sortdown phase is typical
- Cache inefficient
- Slow as compared to QuickSort
