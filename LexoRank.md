# LexoRank

### what is lexorank

Lexorank, the full name is lexographic ranking. It is a ranking algorithm to generate unique rank for each element which can maintain the overall order while new elements are inserted. It is primarily used by products such as Jira and Trello, very suitable for the drag and drop sorting scenario.



### why lexorank

For example, in a LCIA (lifecycle impact assessment) scenario,there is a three-levels architecture consisting of datapoint-stage-process. The relationship from left to right is one to many. In the past, we use consecutive sequence number to mark the stage and process(1,2,3...). It is straightforward but when a new element need to be inserted in the middle, all the elements after the insert position need to be updated. This solution become  inefficient  while having numerous elements.

Another good way to solve the shortfall above is to use linked list. Then it can achieve O(1) time complexity for updating. However, you will need to query recursively to get a certain element. But in the LCIA system, wo try to build linkedlist and point to the next element's id(UUID). But one of our features is to copy and insert. For example, users can copy one stage then the stage along with the processes under it will be copied. In this scenario, things will become tricky because we need to reconstruct the whole linkedlist again since all the id for new stage and processes are changed.

So I do some research and try to implement lexorank for my LCIA system, which is most suitable for my drag and drop to copy scenario.



### how does lexorank work

* Base36

  In lexorank, we use base36 string to store the rank. Base36