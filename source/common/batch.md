# Batch

Class `Batch` is the internal data structure with the characteristics of both `dict` and `tensor`.

It is designed to store the multi-types (`numpy.ndarray`, `torch.Tensor`, etc.) of data obtained from the interaction with the environment.

Meanwhile, it is used to package the data for transfer between clients and servers as well.

Note: we design the `Batch` with reference to [https://github.com/thu-ml/tianshou](https://github.com/thu-ml/tianshou)

## Class `Batch`

## Args

### `batch_dict`

type: `dict` or `Batch` or `BatchSet`(details refer to the code), optional

The  whole data to create the `Batch` instance, defaults to `None`.

### `init_type`

type: `str`

Set the construction mode, must be `'whole_data'` or `'sample_data'`, defaults to `'whole_data'`.

The differences between the two modes can be found in the next part **How to construct a new `Batch` instance**.

### `max_size`

type: `int`, optional

The max size of the instance constructed from the `sample_data`, defaults to `None`.

It is required when the mode is `sample_data`.

### `sample_data`

type: `dict` or `Batch`, optional

The sample data to create the instance, defaults to `None`.

It is required when the mode is `sample_data`.

### `unsqueeze`

type: `bool`, optional

Whether to expand the first dimension when creating a `Batch` instance given `max_size` & `sample_data`, defaults to `None`(in the `'sample_data'` mode, `None` equals to `False`).

It is required when the mode is `sample_data`.

### `copy`

type: `bool`

Whether to deepcopy the given data, defaults to `False`.

### `**kwargs`

Use this variable to construct the `Batch` in a more flexible way.

Please refer to the next part **How to construct a new `Batch` instance**.

## Methods

### `createEmptyBatch()`

`createEmptyBatch(self, max_size: int, sample_data: Union[dict, Batch], unsqueeze=False) -> None`

Create a new empty batch with the given sample_data.

**Example:**

```python
>>> batch = Batch(init_type='sample_data')
```

## In-place Methods

### `__iadd__()`

`__iadd__(self, other: Union[Batch, Number, np.number]) -> Batch`

Algebraic addition with another Batch instance in-place.

### `__imul__()`

`__imul__(self, value: Union[Number, np.number]) -> Batch`

Algebraic multiplication with a scalar value in-place.

### `__itruediv__()`

`__itruediv__(self, value: Union[Number, np.number]) -> Batch`

Algebraic division with a scalar value in-place.

## Out-of-place Methods

### `__add__()`

`__add__(self, other: Union[Batch, Number, np.number]) -> Batch`

Algebraic addition with another Batch instance out-of-place.

Here the func utilizes the copy.deepcopy() since the Batch is based on the dict which will be changed when as a input to a certain function.

### `__mul__()`

`__mul__(self, value: Union[Number, np.number]) -> Batch`

Algebraic multiplication with a scalar value out-of-place.

Here the func utilizes the copy.deepcopy() since the Batch is based on the dict which will be changed when as a input to a certain function.

### `__truediv__()`

`__truediv__(self, value: Union[Number, np.number]) -> Batch`

Algebraic division with a scalar value out-of-place.

Here the func utilizes the copy.deepcopy() since the Batch is based on the dict which will be changed when as a input to a certain function.

## Basic Usages

## How to construct a new `Batch` instance

There are two modes of the construction of `Batch` . One is to create an new `Batch` instance of a specified size from a given `sample_data`, and the other is to create it directly with all the data.

### Mode `init_type = 'sample_data'`

The arguments `max_size, sample_data, unsqueeze` are required in this case.

The construction of `Batch` will return a new instance with size=`max_size` which contains the keys in `sample_data` and all the values will be the default value(0, None, etc.). When the argument `unsqueeze = True`, all the values in the new instance will be expanded in the first dimension.

Note: all the values in the `sample_data` must can be converted to an `array`.

- **Construct Batch from the given `sample_data`**

    ```python
    >>> # unsqueeze = False
    >>> batch = Batch(init_type='sample_data',
    					        max_size=3,
    					        sample_data={'a': torch.Tensor([1, 2]),
    					                 		 'b': np.array([[1, 2]]),
    														 	 'c': [1, 2]})
    >>> print(batch)
    Batch(
        a: tensor([0., 0., 0.]),
        b: array([[0, 0],
                  [0, 0],
                  [0, 0]]),
        c: array([0, 0, 0]),
    )
    
    #----------------------------------------------------------------
    >>> # unsqueeze = True
    >>> batch = Batch(init_type='sample_data',
    					        max_size=3,
    					        sample_data={'a': torch.Tensor([1, 2]),
    					                 		 'b': np.array([[1, 2]]),
    														 	 'c': [1, 2]},
    									unsqueeze=True)
    >>> print(batch)
    Batch(
        a: tensor([[0., 0.],
                   [0., 0.],
                   [0., 0.]]),
        b: array([[[0, 0]],
    
                  [[0, 0]],
    
                  [[0, 0]]]),
        c: array([[0, 0],
                  [0, 0],
                  [0, 0]]),
    )
    ```

### Mode `init_type = `whole_data'`

As opposed to the previous, here b is used only as a storage carrier for the given data `batch_dict` or `**kwargs`.

Note: By default, `Batch` only keeps the reference to the given data, but it can alse support data copying by set argument `copy = True`.

- **Construct Batch from given data**

    ```python
    >>> # directly passing a dict object
    >>> batch = Batch({'a': torch.Tensor([1, 2]),
    								   'b': np.array([[1, 2]]),
    									 'c': [1, 2],
    									 'd': 1,
    									 'e': '12',
    									 'f': {'f1': 1}})
    >>> print(batch)
    Batch(
        a: tensor([1., 2.]),
        b: array([[1, 2]]),
        c: array([1, 2]),
        d: 1,
        e: '12',
        f: Batch(
               f1: 1,
           ),
    )
    
    #----------------------------------------------------------------
    >>> # passing a list/tuple of dict/Batch (i.e. BatchSet)
    >>> # the missing keys will be filled with the default value
    >>> batch = Batch([{'a': 1.0, 'b': np.array([1])},
    									 {'a': 2.0, 'b': np.array([2]), 'c': 'hello'}])
    >>> print(batch)
    Batch(
    	  a: array([1., 2.]),
        b: array([[1],
                  [2]]),
    		c: array([None, 'hello'], dtype=object),
    )
    
    #----------------------------------------------------------------
    >>> # construct a Batch with keyword arguments
    >>> batch = Batch(a=torch.Tensor([1]), b=np.array([1]), c=[None])
    >>> print(batch)
    Batch(
        a: tensor([1.]),
        b: array([1]),
        c: array([None], dtype=object),
    )
    
    #----------------------------------------------------------------
    >>> # conbining keyword arguments and batch_dict works fine as well
    >>> batch = Batch({'a': torch.Tensor([1]), 'b': np.array([1])}, c=[None])
    >>> print(batch)
    Batch(
        a: tensor([1.]),
        b: array([1]),
        c: array([None], dtype=object),
    )
    ```

## Data Manipulation
