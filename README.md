HDFStore
========
High level interface to `PyTables` for reading and writing structures to disk

This piece of code allows you to work with HDF5 files either as a (key, value)
storage or as a classic tables.file.File object.

Persistent Dictionary usage
---------------------------
In the case of persistent (key, value), this interface allows you to dump/load multiple formats transparently.
**single type numpy arrays** are saved into pytables arrays objects,  while **structured numpy arrays** (named / multiple dtype) or **dictionaries** are saved into pytables.Table formats. 
Conversions are meant to be transparent to the user, including multidimensional data types.

Example usage
-------------

```python
import numpy as np
#make a store
with HDFStore('tmp.hd5', mode='w') as hd:
#make some variables
d = {}
d['a'] = np.arange(10, dtype=float)
d['b'] = np.arange(10, dtype='int')
c = np.random.normal(0, 1, (10, 10))
d['c'] = c
#put values into the store
hd['/subdir/table'] = d
hd['/subdir1/array'] = c
#check values
print hd.keys()
print hd['subdir/table']
print hd['subdir1/array']
hd.removeNode('/subdir1', recursive=True)
```

API
---

####convert_dict_to_structured_ndarray(data)
> convert a dictionary type structure into a structured numpy array
>
> ######keywords
>>
>> `data`: `dictionary like object`
>>> data structure which provides iteritems and itervalues
>
> ######returns
>
>>`tab` : `structured ndarray`
>>>structured numpy array

###class HDFStore(object)
Handles quick in and out of the HDF5 file. This class can be used as a context manager as well.

Any attribute of the HDF will be directly available transparently if the source if opened.

> ####__init__(self, lnpfile, mode='r', **kwargs)
>> 
>> ######keywords
>>> `lnpfile`: `str` or `tables.file.File`
>>>> storage to use
>>>
>>> `mode`: `str` ('r', 'w', 'a', 'r+') (see `set_mode`)
>>>>  The mode to open the file. (see set_mode)
>>> 
>>> `**kwargs` can be use to set options to `tables.openFile`

> ####set_mode(self, val=None, **kwargs)
> set_mode - set a flag to open the file in a given mode operations if the mode changes, > and the storage already opened, it will close the storage and reopen it in the new mode
>
>> ######keywords
>>
>> `val`: str (optional)
>>
>>> The mode to open the file.  It can be one of the following:
>>> 
>>>> `'r'`  -- Read-only; no data can be modified.
>>>> 
>>>> `'w'`  -- Write; a new file is created (existing file would be deleted)
>>>> 
>>>> `'a'`  -- Append; an existing file is opened for reading and writing, and if the file does not exist it is created.
>>>> 
>>>> `'r+'` -- It is similar to 'a', but the file must already exist.
>
>>> `**kwargs` is forwarded to tables.openFile
>
>> ######returns
>
>>> `status`: `str`
>>>> return the status only if `None` was provided
>
> ####keep_open(self, val=None)
> set a flag to keep the storage file open for multiple operations
>
>> ######keywords
>>> `val`: `bool` (optional)
>>>> if set, set the flags to apply
>> ######returns
>>> `status`: `bool`
>>>> return the status only if None was provided
>
>
> ####open_source(self)
> Open the file if needed, handles that the object was initialized with an opened file
>
>> ######returns
>>> `v`: `bool`
>>>> returns if any operation was actually done (`True`) else the file was already opened (`False`)
>
>
> ####close_source(self, force=False)
>>close the file only if the object opened any file
>
>> ######keywords
>>>`force`: `bool` (default: `False`)
>>>>if set, bypass keep_open status
>
> ####keys(self)
>> Return a (potentially unordered) list of the keys corresponding to the objects stored in the HDFStore. These are ABSOLUTE path-names (e.g. have the leading '/'
>
> ####items(self)
>> iterator over the keys (groups)
>
> ####iteritems(self)
>> iterator on the keys (groups)
>
> ####groups(self, where='/')
>> return a list of all the top-level nodes (that are not themselves a pandas storage object)
>
> ####write(self, data, group='/', tablename=None, append=False, silent=False, header={}, **kwargs)
>>write a data structure into the source file
>> ###### keywords
>>
>>>`data`: `dict` like object
>>>>structure that contains individual columns
>>
>>>`hd5`: `tables.file.File`
>>>> opened hdf file
>>
>>> `group`: `str`
>>>>path to where the table will be created
>>
>>>`tablename`: `str`
>>>>name of the node for the table
>>
>>>`append`: `bool`
>>>>    if set, attempts to add content at the end of an existing node. Only works for Tables (not Arrays)
>>
>>>`silent`: `bool`
>>>>if set, do not print information content
>>
>>>`header`: `dictionary` like
>>>>attributes to add to the node
>>
>> ####__getitem__(self, key)
>>>Returns the node corresponding to the key
>>
>> ####__setitem__(self, key, value)
>>>create a node with the key path/name and set value as its content
>>
>> ####__getattr__(self, name)
>>>Give direct access to any function from self.source (tables.file.File)
>>
>> ####__repr__(self)
>>> Object representation
>>
>> ####__enter__(self)
>>> entering context
>>
>> ####__exit__(self, exc_type, exc_val, exc_tb)
>>> closing context
>> ####__del__(self)
>>> Destructor
>>
>>####__contains__(self, key)
>>> check for existance of this key can match the exact pathname or the pathnm w/o the leading '/'
>>
>> ####__len__(self)
>> Returns the number of nodes in the root path
>>
>> ####get_Q_from_node(self, nodename, expr, condvars={}, coordinates=None)
>>> returns a quantity from a HDF5 Node given its math expression. Assuming that all quantities are either from the node or in condvars
>>>
>>> attempt to be clever and optimizing speed when coordinates are provided
>>>
>>> all np function can be used (log, exp, pi...)
>>> ###### method            
>>>> Pure eval expression was too slow when using disk access and HD5 nodes.  Instead, we use the tables.expression feature that parse "expr" and extracts the variable names for interest.  Based on this list, we build the context of evaluation by adding missing values from the node.
>>
>>> ###### keywords
>>>
>>>>`nodename`: `str` or `tables.table.Table`
>>>>> the Table node to get data from (need named columns)
>>>
>>>>`expr`: `str`
>>>>>the mathematical expression which could include numpy functions or simply be a column name
>>>
>>>>`condvars`: `dict`
>>>>>The condvars mapping may be used to define the variable names appearing in the condition.
>>>
>>>> `coordinates`: `iterable` of `int`
>>>>>reduces the computations to only set of rows given their indexes. Only works with nodes of tables.table.Table type
>>>
>>> ######returns
>>>
>>>>`q`: `ndarray` like
>>>>>the column values requested with type and shape defined in the table.
