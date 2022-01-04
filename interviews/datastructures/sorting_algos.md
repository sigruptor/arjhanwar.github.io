# Sorting Algorithms

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
