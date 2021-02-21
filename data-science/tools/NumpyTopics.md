#NumPy 

### Installing NumPy
`conda install numpy`
or
`pip install numpy`

### How to import NumPy
`import numpy as np`

### What’s the difference between a Python list and a NumPy array?
NumPy gives you an enormous range of fast and efficient ways of creating arrays and manipulating numerical data inside them. While a Python list can contain different data types within a single list, all of the elements in a NumPy array should be homogeneous. The mathematical operations that are meant to be performed on arrays would be extremely inefficient if the arrays weren’t homogeneous.

### Why use NumPy?

NumPy arrays are faster and more compact than Python lists. An array consumes less memory and is convenient to use. NumPy uses much less memory to store data and it provides a mechanism of specifying the data types. This allows the code to be optimized even further.

### What is an array?

`dtype` - the type of elements
`rank` - the number of dimensions
`shape` - a tuple of integers giving the size of the array along each dimension
`ndarray` - n dimensional array
`vector` - 1 dimensional array
`matrix` - 2 dimensional array
`tensor` - 3 or more dimensional array

In NumPy, dimensions are called axes.

### How to create a basic array?

np.array(), np.zeros(), np.ones(), np.empty(), np.arange(), np.linspace(), dtype


### Adding, removing, and sorting elements

np.sort(), np.concatenate()

### How do you know the shape and size of an array?

ndarray.ndim, ndarray.size, ndarray.shape

### Printing Array

When you print an array, NumPy displays it in a similar way to nested lists, but with the following layout:

* the last axis is printed from left to right,
* the second-to-last is printed from top to bottom,
* the rest are also printed from top to bottom, with each slice separated from the next by an empty line.



### Basic array operations



P