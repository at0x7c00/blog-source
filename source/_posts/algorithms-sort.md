---
title: 算法学习之排序
date: 2017-02-23
tags: 
- 算法
- 排序
categories: 算法
---



# Sort
# Insert sort

```java
	public void doSort(Comparable[] items) {
		for(int i = 1; i<items.length; i++){
			for(int j = i;j > 0 && less(items[j],items[j-1]); j--){
				exchange(items, j, j - 1); 
			}
		}
	}
```

![Alt Insert sort](https://github.com/xooxle/algs4/raw/master/sort/me/huqiao/algs4/sort/pic/insert-sort.gif "Insert sort")

# Select sort

```java

	public void doSort(Comparable[] items) {
		for(int i = 0; i<items.length; i++){
			int minIndex = i;
			for(int j = i + 1;j<items.length; j++){
				if(less(items[j],items[minIndex])){
					minIndex = j;
				}
			}
			exchange(items, i, minIndex);
		}
	}
```


![Alt Select sort](https://github.com/xooxle/algs4/raw/master/sort/me/huqiao/algs4/sort/pic/select-sort.gif "Select sort")

# Shell sort

```java
	public void doSort(Comparable[] items) {
		int N = items.length;
		int h = 1;
		while(h<N) h = h*3 + 1;
	    
		while(h>0){
			for(int i = 0;i<N;i++){
				List<Integer> highlight = new ArrayList<Integer>();
				for(int j = i;j>=h && less(items[j],items[j - h]); j-=h){
					exchange(items,j,j - h);
				}
			}
			h/=3;
		}
	}
```

![Alt Shell sort](https://github.com/xooxle/algs4/raw/master/sort/me/huqiao/algs4/sort/pic/shell-sort.gif "Shell sort")

# Merge sort

```java

	public void doSort(Comparable[] items) {
		aux = new Comparable[items.length];
		sort(items,0,items.length  - 1);
	}
	
	private void sort(Comparable[] items,int lo,int hi){
		if(hi<=lo) return;
		int mid = lo + (hi - lo) / 2;
		sort(items,lo,mid);
		sort(items,mid + 1,hi);
		merge(items,lo,mid,hi);
	}

	private void merge(Comparable[] items,int lo,int mid,int hi){
		int i = lo,j = mid + 1;
		for(int k = lo; k <= hi; k++){
			aux[k] = items[k];
		}
		for(int k = lo; k <= hi; k++){
			if(i > mid){
				items[k] = aux[j];
				j++;
			}else if(j > hi){
				items[k] = aux[i];
				i++;
			}else if(less(aux[i] , aux[j])){
				items[k] = aux[i];
				i++;
			}else{
				items[k] = aux[j];
				j++;
			}
		}
	}

````

![Alt Merge sort](https://github.com/xooxle/algs4/raw/master/sort/me/huqiao/algs4/sort/pic/merge-sort.gif "Merge sort")

# Quick sort

```java

	public void doSort(Comparable[] items) {
		sort(items,0,items.length - 1);
	}
	
	private void sort(Comparable[] items,int lo,int hi){
		if(hi<=lo) return;
		int j = paratition(items,lo,hi);
		sort(items,lo,j);
		sort(items,j+1,hi);
	}

	private int paratition(Comparable[] items, int lo, int hi) {
		
		//lo mark
		int i = lo;
		//hi mark
		int j = hi + 1;
		Comparable v = items[lo];
		
		while(true){
			
			//find an item bigger than v
			while(less(items[++i],v)) if(i==hi) break;
			//find an item smaller than v
			while(less(v,items[--j])) if(j==lo) break;
			
			// when lo mark meet hi mark then stop all(sort complate)
			if(i>=j) break;
			
			//exchange smaller item and bigger item from left to right
			exchange(items, i, j);
			
		}
		exchange(items, lo, j);
		return j;
	}
	

```

![Alt Quick sort](https://github.com/xooxle/algs4/raw/master/sort/me/huqiao/algs4/sort/pic/quick-sort.gif "Quick sort")

# Note
Video record by [Bandicam](http://www.bandicam.com/). Use [格式工厂](http://www.pcfreetime.com/) to convert MP4 file to GIF picture.

Code in github:[algs4](https://github.com/xooxle/algs4).