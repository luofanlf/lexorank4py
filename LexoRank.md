# LexoRank

### what is lexorank

Lexorank, the full name is lexographic ranking. It is a ranking algorithm to generate unique rank for each element which can maintain the overall order while new elements are inserted. It is primarily used by products such as Jira and Trello, very suitable for the drag and drop sorting scenario.



### why lexorank

* **Context**

For example, in a LCIA (lifecycle impact assessment) scenario,there is a three-levels architecture consisting of datapoint-stage-process. The relationship from left to right is one to many. In the past, we use consecutive sequence number to mark the stage and process(1,2,3...). It is straightforward but when a new element need to be inserted in the middle, all the elements after the insert position need to be updated. This solution become  inefficient  while having numerous elements.

* **Linkedlist**

Another good way to solve the shortfall above is to use linked list. Then it can achieve O(1) time complexity for updating. However, you will need to query recursively to get a certain element. But in the LCIA system, wo try to build linkedlist and point to the next element's id(UUID). But one of our features is to copy and insert. For example, users can copy one stage then the stage along with the processes under it will be copied. In this scenario, things will become tricky because we need to reconstruct the whole linkedlist again since all the id for new stage and processes are changed.

* **Decimal**

A similar solution is to used decimal number to store and rank and use the average value to insert the middle element's rank. We can use python's decimal package to avoid the precision loss issue, and extend the precision limit manually but it is still not ifinite. What's more, the precision growth rate of Decimal increases much faster than that of LexoRank. eg: we have 1.0, 2.0. In worst situation, it will be like 1.5,1.25,1.125... every time i try to get the average the precision will expand one digit.



So I do some researchs and try to implement lexorank for my LCIA system, which is most suitable for my drag and drop to copy scenario.



### how does lexorank work

* **Base36**

  In lexorank, we use base36 string to store the rank. Base36 is a radix 36 numeral system, using 36 character to present number. "a" presents 10, "z" presents 35.

* **Lexographic sort (aka alphbetical sort)**

  It is a method of ordering strings (or sequences) in the same way words are arranged in a dictionary. It compares elements from left to right, one character at a time, using their numerical or alphabetical values. By default, if we store the rank in text in database. It will be sorted by lexographic order if we order by that column.

* **Generating new rank**
  
  ​	Generating a new rank between elements is the most common operation in lexorank. The logic is in this way: First, get the 	rank of left element("a") and right element("z"). Second, convert rank into int: a to 10, z to 35. Third, sum the value and floor   	divided by 2 :10+35）/2 ->get  22.Last, convert into rank and insert: insert “m” in the middle. But there will be situations when 	no gap exist between the target elements like "a" and "b"

* **Shortfall of lexorank**

  There is no silver bullet. Lexorank need to expand the rank when there is no gap between two elements, so it also cannot avoid the issue of rank string growing longer over time. This causes the database storage cost and decline in database's performance while having a lot of elements in the system.

* **Fixed length rank or variable rank**

  fixed-length or variable length both has its pros and cons so it depends on your demand.

  fixed-length **pros**:

  * better for database indexing
  * stable storage

  fixed-length **cons**:

  * Limited insertion count, must rebalance when the gap is run out

  Variable length **pros**:

  * theoretically infinite insertion count
  * storage-saving for the start

  Variable length **cons**:

  * cause degradation of db performance while rank's length keep expanding (better adopt rebalance)

* **Worst case**:

  Imagine this scenario, insert a new element between the head and tail and keep inserting to the middle of head and the new element, then the gap will shrink at a halving rate. 

  If we adopt the fixed length of 10 digits and assume only 2 elements exists("000..." and "zzz...."),  then the initial gap is 36^10. In worst case, the maximum insertion count will be log_2(36^10)，approximately it is 51.That's mean we need to rebalance after 51 insertion.

  If we adopt the variable length, let's initialize the elements with '0' and 'z',then after log_2(36), which is five insertion, we need to expand the rank's length

* **initialization**

  When initializing the rank, we try to evenly distribute the gap and generate the ranks as short as possible. Since we need remain the gap for the situation inserting at the head and tail, so when elements less than 35 we can initialize the rank with 1 digit. Otherwise, we use two digit for initialization, then it can support 36*36+36 - near 1k elements.

