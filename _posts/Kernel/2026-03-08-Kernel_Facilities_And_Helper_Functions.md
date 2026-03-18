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


**NOTE**: Thực tế main function `wait_event`, `wake_up`, `wake_up_all`. Chúng đước sử dụng với process trong queue ở trạng thái exclusive (uninterruptible) wait, bởi vì chúng không thế interrupt bằng signal. Chỉ nên sử dụng cho các task quan trọng. Các hàm có thể bị ngắt (interruptible functions) chỉ là tùy chọn (nhưng được khuyến khích sử dụng). Vì chúng có thể bị gián đoạn bởi các tín hiệu, bạn nên kiểm tra giá trị trả về của chúng. Một giá trị khác không (nonzero) có nghĩa là tiến trình đang ngủ của bạn đã bị ngắt bởi một loại tín hiệu nào đó, và khi đó driver nên trả về mã lỗi `ERESTARTSYS`.

**Example**:
```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/sched.h>
#include <linux/time.h>
#include <linux/delay.h>
#include <linux/workqueue.h>

static DECLARE_WAIT_QUEUE_HEAD(my_wq);
static int condition = 0;

/*declare a work queue*/
static struct work_struct wrk;

static void work_handler(struct work_struct *work)
{
    printk("Waitqueue module handler %s\n", __FUNCTION__);
    msleep(5000);
    printk("Wake up the sleeping module\n");
    condition = 1;
    wake_up_interruptible(&my_wq);
}

static int __init my_init(void)
{
    printk("Wait queue example\n");

    INIT_WORK(&wrk, work_handler);
    wait_event_interruptible(my_wq, condition != 0);

    pr_info("woken up by the work job\n");
    return 0;
}

void my_exit(void)
{
    printk("Wait queue example cleanup\n");
}

module_init(my_init);
module_exit(my_exit);
MODULE_AUTHOR("John Madieu <john.madieu@foobar.com>");
MODULE_LICENSE("GPL");
```
Khi `insmod` sẽ được vào sleep trong wait queue khoảng 5 giây, và sau đó được wake up bằng work handler. Dsmeg để xem kết quả:


![User space and kernel space](/assets/Kernel/Kernel_Facilities_and_Helper_Functions/example1.png)
## Delay and timer management
Time được sử dụng nhiều nhất, chỉ đứng sau bộ nhớ. Nó được dùng để làm hầu hết mọi thức, sleep, scheduling, timout và nhiều tác vụ khác.


Có 2 loại timer. Kernel sử dụng thời gian tuyệt đối (absolute time) để biết mấy giờ, nghĩa là ngày và giờ hiện tại. Trong khi đó, thời gian tương đối (relative time), là một hardware chip gọi (RTC). Mặt khác, để xử lý thời gian tương đối, kernel dựa vào một tính năng của CPU (ngoại vi) được gọi là bội định thời (timer), mà theo quan điểm của kernel, được gọi là kernel timer. Kernel timer chính là nội dung mà chúng ta sẽ thảo luận trong phần này.

Kernel timers có 2 loại khác nhau:
- Standard timers hoặc system timers.
- High-resolution timers.
### Standard timers
Standard timer là kernel timer operating dựa trên bộ phân giải (granularity) của jiffies (nhịp của hệ thống).
#### Jiffies and Hz
Một jiffy là kernel unit được declared trong `include/linux/jiffies.h`. Hiểu về **jiffies** cần biết về **Hz** nó là số lần **jiffies** được tăng lên trong 1 giây. Mỗi lần tăng như vậy gọi là **tick**. Hz được đại diện cho size của **jiffy**. **Hz** phụ thuộc vào hardware và kernel version, cũng xác định tần suất clock interrupt xuất hiện. Chỉ số này được config trên một số archtectures, đã được cố định trên các kiến trúc. 


**Jiffies** sẽ được tăng lên **Hz** lần trong 1 giây. Nếu **Hz** = 1000, thì nó được tăng lên 1000 lần (tức là cứ mỗi 1/1000 giây lại có một tick). Sau khi được xác định, **programmable interrupt timer (PIT)**, nó là một linh kiện hardware, đã được lập trình với giá trị đã được lập trình để tăng **jiffies** mỗi khi ngắt PIT xảy ra.


Tùy vào platform, jiffies có thể dẫn đến overflow. Trên hệ thống 32-bit, Hz = 1000 sẽ duy trì được 50 ngày, trong khi con số này lên đến 600 million years trên hệ thống 64-bit. Được lưu trữ jiffies bằng biến 64-bit, vấn đề được giải quyết. Một biến thứ 2 được giới thiệu được define trong `<linux/jiffies.h>`:
```c
extern u64 jiffies_64;
``` 
Bằng cách này, hệ thống 32-bit system jiffies sẽ point vào 32-bit thấp, và `jiffies_64` sẽ point bao gồm cả bit cao. Trên 64-bit platform, `jiffies` và `jiffies_64`.
#### The timer API
Một **timer** được đại điện trong kernel dưới dạng `struct timer_list` được define trong `<linux/timer.h>`:
```c
struct timer_list {
    struct list_head entry;
    unsigned long expires;
    struct tvec_t_base_s *base;
    void (*function)(unsigned long);
    unsigned long data;
};
```
`expires` là giá trị tuyệt đối tình bằng jiffies. `entry` là một danh sách liên kết đôi, và `data` là thành phần không bắt buộc và được truyền vào hàm callback.
##### Timer setup initialization
**1. Setting up the timer**: Set up the timer, cung cấp user-define callback and data:
```c
void setup_timer(struct timer_list *timer, void (*function)(unsigned long), unsigned long data);
```
Có thể sử dụng như sau:
```c
void init_timer(struct timer_list *timer);
```
`setup_timer` là một wrapper của `init_timer`. 
**2. Setting the expiration time**: Khi timer được khai báo, cần set expiration trước khi callback dược gọi:
```c
int mod_timer(struct timer_list *timer, unsigned long expires);
```
**3. Releasing the timer**: Khi không còn dùng timer, cần được giải phóng:
```c
void del_timer(struct timer_list *timer);
int del_timer_sync(struct timer_list *timer);
```
`del_timer` trả về **void**, bất kể nó có deactived một pending timer (bộ định thời) hay không. Nó return value là 0 nếu timer không hoặc động, hoặc 1 đang active. Cuối cùng, `del_timer_sync`, chờ cho đến khi xử lý xong, ngay cả khi xảy ra trên CPU khác. Không nên giữ lock chặn quá trình xử lý, nếu không sẽ đẫn đến tình trạng khóa chết (deadlock). Nên giải phóng timer trong **module cleanup routine** (hàm dọn dẹp). Có thể kiểm tra độc lập liệu có chạy hay không:
```c
int timer_pending(const struct timer_list *timer);
```
Hàm check liệu có bất kì hàm callback của timer đã được kích hoạt.
##### Standard timer example
```c
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/timer.h>

static struct timer_list my_timer;

void my_timer_callback(unsigned long data)
{
    printk("%s called (%ld).\n", __FUNCTION__, jiffies);
}

static int __init my_init(void)
{
    int retval;

    printk("Timer module loaded\n");

    /* Initialize the timer structure */
    setup_timer(&my_timer, my_timer_callback, 0);

    printk("Setup timer to fire in 300ms (%ld)\n", jiffies);

    /* Activate/Modify the timer to expire after 300ms */
    retval = mod_timer(&my_timer, jiffies + msecs_to_jiffies(300));

    if (retval)
        printk("Timer firing failed\n");

    return 0;
}

static void my_exit(void)
{
    int retval;

    /* Deactivate the timer */
    retval = del_timer(&my_timer);

    /* Check if the timer was still pending when deleted */
    if (retval)
        printk("The timer is still in use...\n");

    pr_info("Timer module unloaded\n");
}

module_init(my_init);
module_exit(my_exit);

MODULE_AUTHOR("John Madieu <john.madieu@gmail.com>");
MODULE_DESCRIPTION("Standard timer example");
MODULE_LICENSE("GPL");
```
### High-resolution timers (HRTs)
Standart timer có độ chính xác thấp và không phụ hợp ứng dụng. HRT được giới thiệu trong kernel v2.6.16( và enable bằng cách CONFIG_HIGH_RES_TIMERS option trong kernel configuration) có độ phân giải ở mức microsecond (up to nanosecond, phụ thuộc vào platform), so sánh với millisecond của standard timer.


Standard timer phụ thuộc vào HZ (vì chúng dựa trên jiffies), trong khi HRT được implement dựa trên `ktime`. Kernel và hardware phải support HRT trước khi sử dụng. Nói các khác phải có phần mã phụ thuốc kiến trúc (architecture-dependent) được triển khai để cho phép hardware HRTs.
#### HRT API
Yêu cầu header:
```c
#include <linux/hrtimer.h>
```
HRT được đại diện trong kernel thể hiện như `hrtimer`:
```c
struct hrtimer {
    struct timerqueue_node node;
    ktime_t _softexpires;
    enum hrtimer_restart (*function)(struct hrtimer *);
    struct hrtimer_clock_base *base;
    u8 state;
    u8 is_rel;
};
```
##### HRT setup initialization
HRT setup initialization được thực hiện như sau:


**1. Initialize the hrtimer**: Trước khi hrtimer initialization, cần setup a timer, nó đại diện cho thời gian.
```c
void hrtimer_init(struct hrtimer *timer, clockid_t which_clock, enum hrtimer_mode mode);
```
**2. Starting hrtimer**: hrtimer có thể được bắt đầu như ví dụ sau:
```c
int hrtimer_start(struct hrtimer *timer, ktime_t time, const enum hrtimer_mode mode);
```
`mode` đại diện cho expiry mode. Nó phải là `HRTIMER_MODE_ADS` cho một thời gian tuyệt đối, hoặc `HRTIME_MODE_REL` nếu sử dụng daiga strij thời gian tương đối so với thời điểm hiện tại. 
**3. hrtime cancellation**: Có thể hủy timer hoặc xem là có thể hủy hay không:
```c
int hrtimer_cancel(struct hrtimer *timer);
int hrtimer_try_to_cancel(struct hrtimer *timer);
```
Cả 2 đều return 0 khi timer không active và 1 khi timer active. Khác biết giữa 2 function này là `hrtimer_try_to_cancel` fails nếu timer active hoặc callback của nó đang run, return -1, trong khi `hrtimer_cancel` sẽ chơ cho đến khi callback hoàn thành.


Có thể kiểm tra xem hrtimer callback vấn đang chạy theo như sau:
```c
int hrtimer_callback_running(struct hrtimer *timer);
```
Hãy nhớ rằng `hrtimer_try_to_cancel` gọi bên trong `hrtimer_callback_running`.


**NOTE**: Để ngăn chặn timer tự động restart, hrtimer callback function phải return `HRTIMER_NORESTART`.


Có thể kiểm tra xem liệu HRT có sẵn trong hệ thống mình làm không bằng cách:
- Kiếm kernel config file, file này nên chưa dòng tương tự `CONFIG_HIGH_RES_TIMERS=y`
```bash
zcat /proc/configs.gz | grep CONFIG_HIGH_RES_TIMERS
```
- Tìm kiếm bằng cách `cat /proc/timer_list` hoặc `cat /proc/timer_list | grep resolution`. `.resolution` phải hiển thị 1 nsecs và event_handler phải show `hrtimer_interrupt`.
- Bằng cách sử dụng system call `clock_getres`.
- Từ bên trong kernel code, bằng cách sử dụng `#ifdef CONFIG_HIGH_RES_TIMERS`.


Với HRTs enabled trên hệ thống, độ chính xác của sleep và timer system calls khong phụ thuộc vào **jiffies** nữa, thay vào đó chúng sẽ có độ chính xác tương đương với HRT. Đây cũng là lý do vì sao một số hệ thống không hỗ trợ `nanosleep()`.
### Dynamic tick/tickless kernel
Với option Hz trước đây, kernel sẽ luôn interrupt mỗi giây để schedule task, thậm chí system đang trong trạng thái idle. Nếu Hz set 1000 sẽ có 1000 interrupt mỗi giây, điều này ngay ngăn cản CPU idle trong thời gian dài, làm ảnh hưởng tiêu thụ điện năng của CPU.


Thử xem kernel no fixed hoặc được xác định trước, ở trong đó các tick bị disable cho đến khi có tác vụ nào đó thực thi. Các hệ thống như vậy gọi là **tickless kernel**. Thực tế việc kích hoạt tick được schedule dựa vào action tiếp theo. Tên chính xác hơn là **dynamic tick kernel**. Kernel chịu trách nhiệm cho task schedule và duy trì list những task sẵn sàng chạy. Khi không có task để schedule, scheduler switch tới idle thread, nó enable dynamic tick bằng cách disabling periodic tick cho đến khi hoàn tất timer.


Ở hệ thống nền tảng, kernel cũng duy trì danh sách những task thời gian chờ (timeout) (biết khi nào phải sleep và sleep trong bao lâu). Trong idle state, nếu tick tiếp theo xa hơn so với thời gian chờ ngắn (lowest timeout), kernel sẽ lập trình timer với timeout value. Khi timer expires, kernel re-enable periodic ticks và gọi scheduler, lúc này schedule gắn liên với timeout. Kernel tickless loại bỏ periodic ticks và tiết kiệm năng lượng khi idle.
### Delays and sleep in the kernel
Có 2 loại delay:
- **atomic**
- **non-atomic**
Khai báo header `#include <linux/delay.h>`
#### Atomic context
Các task atomic context (như ISR) không thể sleep, và không thể scheduled. Đó lả lý do tại sao busy-wait loop được sử dungj để tạo độ trễ trong atomic context. Kernel cung cấp `Xdelay` function sẽ tiêu tốn thời gian trong vòng lặp, đủ lâu (dựa vào biến `jiffies`) để đạt được độ trễ.
```c
ndelay(unsigned long nsecs);
udelay(unsigned long usecs);
mdelay(unsigned long msecs);
```
Nên ưu tiên dùng `udelay()` bởi vì `ndelay()` phụ thuộc vào độ chính xác của hardware timer và không phải tất cả các hệ thống đều hỗ trợ nó. Sử dụng `mdelay()` cũng không được khuyến khích.   


Timer handlers (callbacks) thì đã thực thi trong atomic context, có nghĩa là sleep không được phép.
#### Nonatomic context
Trong non-atomic context, kernel cũng cấp hàm `sleep[_range]` và việc sử dụng hàm sẽ phụ thuộc vào việc cần tạo delay bao lâu:
- `udelay` (unsigned lonng usecs): Dựa vào busy-wait loop. Nên sử dụng function này nếu cần sleep trong vài usecs (<~10us).
- `usleep_range` (unsigned long min, unsigned long max): Dựa trên hrtimers, khuyến khích sleep một vài usecs hoặc small msecs (10us - 20ms), giúp tránh việc phải dùng vòng lặp busy-wait loop của `udelay()`.
- `msleep` (unsigned long msecs): được hỗ trợ bởi jiffies/legacy_timer. Nên sử dụng cho thời gian lớn hơn, msecs sleep(10ms+).
**NOTE**: sleep và delay là chủ đề được giải thích rõ ràng trong `Documentation/timers/timers-howto.txt` trong kernel source.
## Kernel locking mechanism
Lock là cơ chế giúp chia sẽ tài nguyên giữa nhiều thread hoặc process khác nhau. Share resource là data hoặc một device có thể được hệ thống truy cập bởi ít nhất 2 user, dù có diễn ra cùng lúc hay không. Các cơ chế khóa giúp ngăn chặn truy cập xung đột/sai mục đích (abusive access).


**Example**: Như khi có một process đang ghi dữ liệu trong khi một process khác đang đọc đúng vị trí đó, hoặc 2 tiến trình cùng tranh giành truy cập vào chung một thiết bị. Kernel cunng cấp nhiều cơ chế khóa khác nhau nhưng quan trọng là:
- Mutex
- Spinlock
- Semaphore
### Mutex
**Mutex** là viết tắt của **mutual exclusion**, là cơ chế khóa được dùng phổ biến nhất hiện nay. Để biết nó làm việc như thế nào xem ở `include/linux/mutex.h`:
```c
struct mutex {
    /* 1: unlocked, 0: locked, negative: locked, possible waiters */
    atomic_t count;
    spinlock_t wait_lock;
    struct list_head wait_list;
};
```
Như đã thấy trong phần **wait queue**, cấu trúc này cũng chưa field thuộc kiểu dữ liệu list: `wait_list`. Nguyên lý sleep cũng tương tự.


Các bên tranh khóa (Contenders) sẽ bị gỡ khỏi scheduler run queue và đẩy vào `wait_list` trong trạng thái sleep. Kernel sẽ lập lịch và thực thi task khác. Khi lock được release, một waiter trong wait queue sẽ được wake-up, đưa ra khỏi `wait_list` và được scheduled back.
#### Mutex API
Sử dụng một mutex chỉ yêu cầu một vài function cơ bản.
##### Declare
- Statically:
```c
DEFINE_MUTEX(my_mutex);
```
- Dynamically:
```c
struct mutex my_mutex;
mutex_init(&my_mutex);
```
##### Accquire and release
- Lock:
```c
void mutex_lock(struct mutex *lock);
int mutex_lock_interruptible(struct mutex *lock);
int mutex_lock_killable(struct mutex *lock);
```
- Unlock:
```c
void mutex_unlock(struct mutex *lock);
```


Thỉnh thoảng cần check liệu mutex đã lock hoặc không. Mục đích đó bạn có thể sử dụng hàm:
```c
int mutex_is_locked(struct mutex *lock);
```
Những gì hàm này làm là kiểm tra liệu mutex này có owner đang rỗng (NULL) hay không. Còn có hàm `mutex_trylock()`, hàm này sẽ lấy khóa mutex nếu nnos hiện chưa bị ai khóa và trả về 1; ngược lại thì nó sẽ trả về 0.
```c
int mutex_trylock(struct mutex *lock);
```
Hàm `mutex_lock_interruptible()` nó được khuyên dùng, sẽ giúp cho driver có khả năng interrupted bằng bất cứ signal. Trong khi đó, với hàm `mutex_lock_killable()`, chỉ những signal kill processs có thể interrupt driver.


Nên cẩn thận `mutex_lock()`, và sử dụng nó khi có thể đảm bảo rằng mutex sẽ được release, dù cho bất cứ cái gì xảy ra. Trong user context, khuyên luoonn dùng `mutex_lock_interruptible()` để xin cấp khóa mutex, bởi vì hàm `mutex_lock()` sẽ không return nếu signal đã nhận được signal (thậm chí Ctrl + C).


**Example**:
```c
struct mutex my_mutex;
mutex_init(&my_mutex);

/* inside a work or a thread */
mutex_lock(&my_mutex);
access_shared_memory();
mutex_unlock(&my_mutex);
```
Vui lòng tìm kiếm ở `include/linux/mutex.h` trong kernel source 
### Spinlock
**1. Cơ chế hoạt động (Spinning vs sleeping)**
- **Spinlock** sử dụng active-loop: Luồng sẽ spin (xoay vòng) liên túc để kiểm tra khóa cho đến khi lấy được thì thôi.
- Khác với **mutex** (cho luồng sleep và nhường CPU), Spinlock chiếm giữ CPU hoàn toàn trong khi chờ.
**2. Khi nào nên dùng?**
- Chỉ dùng cho các tác vụ cực ngắn (vùng tranh chấp nhỏ).
- Sử dụng khiL THời gian giữ khóa < Thời gian hệ điều hành cần để điều phối luồng  (reschedule/context switch).
**3. Tương tác với hệ điều hành (Preemption)**
- Khi một luồng giữ Spinlock, kernel sẽ vô hiệu hóa quyền ưu tiên (disable preemption).
- **Mục đích:** Đảm bảo luồng đang giữ khóa không bị "đuổi" ra khỏi CPU, vì nếu nó bị tạm dừng, các luồng khác sẽ phải xoay vòng chờ đợi vô ích trong thời gian rất dài.
**4. Trên hệ thống đơn nhân (Single Core)**
- Sử dụng Spinlock trên máy single core là vô nghĩa và dễ gây deadlock (bế tắc) hoặc làm chậm hệ thống.
- Trên single core, hàm `spin_lock()` thực tế chỉ còn nhiệm vụ là vô hiệu hóa quyền ưu tiên (preemption).


Trong system **single core** nên dùng `spin_lock_irqsave()` và `spin_unlock_irqrestore()`, nó sẽ vô hiệu hóa interrupts trên CPU, giúp ngăn chặn tình trạng tranh chấp do interrupt gây ra (interrupt concurrency).


Vì khi viết driver không biết sẽ chạy trên hệ thống nào nên dùng `spin_lock_irqsave(spinlock_t *lock,unsigned long flags)`, nó sẽ disable interrupt trên hệ thống trước khi lấy spinlock. Bên trong nó calls `local_irq_save(flags);`, hàm này phụ thuốc vào kiến trúc để lưu lại trạng thái của IRQ hiến tại, và sau đó là `preempt_disable()` để disable preeption trên CPU. Nên gọi `spin_unlock_irqrestore()` nó sẽ làm ngược với những bước trên.
```c
/* some where */
spinlock_t my_spinlock;
spin_lock_init(my_spinlock);

static irqreturn_t my_irq_handler(int irq, void *data)
{
    unsigned long status, flags;
    spin_lock_irqsave(&my_spinlock, flags);
    status = access_shared_resources();
    spin_unlock_irqrestore(&gpio->slock, flags);
    return IRQ_HANDLED;
}
```
#### Spinlock versus mutexes
Tranh chấp (concurrency) trong kernel, **spinlock** và **mutex** cả 2 đều có mục tiêu riêng biệt:
- **Mutex**: protect resource của process, spinlock protect những section quan trong IRQ handler.
- **Mutex** sleep cho đến khi lấy được khóa, spinlock spin loop cho đến khi lấy được khóa.
**NOTE**: Preemption sẽ bị disable đối với thread đang giữ spinlock.
## Work deferring mechanism
Defering (trì hoãn) là phương pháp giúp lập kế hoạch để một phần công việc được thực thi trong tương lai. Trì hoãn các hàm (bất kể loại nào) để gọi và thực thi chúng sau. Có 3 cách:
- **SoftIRQs**: thực thi trong atomic context.
- **Tasklets**: thực thi trong atomic context.
- **Workqueues**: thực thi trong process context.

### So sánh giữa Atomic Context và Process Context trong Linux Kernel

| Đặc điểm | Atomic Context (Ngữ cảnh nguyên tử) | Process Context (Ngữ cảnh tiến trình) |
| :--- | :--- | :--- |
| **Đại diện** | Interrupts, SoftIRQs, Tasklets | Kernel threads, System calls, Workqueues |
| **Khả năng ngủ (Sleep)** | **Tuyệt đối không** | **Có thể** |
| **Khả năng bị lập lịch** | Không (Chạy liên tục cho đến khi xong) | Có (Có thể bị Scheduler tạm dừng) |
| **Truy cập User Space** | Không thể | Có thể (Dùng `copy_to_user`, `copy_from_user`) |
| **Cơ chế khóa (Locking)** | Spinlocks | Mutexes, Semaphores |
| **Mức độ ưu tiên** | Rất cao (Đáp ứng phần cứng) | Thấp hơn (Xử lý tác vụ thông thường) |
### Softirqs and ksoftirqd
### Tasklets
### Tasklet scheduling
### Work queues
### Kernel threads
## Threaded IRQs
### Threaded bottom half
## Invoking user space applications from the kernel

