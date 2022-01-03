## Sorting Algorithms

#### Insertion Sort
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
