# List Operations in Python

## Overview
In Python, lists are mutable, ordered collections of items. They can contain elements of different types and support various operations.

## List Creation
### Method Signature:
```python
list_name = [item1, item2, item3]
```
### Example:
```python
my_list = [1, 2, 3, 4, 5]
```

## Common List Methods

### 1. append()
**Method Signature:** `list.append(item)`  
Appends an item to the end of the list.

**Example:**
```python
my_list.append(6)
# Result: [1, 2, 3, 4, 5, 6]
```

### 2. extend()
**Method Signature:** `list.extend(iterable)`  
Extends the list by appending elements from the iterable.

**Example:**
```python
my_list.extend([7, 8])
# Result: [1, 2, 3, 4, 5, 6, 7, 8]
```

### 3. insert()
**Method Signature:** `list.insert(index, item)`  
Inserts an item at a specified position.

**Example:**
```python
my_list.insert(0, 0)
# Result: [0, 1, 2, 3, 4, 5, 6, 7, 8]
```

### 4. remove()
**Method Signature:** `list.remove(item)`  
Removes the first occurrence of a specified item.

**Example:**
```python
my_list.remove(0)
# Result: [1, 2, 3, 4, 5, 6, 7, 8]
```

### 5. pop()
**Method Signature:** `list.pop([index])`  
Removes and returns an item at the specified index (last item if index is not provided).

**Example:**
```python
last_item = my_list.pop()  
# Result: [1, 2, 3, 4, 5, 6, 7]  
# last_item = 8
```

### 6. index()
**Method Signature:** `list.index(item[, start[, end]])`  
Returns the index of the first occurrence of the specified item.

**Example:**
```python
index_of_3 = my_list.index(3)
# Result: index_of_3 = 2
```

### 7. count()
**Method Signature:** `list.count(item)`  
Returns the number of occurrences of a specified item.

**Example:**
```python
count_of_3 = my_list.count(3)
# Result: count_of_3 = 1
```

### 8. sort()
**Method Signature:** `list.sort(key=None, reverse=False)`  
Sorts the items of the list in place.

**Example:**
```python
my_list.sort(reverse=True)
# Result: [7, 6, 5, 4, 3, 2, 1]
```

### 9. reverse()
**Method Signature:** `list.reverse()`  
Reverses the elements of the list in place.

**Example:**
```python
my_list.reverse()
# Result: [1, 2, 3, 4, 5, 6, 7]
```

### 10. clear()
**Method Signature:** `list.clear()`  
Removes all items from the list.

**Example:**
```python
my_list.clear()
# Result: []
```

## Conclusion
These are some of the commonly used operations for Python lists. Understanding these methods will help you manipulate lists efficiently in your Python programs.