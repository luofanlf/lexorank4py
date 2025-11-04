# LexoRank

### what is lexorank

Lexorank, the full name is lexographic ranking. It is a ranking algorithm to generate unique rank for each element which can maintain the overall order while new elements are inserted. It is primarily used by products such as Jira and Trello, very suitable for the drag and drop sorting scenario.



### why lexorank

For example, in a LCIA (lifecycle impact assessment) scenario,there is a three-levels architecture consisting of datapoint-stage-process. The relationship from left to right is one to many. In the past, we use consecutive sequence number to mark the stage and process(1,2,3...). It is straightforward but when a new element need to be inserted in the middle, all the elements after the insert position need to be updated. This solution become  inefficient  while having numerous elements.

Another good way to solve the shortfall above is to use linked list. Then it can achieve O(1) time complexity for updating. However, you will need to query recursively to get a certain element. But in the LCIA system, wo try to build linkedlist and point to the next element's id(UUID). But one of our features is to copy and insert. For example, users can copy one stage then the stage along with the processes under it will be copied. In this scenario, things will become tricky because we need to reconstruct the whole linkedlist again since all the id for new stage and processes are changed.

So I do some research and try to implement lexorank for my LCIA system, which is most suitable for my drag and drop to copy scenario.



### how does lexorank work

* **Base36**

  In lexorank, we use base36 string to store the rank. Base36 is a radix 36 numeral system, using 36 character to present number. "a" presents 10, "z" presents 35.

* **Lexographic sort (aka alphbetical sort)**

  It is a method of ordering strings (or sequences) in the same way words are arranged in a dictionary. It compares elements from left to right, one character at a time, using their numerical or alphabetical values. By default, if we store the rank in text in database. It will be sorted by lexographic order if we order by that column.

* **generating new rank**

​	Generating a new rank between elements is the most common operation in lexorank. The logic is in this way: First, get the 	rank of left element("a") and right element("z"). Second, convert rank into int: a to 10, z to 35. Third, sum the value and floor   	divided by 2 :10+35）/2 ->get  22.Last, convert into rank and insert: insert “m” in the middle. But there will be situations when 	no gap exist between the target elements like "a" and "b"

* Shortfall of lego

* initialization