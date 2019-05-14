### 冒泡排序

### 快速排序

js代码：

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
    const i = partition(arr, start, end)
    if (i - start > 1) {
      sort(arr, start, i - 1)
    }
    if (end - i > 1) {
      sort(arr, i + 1, end)
    }
  }
  sort(arr, 0, arr.length - 1)
  return arr
}
// let arr = [6, 2, 5, 4, 3]
// console.log(quickSort(arr))
```
空间复杂度：O(n)

时间复杂度：O(nlogn)，最差O(n²)
