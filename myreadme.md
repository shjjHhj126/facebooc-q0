# facebooc-q0
- [github](https://github.com/shjjHhj126/facebooc-q0)
- acheivement : queue.c 補code
### result
![](https://i.imgur.com/psd0UAz.png)

### used struct
```c
typedef struct element {
    /* Pointer to array holding string.
     * This array needs to be explicitly allocated and freed
     */
    char *value;
    struct element *next;
} element_t;

typedef struct {
    element_t *head; /* Linked list of elements */
    element_t *tail;
    size_t size;
} queue_t;
```
### q_new()
malloc a block of memory of queue_t
```c 
/*
 * Create empty queue.
 * Return NULL if could not allocate space.
 */
queue_t *q_new()
{
    queue_t *myqueue = (queue_t*)malloc(sizeof(queue_t));
    if(!myqueue)
	return false;//coz if not enough memory, malloc() may cause error
    myqueue->head = myqueue->tail = NULL;
    myqueue->size = 0;
    return myqueue;
}
```
### q_free()
1. free the `element_t`'s value
2. free the `element_t`
```c
/* Free all storage used by queue */
void q_free(queue_t *q)
{
    if (!q)
        return;
    element_t *tmp;
    while (q->head != q->tail)
    {
        tmp = q->head;
        q->head = q->head->next;
        free(tmp->value);
        free(tmp);
    }
    free(q->head->value);
    free(q->head);
    free(q);
}
```
### q_insert_head()
1. `strcpy` vs `strncpy` vs `strdup`:
    &emsp;&emsp;`strncpy` 比 `strncpy` 安全，因為`strcpy` 會複製到遇到`\0`為止，**但是`\0`有時不會出現**。`strncpy(dest,src,size)` 需使用者自己輸入src's size，**超過就不複製**。
    &emsp;&emsp;但前兩者用在此處都不理想(以粗體標示不理想處)，原先我使用`strncpy`處理，後來看到有人用`strdup`會在`dest`無足夠空間時dynamic allocate memory。nice sol!
2. 處理dynamic memory allocation 無法運作的情況
3. 需處理`q->head == NULL`的情況
```c
/*
 * Attempt to insert element at head of queue.
 * Return true if successful.
 * Return false if q is NULL or could not allocate space.
 * Argument s points to the string to be stored.
 * The function must explicitly allocate space and copy the string into it.
 */

bool q_insert_head(queue_t *q, char *s)
{
    if(!q)
	return false;
    element_t *newElement = (element_t *)malloc(sizeof(element_t));
    if(!newElement)
	return false;//need this code?
    newElement->value = strdup(s);//strdup already malloc??
    // handle failure of allocation
    if(!newElement->value)
	    return false;
    
    // move head:head might be NULL
    newElement->next = q->head;
    if(!q->head)
	    q->tail = newElement;
    q->head = newElement;
    q->size += 1;
    return true;
}
```
### q_insert_tail()
跟`q_insert_head()`差不多
```c 
/*
 * Attempt to insert element at tail of queue.
 * Return true if successful.
 * Return false if q is NULL or could not allocate space.
 * Argument s points to the string to be stored.
 * The function must explicitly allocate space and copy the string into it.
 */
bool q_insert_tail(queue_t *q, char *s)
{
    if(!q)
	return false;
    element_t *newElement = (element_t *)malloc(sizeof(element_t));
    if(!newElement)
	return false;
    newElement->value = strdup(s);
    // handle failure of allocation
    if(!newElement->value)
	    return false;
    newElement->next=NULL;
    // move tail:tail might be NULL
    if(!q->tail)
	    q->head = newElement;
    else{
    	q->tail->next = newElement;
    }
    q->tail = newElement;
    q->size+=1;
    return true;
}
```
### q_remove_head()
1. why move to *sp? 
2. why use bufsize-1 instead of bufsize?
    (可能因為`\0`不一定要move過去，為節省空間)
```c 
/*
 * Attempt to remove element from head of queue.
 * Return true if successful.
 * Return false if queue is NULL or empty.
 * If sp is non-NULL and an element is removed, copy the removed string to *sp
 * (up to a maximum of bufsize-1 characters, plus a null terminator.)
 * The space used by the list element and the string should be freed.
 */
bool q_remove_head(queue_t *q, char *sp, size_t bufsize)
{
    if (!q || q->size == 0)
        return false;
    element_t *tmp = q->head;
    q->head = q->head->next;
    if(sp){
    	strncpy(sp,tmp->value,bufsize-1);//not sure why it's required to do this
    }
    free(tmp->value);
    free(tmp);
    q->size -= 1;
    return true;
}
```
### q_size()
就醬
```c 
/*
 * Return number of elements in queue.
 * Return 0 if q is NULL or empty
 */
size_t q_size(queue_t *q)
{
    if(!q)return 0;
    return q->size;
}
```
### q_reverse()
先用之前leetcode寫過的解法
有空學jserv用pointer to pointer處理的技巧
```c 
/*
 * Reverse elements in queue
 * No effect if q is NULL or empty
 * This function should not allocate or free any list elements
 * (e.g., by calling q_insert_head, q_insert_tail, or q_remove_head).
 * It should rearrange the existing ones.
 */
void q_reverse(queue_t *q) // jserv use ** to solve this:shortcoming?
{
    if (!q)
        return;

    q->tail = q->head; // move q->tail

    element_t *pre, *cur, *nxt;
    nxt = pre = NULL;
    cur = q->head;
    while (cur)
    {
        nxt = cur->next; // move next
        cur->next = pre; // reverse cur->next
        pre = cur;       // move pre
        cur = nxt;       // move cur
    }
    q->head = pre; // move q->head
}
```
### merge_sort()
參考 [geeksforgeeks](https://www.geeksforgeeks.org/merge-sort-for-linked-list/)
1. when to use `**` and `*`?
    `**` saves the change, `*` is just for indicating.
2. `split()`使用快慢指針
3. `merge()`因為上述1.，所以使用`*`
```c 
/*
 * The function's sorting algorithm should be merge sort.
 */
void split(element_t *head, element_t **front, element_t **back)
{
    element_t *fast, *slow;
    slow = head;
    fast = head->next;

    while (fast)
    {
        fast = fast->next;
        if (fast)
        {
            slow = slow->next;
            fast = fast->next;
        }
    }

    *front = head; 
    *back = slow->next;
    slow->next = NULL;
}
//as mentioned in split(),here use *(not sure)
element_t *merge(element_t *a, element_t *b)
{
    element_t *result = NULL;
    // base case
    if (!a)
        return b;
    else if (!b)
        return a;

    // Pick either front or back, and recur
    if (strcmp(a->value, b->value) <= 0)
    {
        result = a;
        result->next = merge(a->next, b);
    }
    else
    {
        result = b;
        result->next = merge(a, b->next);
    }
    return result;
}
void merge_sort(element_t **head)
{
    if (!(*head) || !(*head)->next)
        return;

    element_t *a = NULL;
    element_t *b = NULL;
    split(*head, &a, &b); // &(**head)? 
    merge_sort(&a);
    merge_sort(&b);

    *head = merge(a, b);
}
```
### q_sort()
就醬
```c 
/*
 * Sort elements of queue in ascending order
 * No effect if q is NULL or empty. In addition, if q has only one
 * element, do nothing.
 */
void q_sort(queue_t *q)
{
    if (!q || !q->head || q->size == 1)
        return;

    merge_sort(&q->head); // so q->head point at the smallest value
    element_t *walk = q->head;
    while(walk->next){
    	walk=walk->next;
    }
    q->tail=walk;
}
```
