> 本文列举几种常用的排序方法，按照推荐程度从低到高的顺序排列，即越后面排序方法的越推荐

### 选择排序

|空间复杂度|时间复杂度|是否稳定|
|--|--|--|
|O(1)|O(n²)|否|

js实现代码：

```js
function selectionSort (arr) {
  if (!arr || !arr.length) return

  for (let i = 0; i < arr.length - 1; i++) {
    let minIndex = i
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[j] < arr[minIndex]) {
        minIndex = j
      }
    }
    if (minIndex !== i) {
      const tmp = arr[i]
      arr[i] = arr[minIndex]
      arr[minIndex] = tmp
    }
  }

  return arr
}

// let arr = [6, 2, 5, 4, 3, 9, 7, 1]
// console.log(selectionSort(arr))
```

### 冒泡排序

|空间复杂度|时间复杂度|是否稳定|
|--|--|--|
|O(1)|O(n²)|是|

js实现代码：

```js
function bubbleSort (arr) {
  if (!arr || !arr.length) return

  for (let i = arr.length - 1; i > 0; i--) {
    for (let j = 0; j < i - 1; j++) {
      if (arr[j] > arr[j + 1]) {
        const tmp = arr[j]
        arr[j] = arr[j + 1]
        arr[j + 1] = tmp
      }
    }
  }

  return arr
}
// let arr = [6, 2, 5, 4, 3, 9]
// console.log(bubbleSort(arr))
````
### 插入排序排序

|空间复杂度|时间复杂度|是否稳定|
|--|--|--|
|O(1)|O(n²)|是|

js实现代码：

```js
function insertionSort (arr) {
  if (!arr || !arr.length) return

  for (let i = 1; i < arr.length; i++) {
    let j = i - 1
    let value = arr[i]
    for (; j >= 0; j--) {
      if (arr[j] > value) {
        arr[j + 1] = arr[j]
      } else {
        break
      }
    }
    arr[j + 1] = value
  }

  return arr
}
// let arr = [6, 2, 5, 4, 3, 9, 7]
// console.log(insertionSort(arr))
````

### 归并排序

|空间复杂度|时间复杂度|是否稳定|使用思想|
|--|--|--|--|
|O(n)|O(nlogn)|是|分治|

js实现代码：

```js

function mergeSort (arr) {
  if (!arr || !arr.length) return

  function merge (arr, start, middle, end) {
    let tmp = []
    let i = start
    let j = middle + 1
    // 按大小插入数据到临时数组
    while (i <= middle && j <= end) {
      tmp.push(arr[i] < arr[j] ? arr[i++] : arr[j++])
    }
    let from = i
    let to = middle
    if (i > middle) {
      from = j
      to = end
    }
    // 剩余的数据插入到临时数组末尾
    for (let k = from; k <= end; k++) {
      tmp.push(arr[k])
    }
    // 把排好序的数据拷贝回原数组
    for (let l = start, m = 0; l <= end; l++) {
      arr[l] = tmp[m++]
    }
    tmp = null
  }

  function sort (arr, start, end) {
    if (start < end) {
      const middle = Math.floor((start + end) / 2)
      sort(arr, start, middle)
      sort(arr, middle + 1, end)
      merge(arr, start, middle, end)
    }
  }

  sort(arr, 0, arr.length - 1)
  return arr
}

// let arr = [12, 2, 0, 4, 3, 9, 7, 1]
// console.log(mergeSort(arr))
````

### 快速排序

|空间复杂度|时间复杂度|是否稳定|使用思想|
|--|--|--|--|
|O(1)|O(nlogn)，最差O(n²)|否|分治|

js实现代码：

```js
function quickSort (arr) {
  if (!arr || !arr.length) return
  // 拆分为两部分
  function partition (arr, start, end) {
    let i = start
    let pivot = arr[end]
    let tmp
    for (let j = start; j < end; j++) {
      if (arr[j] < pivot) {
        tmp = arr[i]
        arr[i] = arr[j]
        arr[j] = tmp
        i++
      }
    }
    arr[end] = arr[i]
    arr[i] = pivot
    return i
  }
  function sort (arr, start, end) {
    if (start > end) return
    const i = partition(arr, start, end)
    sort(arr, start, i - 1)
    sort(arr, i + 1, end)
  }
  sort(arr, 0, arr.length - 1)
  return arr
}
// let arr = [6, 2, 5, 4, 3]
// console.log(quickSort(arr))
```
