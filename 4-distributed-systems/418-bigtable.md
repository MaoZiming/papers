# BigTable

* A Bigtable is a sparse, distributed, persistent multi- dimensional sorted map. The map is indexed by a row key, column key, and a timestamp; each value in the map is an uninterpreted array of bytes.
* `(row:string, column:string, time:int64) → string`
* Each row range is called a tablet, which is the unit of distribution and load balancing. 
* Column keys are grouped into sets called column families, which form the basic unit of access control. All data stored in a column family is usually of the same type. 
* Timestamps. Applications that need to avoid collisions must generate unique timestamps themselves. 

## SSTable

* The Google SSTable file format is used internally to store Bigtable data. An SSTable provides a persistent, ordered immutable map from keys to values, where both keys and values are arbitrary byte strings.

## Chubby

* Distributed lock service. 