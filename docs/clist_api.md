# STC [clist](../stc/clist.h): Forward List
![List](pics/list.jpg)

The **clist** container supports fast insertion and removal of elements from anywhere in the container.
Fast random access is not supported.

Unlike the c++ class *std::forward_list*, **clist** has an API similar to ***std::list***, and also supports
*push_back()* (**O**(1) time). It is still implemented as a singly-linked list. A **clist** object
occupies only one pointer in memory, and like *std::forward_list* the length of the list is not stored.
All functions have **O**(1) complexity, apart from *clist_X_count()*, which returns size of list in **O**(*n*) time.

***Iterator invalidation***: Adding, removing and moving the elements within the list, or across several lists
will invalidate other iterators currently refering to these elements and their immediate succesive elements.
However, an iterator to a succesive element can both be dereferenced and advanced. After advancing, the
iterator is in a valid state. This implies:

- `clist_X_insert(&L, clist_X_fwd(it,1), x)` is identical to *std::forward_list* `L.insert_after(it, x)`.
- `clist_X_erase_at(&L, clist_X_fwd(it,1))` is identical to *std::forward_list* `L.erase_after(it)`.
- Iterators returned from *clist_X_insert()* and *clist_X_erase_at()* are always valid.
- Elements can be safely removed from a list via multiple iterators if done back to front order.

See the c++ class [std::list](https://en.cppreference.com/w/cpp/container/list) for similar API and
[std::forward_list](https://en.cppreference.com/w/cpp/container/forward_list) for a functional description.

## Header file and declaration

```c
#include <stc/clist.h>

using_clist(X, Value);
using_clist(X, Value, valueCompare);
using_clist(X, Value, valueCompare, valueDel, valueClone = c_no_clone);
using_clist(X, Value, valueCompareRaw, valueDel, valueFromRaw, valueToRaw, RawValue);

using_clist_str()
```
The macro `using_clist()` must be instantiated in the global scope. `X` is a type tag name and
will affect the names of all clist types and methods. E.g. declaring `using_clist(i, int);`, `X` should
be replaced by `i` in all of the following documentation. `using_clist_str()` is a shorthand for
```c
using_clist(str, cstr, cstr_compare_raw, cstr_del, cstr_from, cstr_c_str, const char*)
```

## Methods

```c
clist_X             clist_X_init(void);
clist_X             clist_X_clone(clist_X list);

void                clist_X_clear(clist_X* self);
void                clist_X_del(clist_X* self);                                           // destructor

bool                clist_X_empty(clist_X list);
size_t              clist_X_count(clist_X list);                                          // size() in O(n) time

clist_X_value_t*    clist_X_front(const clist_X* self);
clist_X_value_t*    clist_X_back(const clist_X* self);

void                clist_X_push_front(clist_X* self, Value value);
void                clist_X_emplace_front(clist_X* self, RawValue raw);
void                clist_X_pop_front(clist_X* self);

void                clist_X_push_back(clist_X* self, Value value);                        // note: no pop_back().
void                clist_X_emplace_back(clist_X* self, RawValue raw);
void                clist_X_emplace_n(clist_X *self, const clist_X_rawvalue_t arr[], size_t n);

clist_X_iter_t      clist_X_insert(clist_X* self, clist_X_iter_t it, Value value);        // return iter to new elem
clist_X_iter_t      clist_X_emplace(clist_X* self, clist_X_iter_t it, RawValue raw);

clist_X_iter_t      clist_X_erase_at(clist_X* self, clist_X_iter_t it);                   // return iter after it
clist_X_iter_t      clist_X_erase_range(clist_X* self, clist_X_iter_t it1, clist_X_iter_t it2);
size_t              clist_X_remove(clist_X* self, RawValue raw);                          // removes all elements equal to raw

clist_X             clist_X_split(clist_X* self, clist_X_iter_t it1, clist_X_iter_t it2); // split out [it1, it2)
clist_X_iter_t      clist_X_splice(clist_X* self, clist_X_iter_t it, clist_X* other);     // return updated valid it
clist_X_iter_t      clist_X_splice_range(clist_X* self, clist_X_iter_t it,                // return updated valid it
                                         clist_X* other, clist_X_iter_t it1, clist_X_iter_t it2);

clist_X_iter_t      clist_X_find(const clist_X* self, RawValue raw);
clist_X_iter_t      clist_X_find_in(const clist_X* self,
                                    clist_X_iter_t it1, clist_X_iter_t it2, RawValue raw);

void                clist_X_sort(clist_X* self);

clist_X_iter_t      clist_X_begin(const clist_X* self);
clist_X_iter_t      clist_X_end(const clist_X* self);
void                clist_X_next(clist_X_iter_t* it);

clist_X_iter_t      clist_X_fwd(clist_X_iter it, size_t n);                        // return it n elements ahead. End allowed.
clist_X_value_t     clist_X_value_clone(clist_X_value_t val);
```

## Types

| Type name             | Type definition                     | Used to represent...      |
|:----------------------|:------------------------------------|:--------------------------|
| `clist_X`             | `struct { clist_X_node_t* last; }`  | The clist type            |
| `clist_X_value_t`     | `Value`                             | The clist element type    |
| `clist_X_rawvalue_t`  | `RawValue`                          | clist raw value type      |
| `clist_X_iter_t`      | `struct { clist_value_t *ref; ... }`| clist iterator            |

## Example

Interleave *push_front()* / *push_back()* then *sort()*:
```c
#include <stc/clist.h>
#include <stdio.h>

using_clist(d, double);

int main() {
    c_init (clist_d, list, {
        10.0, 20.0, 30.0, 40.0, 50.0, 60.0, 70.0, 80.0, 90.0
    });

    c_forrange (i, int, 1, 10) {
        if (i & 1) clist_d_push_front(&list, (float) i);
        else       clist_d_push_back(&list, (float) i);
    }

    printf("initial: ");
    c_foreach (i, clist_d, list)
        printf(" %g", *i.ref);

    clist_d_sort(&list); // mergesort O(n*log n)

    printf("\nsorted: ");
    c_foreach (i, clist_d, list)
        printf(" %g", *i.ref);

    clist_d_del(&list);
}
```
Output:
```
initial:  9 7 5 3 1 10 20 30 40 50 60 70 80 90 2 4 6 8
sorted:  1 2 3 4 5 6 7 8 9 10 20 30 40 50 60 70 80 90
```
### Example 2

Use of *erase_at()* and *erase_range()*:
```c
// erasing from clist
#include <stc/clist.h>
#include <stdio.h>

using_clist(i, int);

int main ()
{
    c_init (clist_i, L, {10, 20, 30, 40, 50});
                                                 // 10 20 30 40 50
    clist_i_iter_t it = clist_i_begin(&L);       // ^
    clist_i_next(&it); 
    it = clist_i_erase_at(&L, it);               // 10 30 40 50
                                                 //    ^
    clist_i_iter_t end = clist_i_end(&L);        //
    clist_i_next(&it);
    it = clist_i_erase_range(&L, it, end);       // 10 30
                                                 //       ^
    printf("mylist contains:");
    c_foreach (x, clist_i, L) printf(" %d", *x.ref);
    puts("");

    clist_i_del(&L);
}
```
Output:
```
mylist contains: 10 30
```

### Example 3

Splice `[30, 40]` from *L2* into *L1* before `3`:
```c
#include <stc/clist.h>
#include <stdio.h>

using_clist(i, int);

int main() {
    c_init (clist_i, L1, {1, 2, 3, 4, 5});
    c_init (clist_i, L2, {10, 20, 30, 40, 50});

    clist_i_iter_t i = clist_i_fwd(clist_i_begin(&L1), 2);
    clist_i_iter_t j1 = clist_i_fwd(clist_i_begin(&L2), 2), j2 = clist_i_fwd(j1, 2);

    clist_i_splice_range(&L1, i, &L2, j1, j2);

    c_foreach (i, clist_i, L1) printf(" %d", *i.ref); puts("");
    c_foreach (i, clist_i, L2) printf(" %d", *i.ref); puts("");
}
```
Output:
```
1 2 30 40 3 4 5
10 20 50
```
