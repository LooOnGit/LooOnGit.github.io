---
title: "Kernel Facilities and Helper Functions"
date: 2026-03-08 14:27:32 +0800
categories: [Kernel]
tags: [Kernel]
---

# KERNEL FACILITIES AND HELPER FUNCTIONS
## Understanding the container_of macro
`container_of` là macro dùng để lấy struct cha từ con trỏ của một field trong struct. 


Được define trong `include/linux/kernel.h`.
**Example:**
```c
struct person {
    char *name;
    int age;
    struct list_head list;
};

struct person *p;
struct list_head *l = &p->list;
struct person *p2 = container_of(l, struct person, list);
```
Nếu như dùng `container_of` thì chỉ cần trỏ đến 1 trong các field trong struct, giả sử ở ví dụ này trỏ đến **list**, thì có thể tìm ra struct đầy đủ chưa field đó.
- **Syntax:**
```c
container_of(ptr, type, member)
```
- **ptr:** Con trỏ đến field trong struct.
- **type:** Kiểu dữ liệu của struct.
- **member:** Tên của field trong struct.


Cách nó hoạt động như nào, thì `container_of` sẽ tính đến offset của field đang được trỏ đến so với đầu struct để lấy được địa chỉ con trỏ chính xác của struct.
```bash
(type *)((char *)__mptr - offsetof(type, member));
```

**Example 2:**
```c
struct family {
    struct person *father;
    struct person *mother;
    int number_of_sons;
    int family_id;
} f;

/* khởi tạo f ở đâu đó */

/* con trỏ tới một field trong struct */
int *fam_id_ptr = &f.family_id;

struct family *fam_ptr;

/* lấy lại struct family */
fam_ptr = container_of(fam_id_ptr, struct family, family_id);
```

- **NOTE**: `container_of` không work cho member `char *` hoặc array. Có nghĩa là tham số đầu tiền của `container_of` không được là pointer đến `char` hoặc array trong struct.
Ví dụ trên đi thì khi truyền con trỏ đến `father` hay `mother` là tham số đầu tiên của `container_of` thì sẽ không work, vì các member này đã là pointer field trong struct rồi.

Để sử dụng đúng, bạn cần một con trỏ tới con trỏ của `struct person`, tức là:
```bash
struct person **dad;
struct person **mom;
```
Sau đó sử dụng: 
```bash
struct family *fam =
container_of(dad, struct family, father);
```

**Example in driver:**
```c
struct mcp23016 {
    struct i2c_client *client;
    struct gpio_chip chip;
};

/* retrieve the mcp23016 struct given a pointer 'chip' field */
static inline struct mcp23016 *to_mcp23016(struct gpio_chip *gc)
{
    return container_of(gc, struct mcp23016, chip);
}

static int mcp23016_probe(struct i2c_client *client,
                          const struct i2c_device_id *id)
{
    struct mcp23016 *mcp;

    mcp = devm_kzalloc(&client->dev, sizeof(*mcp), GFP_KERNEL);

    if (!mcp)
        return -ENOMEM;

    [...]
}
```

## Linked lists
Nếu driver quản lý nhiều thiết bị, cần một cách để lưu và theo dõi chúng -> dùng linked list. Có 2 loại linked list:
- **Simple linked list**: Chỉ có 1 con trỏ `next`.
- **Doubly linked list**: Có 2 con trỏ `next` và `prev`.
Trong Linux kernel, các lập trình viên chỉ dùng circular doubly linked list (danh sách liên kết kép dạng vòng). Vì cấu trúc này cho phép triển khai:
- Triển khai FIFO (First In First Out – hàng đợi).
- Triển khai LIFO (Last In First Out – stack).


Header cần include để dùng linked list là:
```bash
#include <linux/list.h>
```


Data struct ở core của việc triển khai list trong kernel là `struct list_head`, được define như sau:
```c
struct list_head {
    struct list_head *next, *prev;
};
```
Trong kernel, trước một data structure có thể biểu diễn dưới dạng linked list, cấu trúc đó phải chứa embed một `struct list_head` field.


**Example:** Khi muốn tạo một list cars:
```bash
struct car {
    int door_number;
    char *color;
    char *model;
};
```
Trước khi create một list cars, chúng ta phải thay đổi structure để embed thêm một field là `struct list_head`: 
```c
struct car {
    int door_number;
    char *color;
    char *model;
    struct list_head list; /* cấu trúc list của kernel */
};
```
Trước tiên, chúng ta cần tạo một biến kiểu struct list_head luôn trỏ đến đầu danh sách (phần tử đầu tiên) của list. Instance list_head này không gắn với bất kỳ chiếc xe nào, và nó là một node đặc biệt:
```c
static LIST_HEAD(carlist);
```

Bây giờ chúng ta có thể tạo các chiếc xe và thêm chúng vào danh sách `carlist`:
```c
#include <linux/list.h>

struct car *redcar = kmalloc(sizeof(*car), GFP_KERNEL);
struct car *bluecar = kmalloc(sizeof(*car), GFP_KERNEL);

/* Initialize each node's list entry */
INIT_LIST_HEAD(&bluecar->list);
INIT_LIST_HEAD(&redcar->list);

/* allocate memory for color and model field and fill every field */
[...]

list_add(&redcar->list, &carlist);
list_add(&bluecar->list, &carlist);
````

### Creating and initializing a list
#### Dynamic method
Dynamic method bao gồm một cấu trúc `list_head` và initializes nó bằng macro `INIT_LIST_HEAD`.
```c
struct list_head mylist;
INIT_LIST_HEAD(&mylist);
```
Flow theo expansion của `INIT_LIST_HEAD`:
```c
static inline void INIT_LIST_HEAD(struct list_head *list)
{
    list->next = list;
    list->prev = list;
}
```
#### Static method
Static allocation được thực hiện thông qua macro `LIST_HEAD`.
```c
static LIST_HEAD(mylist);
```
`LIST_HEAD` definition thì theo sau:
```c
#define LIST_HEAD(name) \
    struct list_head name = LIST_HEAD_INIT(name)
```
Sau đây là phần expansion:
```c
#define LIST_HEAD_INIT(name) { .next = &(name), .prev = &(name) }
```
Điều này gán cho mỗi pointer (prev and next) bên trong field `name` trỏ tới chính `name` đó (giống hệt như các `INIT_LIST_HEAD` thực hiện).

### Creating a list node
Để tạo ra những nodes mới, chỉ cần tạo ra instance cả struct và khởi tạo trường `list_head` được embed bên trong nó. Sử dụng car ví dụ như sau:
```c
struct car *blackcar = kzalloc(sizeof(struct car), GFP_KERNEL);
/* non static initialization, since it is the embedded list field*/
INIT_LIST_HEAD(&blackcar->list);
```
Như chúng tôi đã đề cập trước đó, sử dụng `INIT_LIST_HEAD` nó thì cấp phát động list và thường là cấu trúc khác.
### Adding a list node
Kernel cung cấp `list_node` để thêm nội dung (entry) mới vào danh sách, đây là hàm bọc (wrapper) bao quanh hàm nội bộ `__list_add`.
```c
void list_add(struct list_head *new, struct list_head *head);
static inline void list_add(struct list_head *new, struct list_head *head)
{
    __list_add(new, head, head->next);
}
```
`list_add` sẽ thêm 2 mục nội dung (entries) như parameter, và insert những phần tử của bạn vào giữa chúng. Nó thì implement trong kernel khá đơn giản.
```c
static inline void __list_add(struct list_head *new,
                              struct list_head *prev,
                              struct list_head *next)
{
    next->prev = new;
    new->next = next;
    new->prev = prev;
    prev->next = new;
}
```
Dưới đây là một ví dụ thêm 2 cars vào list:
```c
list_add(&redcar->list, &carlist);
list_add(&blue->list, &carlist);
```
Mode này có thể sử dụng để implement một **stack**. Hàm khác để thêm một entry vào list:
```c
void list_add_tail(struct list_head *new, struct list_head *head);
```
Insert này thì thêm entry vào cuối list. Sử dụng như sau: 
```c
list_add_tail(&redcar->list, &carlist);
list_add_tail(&blue->list, &carlist);
```
Mode này có thể sử dụng để implement một **queue**.
### Deleting a node from the list
Để Handle List dễ dàng trong kernel. Delete một node là một quá trình khá dễ. Hàm được sử dụng là `list_del`.
```c
void list_del(struct list_head *entry);
```
Tiếp tục với ví dụ trước, để delete red car:
```c
list_del(&redcar->list);
```
**NOTE**: `list_del` disconnect `prev` và `next` pointer của mục nội dung (entry) được cung cấp, dẫn đến việc loại bỏ entry đó. Vung nhớ cấp phát cho node thì vẫn chưa được `free`,  bạn cần phải thực hiện điều đó một cách thủ công (bằng tay) bằng hàm `kfree`.
### Linked list traversal
Dùng `list_for_each_entry(pos, head, member)` để duyệt qua list:
- `head`: là những head của node.
- `member`: là tên của `struct list_node` nằm bên trong data struct.
- `pos` là sử dùng cho việc lặp lại (iteration). Nó hoạt động như một con trỏ (giống i trong `for(i=0; i<foo;i++)`). `head` có thể là head node của linked list, hoặc bất kỳ phần tử nào khác.
```c
struct car *acar; /*loop counter*/
int blue_car_num = 0;

/* 'list' iss the name of the list_head struct in our data structure */
list_for_each_entry(acar, carlist, list){
    if(acar->color == "blue")
        blue_car_num++;
}
```
`list_for_each_entry` được definition:
```c
#define list_for_each_entry(pos, head, member)
for (pos = list_entry((head)->next, typeof(*pos), member); 
    &pos->member != (head); 
    pos = list_entry(pos->member.next, typeof(*pos), member))

#define list_entry(ptr, type, member) container_of(ptr, type, member)
```

## The kernel sleeping mechanism
Sleep là mechanism mà qua đó process relaxes một processor (nhường lại bộ vi xử lý cho tiến trình khác). Lý do tại sao processor sleep có thể sensing data availability, hoặc waiting cho một resource được free.


Kernel scheduler manager một list những task run, biết như là run queue. Sleep processes sẽ không được scheduleed, bởi vì nó xóa khỏi run queue, trừ khi nó được wakup.
### Wait queue
Wait queues được sử dụng để xử lý block I/O, để chờ một số điều kiện cụ thể diễn ra để trở thành true, và nhận biết khi nào có dữ liệu hoặc resource availability. Để hiểu làm việc như nào xem trong `include/linux/wait.h`:
```c
struct __wait_queue {
    unsigned int flags;
#define WQ_FLAG_EXCLUSIVE 0x01
    void *private;
    wait_queue_func_t func;
    struct list_head task_list;
}; 
```
Mỗi process muốn đưa vào sleep thì đều phải queued trong list và được đưa vào sleep state cho đến khi có điều kiện true. Một số hàm sẽ luôn gặp phải khi làm việc với wait queue:
- **Static declaration**:
```c
DECLARE_WAIT_QUEUE_HEAD(name)
```
- **Dyanmic delaration**:
```c
wait_queue_head_t my_wait_queue;
init_waitqueue_head(&my_wait_queue);
```
- **Blocking**:
```c
/*
* block the current task (process) in the wait queue if
* CONDITION is false
*/
int wait_event_interruptible(wait_queue_head_t q, CONDITION);
```
- **Unblocking**:
```c
/*
* wake up one process sleeping in the wait queue if
* CONDITION above has become true
*/
void wake_up_interruptible(wait_queue_head_t *q);
```
Hàm `wait_event_interruptible` không liên tục thăm dò (thực hiện vòng lặp kiểm tra liên tục/polling), mà chỉ đơn thuần đánh giá (kiểm tra) điều kiện khi được gọi. Nếu điều kiện trả về là false, tiến trình sẽ được được vào trạng thái `TASK_INTERRUPTIBLE` và rời khỏi run queue.
- Điều kiện chỉ kiểm tra lại mỗi lần gọi `wake_up_interruptible` trong wait queue.
- Điều kiện true khi `wake_up_interruptible` được chạy, một process in wait queue sẽ được đánh thức và set trạng thái `TASK_RUNNING`.
- Muốn gọi tất cả các processes chờ trong queue, nên sử dựng `wake_up_interruptible_all`.
## Delay and timer management
### Standard timers
### High-resolution timers (HRTs)
### Dynamic tick/tickless kernel
### Delays and sleep in the kernel
## Kernel locking mechanism
### Mutex
### Spinlock
## Work deferring mechanism
### Softirqs and ksoftirqd
### Tasklets
### Tasklet scheduling
### Work queues
### Kernel threads
## Threaded IRQs
### Threaded bottom half
## Invoking user space applications from the kernel

