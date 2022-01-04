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
