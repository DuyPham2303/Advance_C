# 1. PROCESS và THREAD

<p align = "center">
<img src = "https://github.com/user-attachments/assets/cc1f87c5-36f5-41ea-a4ef-836fad33e2fc"height = "300" width = "700">

## 1.1 Process (tiến trình) 
+ Là các Chương trình chạy trên hệ thống
+ Có địa chỉ riêng 
+ Tương tác được với ngoại vi khác phần cứng bên ngoài
+ 1 process có thể chứa nhiều thread
+ Giao tiếp giữa các process thông qua 
## 1.2 Thread (Luồng
+ Là Đơn vị nhỏ nhất của 1 process 
+ Các luồng trong cùng process chia sẻ tài nguyên chung
+ Có thể hiểu là 1 task chia thành nhiều câu lệnh -> các task sẽ được phân chía thời gian thực hiện xen kẽ nhau
## 1.3 Ứng dụng của thread

__+ Thao tác đa nhiệm trên máy tính mà ta sử dụng hằng ngày:__ Đây là ví dụ rõ nhất cho việc ứng dụng của thread trong cách mà nó xử lý các tác vụ. Mỗi khi ta chạy 1 chương trình nào đó ví dụ như chrome __(tiến trình)__ thì nhờ vào cơ chế đa luồng __(multithread)__ mà ta có thể mở nhiều tab cùng 1 lúc

__+ Giao tiếp giữa các thành phần trong 1 hệ thống điều khiển thời gian thực:__ Các hệ thống như ô tô với nhiều cảm biến và yêu cầu cập nhật dữ liệu và phản hồi liên tục để phối hợp với các thành phần khác để điều khiển được xe thì cần phải sử dụng cơ chế đa luồng để có thể xử lý nhanh chóng, tránh gây chậm trễ dữ liệu dẫn tới mất điều khiển bị sai sót gây mất an toàn cho người lái.

# 2. Lập trình thread

## 2.1 Cơ bản về luồng

### a) Khởi tạo luồng

__Cú pháp__ : std::thread thread_id(function,args...)

__+ thread_id__ : khởi tạo việc thực thi luồng (không kiểm tra trạng thái)

__+ args...__ : macro variadic (nhận số lượng tham số không xác định)

__+ function:__: hàm thực thi hoặc lambda

=> mỗi luồng chỉ thực thi 1 hàm duy nhất, Khi luồng chính kết thúc thì các luồng khác cũng kết thúc

__Ví dụ: tạo ra 1 luồng và thực thi mỗi 1s__ 

```bash
#include<iostream>
#include<stdint.h>
#include<thread>
#include<chrono> //cung cấp các hàm để tạo delay
using namespace std;
void delay(uint32_t time){
    this_thread::sleep_for(chrono::seconds(time)); //hàm tạo delay 1s
}
void task1(uint32_t limi,string str){
    int i = 0;
    while(i < limit>){
        cout << str << i++ << endl;
        delay(1);
    }
}
int main(){
    thread t1(task1,30,"task 1: "); // khởi tạo và chạy 
    return 0;
```

Khi chạy chương trình trên sẽ bị dừng ngay lập tức. Lý do là vì sau khi luồng t1 được khởi tạo trong hàm main __(luồng chính)__ thì trong hàm main không còn tác vụ nào nữa nên có sẽ kết thúc dẫn đến các luồng phụ như t1 __(phụ thuộc vào luồng chính)__ cũng kết thúc theo.

=> Do đó ta sẽ cần 1 số method để quản lý các luồng phụ này
### b) QUẢN LÝ LUỒNG
Thư viện đa luồng của C++ cung cấp 1 số hàm để thao tác với 1 luồng như sau

__Hàm join()__
+ Chờ cho đến khi 1 luồng thực thi xong thì luồng khác mới được thực thi
+ Khi 1 luồng phụ được khai báo nó sẽ phụ thuộc vào luồng chính  khi luồng chính không gọi ra __join()__ thì nó sẽ kết thúc và không chờ luồng phụ thực hiện -> lúc này chương trình sẽ bị xung đột (lỗi)
__=> Luôn đảm bảo luồng phụ kết thúc trước luồng chính__

__Ví dụ tạo ra 2 luồng chạy song song và kết thúc trước khi chạy luồng chính__ 

```bash
int main() {
    thread t1(task1, 10, "task 1: ");
    thread t2(task1, 5, "task 2: ");

    //luồng chính phải chờ 2 luồng t1 và t2 thực hiện xong mới chạy -> t1 và t2 sẽ chạy song song
    t1.join();
    t2.join();

    for (int i = 0; i < 5; i++) {
        cout << "this is main thread" << endl;
        delay(1);
    }
}
```
__+ Kết quả:__

<p align = "center">
<img src = "https://github.com/user-attachments/assets/b44bbf6f-899e-4060-9eea-612bae3849b3"height = "280" width = "600">


__Ví dụ tạo ra 2 luồng chạy song song với luòng chính__ 


```bash
int main() {
    thread t1(task1,5, "task 1: ");
    thread t2(task1,3, "task 2: ");
    for (int i = 0; i < 8; i++) {
        cout << "this is main thread" << endl;
        delay(1);
    }
    //Cả 3 luồng đều chạy song song 
    t1.join();
    t2.join();
}
```

__+ Kết quả:__

<p align = "center">
<img src = "https://github.com/user-attachments/assets/f4453150-9687-44bd-b16c-6472afc77f56"height = "280" width = "600">
    
### Hàm joinable()

+ kiểm tra 1 luồng đã kết thúc hoặc đã gọi join() chưa
+ return true -> khi luồng chưa kết thúc/chưa gọi join()
+ return false -> khi luồng đã kết thúc

```bash
int main() {
    thread t1(task1,5, "task 1: ");
    thread t2(task1,3, "task 2: ");
    for (int i = 0; i < 8; i++) {
        cout << "this is main thread" << endl;
        delay(1);
    }

    t2.join();

    if (t1.joinable()) { //kiểm tra t1 đã gọi method join() chưa
      cout << "luong t1 chua ket thuc";
      t1.join();
    }
    else cout << "luong 1 da ket thuc";
}
```




### Hàm detach()
+ cho phép luồng phụ tách khỏi luồng chính và chạy độc lập
+ Luồng chính không cần chờ hàm phụ kết thúc
+ Chương trình sẽ kết thúc nếu luồng chính kết thúc trước luồng phụ (xóa kết quả luồng phụ -> không bị xung đột)
+ Nếu gọi join trước detach và luồng chính kết thúc trước luồng gọi detach -> báo lỗi do gọi ra luồng đã kết thúc


```bash
int main() {
  thread t1(task1, 10, "task 1: ");
  thread t2(task2, 6, "task 2: ");
  t1.detach(); //cho phép t1 chạy song song với luồng chính 

  for (int i = 0; i < 8; i++) {
      cout << "this is main thread" << endl;
      delay(1);
  }

  t2.join();
```
Chương trình trên t1 sẽ chạy độc lập so với main, trong khi t2 vẫn phụ thuộc vào main. Chính vì vậy t2 sẽ luôn kết thúc trước main do sử dụng __join()__, Do đó khi t2 hoàn thành thì main cũng kết thúc luôn và khi t1 vẫn còn chạy do không phụ thuộc vào main, nó cũng sẽ bị dừng ngay lập tức mặc dù vẫn chưa hoàn thành xong công việc

__Lưu ý__


__=> Đảm bảo luồng phụ sử dụng detach() luôn hoàn thành trước khi luồng chính kết thúc__


__=> tránh việc gọi join() và detach() của cùng 1 luồng trong chương trình trên 1 luồng vì có thể gây lỗi gọi ra 1 luồng đã kết thúc__ 



###  Khi nào dùng join() hay detach()

<p align = "center">
<img src = "https://github.com/user-attachments/assets/a2a2200b-f065-4cae-8f91-17e1061403a5" height = "350" width = "800">


+ __join()__ : Khi kết quả task cần được xử lý bởi luồng chính (tài nguyên của luồng chính và phụ chia sẻ với nhau)

+ __detach()__:  Khi đọc dữ liệu từ nhiều cảm biến khác nhau (công việc của luồng phụ không ảnh hưởng đến luồng chính)



## 2.2 Luồng đồng bộ (Synchronous)

### a) Atomic 
+ là 1 struct template cho phép khai báo 1 biến toàn cục chia sẻ giữa các luồng để xử lý
+ cho phép nhiều luồng cùng truy cập vào tài nguyên nhưng không bị gián đoạn việc thao tác trên dữ liệu được chia sẻ chung giữa các luồng 

=> có nghĩa là khi 1 luồng đang sử dụng biến atomic nó sẽ không bị tác động bởi những luồng khác

__So sánh với chương trình không sử dụng Atomic__


<p align = "center">
<img src = "https://github.com/user-attachments/assets/a4b7c0db-e229-42cb-8dc7-7628f32c62e5" height = "280" width = "500">


Khi nhiều luồng cùng truy cập vào 1 biến thì nó sẽ thực hiện thao tác ghi đè lẫn nhau, ví dụ như 

=> cả 2 luồng cùng thực hiện tăng giá trị biến, Tuy nhiên chương trình sẽ chỉ ghi nhận 1 lần tăng dẫn đến số lần thực hiện của các luồng bị sai sót. 

=> 1 luồng thực hiện tăng giá trị tuy nhiên lại có 1 luồng thực hiện việc đọc giá trị cùng lúc, dẫn đến kết quả xử lý của luồng này bị ảnh hưởng đến luồng khác. 

__+ LƯU Ý:__ 

+ atomic chỉ phù hợp với các thao tác đơn giản, chẳng hạn tăng/giảm hoặc hoán đổi giá trị. Đối với các logic phức tạp (như cập nhật nhiều biến hoặc thao tác trên cấu trúc dữ liệu lớn), bạn cần sử dụng mutex.

+ chỉ sử dụng cho biến toàn cục
+ áp dụng trong việc chia sẻ dữ liệu cảm biến giữa các luồng
+ Chưa đảm bảo tuyệt đối việc cho phép 1 luồng truy cập vào tài nguyên chung ở 1 thời điểm -> sử dụng mutex sẽ tối ưu hơn

__Ví dụ:__

```bash
#include<iostream>
#include<stdint.h>
#include<thread>
#include<chrono> //cung cấp các hàm để tạo delay
#include<atomic>

void delay_ms(uint32_t time) {
    this_thread::sleep_for(chrono::milliseconds(time));
}

void taskAtomic(uint32_t limit, string str) {
    while(sharedcnt < limit) {
        cout << str << ++sharedcnt << endl;
        delay_ms(250);
    }
}
int main() {
    thread t1(taskAtomic, 10, "task 1: ");
    thread t2(taskAtomic, 5, "task 2: ");
    t1.join();
    t2.join();

}
```

__Kết quả__

<p align = "center">
<img src = "https://github.com/user-attachments/assets/35ad1221-cae0-44c7-a45f-d430321e5d51" height = "200" width = "500">


Chương trình vẫn chưa tối ưu khi 1 số dòng bị dính vào nhau gây ra do sự chậm trễ trong quá trình xử lý bởi hàm cout khi 1 luồng chưa kịp xử lý lệnh xuống dòng endl thì luồng khác đã can thiệp. Chính vì vây ta sẽ có 1 giải pháp tôi ưu hơn để khắc phục tình trạng này 

### b) Mutex

<p align = "center">
<img src = "https://github.com/user-attachments/assets/aa02dd69-e03f-4f61-bd3f-cdaab9413b32" height = "350" width = "600">

+ đảm bảo chỉ có 1 luồng duy nhất truy cập vào tài nguyên tại 1 thời điểm -> tối ưu phần Atomic
+ Sử dụng cơ chế khóa mutex để lock/unlock tài nguyên 

__Cơ ché hoạt động:__

+ Khi 1 luồng truy cập vào tài nguyên chung, cơ chế mutex sẽ khóa lại tài nguyên đó
+ Những luồng khác sẽ đợi đến khi tài nguyên được mở khóa 
+ Khi luồng đó thực thi xong, nó sẽ mở khóa mutex. Cho phép luồng khác truy cập vào tài nguyên. 

__Các cơ chế của mutex__

### lock guard 
+ Cung cấp cơ chế lock/unlock tự động 
+ Giúp bảo vệ tài nguyên chung không bị thay đổi ngoài ý muốn

__Vi dụ:__

```bash
#include<iostream>
#include<stdint.h>
#include<thread>
#include<chrono> //cung cấp các hàm để tạo delay
#include<atomic>
#include<mutex>
using namespace std;
atomic<int> sharedcnt(0); //biến đếm số lần các luồng thực thi 

mutex key; //Định nghĩa 1 khóa mutex để quản lý việc truy cập tài nguyên chung

void delay_ms(uint32_t time) {
    this_thread::sleep_for(chrono::milliseconds(time));
}
void taskglock(uint32_t limit, string str) {
    while (sharedcnt <= limit) {
        {
            lock_guard<mutex> glock(key); //kích hoạt cơ chế lock guard để khóa tài nguyên lại
            cout << str << sharedcnt++ << endl;
        }
        //mutex được unlock ở đây khi ra khỏi phạm vi định nghĩa
        delay_ms(500);
    }
}
int main() {
    thread t1(taskglock, 10, "task 1: ");
    thread t2(taskglock, 15, "task 2: ");

    t1.join();
    t2.join();

}
```

__Kết quả__ : Luồng t1 và t2 sẽ được thực hiện xen kẽ nhau cùng 1 lúc cứ mỗi 1 250ms 

<p align = "center">
<img src = "https://github.com/user-attachments/assets/ed3bbd6d-8ce3-4c06-bfd0-a48248c03f0b" height = "250" width = "600">


### Unique lock

+ Cho phép lock/unlock 1 cách thủ công hoặc tự động 
+ Ứng dụng trong việc chọn khối lệnh sẽ thực hiện để xử lý tài nguyên chung nào đó chia sẻ giữa nhiều luồng


```bash
#include<iostream>
#include<stdint.h>
#include<thread>
#include<chrono> //cung cấp các hàm để tạo delay
#include<atomic>
#include<mutex>
using namespace std;
atomic<int> sharedcnt(0); //biến đếm số lần các luồng thực thi 

mutex key; //Định nghĩa 1 khóa mutex để quản lý việc truy cập tài nguyên chung

void delay_ms(uint32_t time) {
    this_thread::sleep_for(chrono::milliseconds(time));
}
void taskunilock(uint32_t limit, string str) {
    while (sharedcnt <= limit) {
        unique_lock<mutex> unilock(key);
        cout << str << sharedcnt++ << endl;
        unilock.unlock(); //mở khóa mutex để các luồng khác truy cập 
        delay_ms(500); //hàm này vẫn thực thi song song với các luồng 
    }
}
int main() {
    thread t1(taskunilock, 10, "task 1: ");
    thread t2(taskunilock, 15, "task 2: ");

    t1.join();
    t2.join();

}
```

### condition variable

+ Là 1 class cung cấp các cơ chế giao tiếp và đồng bộ cho phép 1 luồng chờ cho đến khi 1 điều kiện cụ thể được đáp ứng, và có thể gửi thông báo khi điều kiện được thực hiện 

+ Áp dụng khi 1 luồng cần chờ dữ liệu từ 1 luồng khác để xử lý
__Các method__ 
+ wait(): Cho phép 1 luồng chờ cho đến khi 1 điều kiện được thực hiện xong 
-> liên tục kiểm tra điều kiện xử lý từ luồng khác bằng cách unlock tạm thời shared resource -> nếu true thì sẽ dừng wait -> xử lý data 
+ notify_one(): gửi thông báo xử lý đến 1 luồng
+ notify_all(): gửi thông báo xử lý đến tất cả các luồng

__Định nghĩa các biến cần thiết__

```bash
#include <iostream>
#include <thread>
#include <mutex>
#include <atomic>
#include <chrono>
#include <condition_variable>
using namespace std;

atomic<int> sensor_data(0);  //dữ liệu được chia sẻ giữa 2 luồng đọc và xử lý
mutex key;                   //khóa mutex để bảo vệ việc truy cập dữ liệu     
condition_variable signal;   //đối tượng để cho phép 1 luồng thông báo đến luồng khác 
bool IsdataRcv = false;      //lưu trang thái thực thi của luồng 
```

__Hàm đọc dữ liệu__


```bash

void sensor_read() {
    while (1) {
        //wait a bit
        delay_ms(200);
        
        //lock shared resource to get sensor data
        unique_lock<mutex> unilock(key);
        
        //read data
        sensor_data = rand() % 100;
        
        //update action status and print out behaviour
        IsdataRcv = true;
        cout << "done read data" << endl;
        
        //lock shared resources 
        unilock.unlock();
        
        //fire notification to execute action with received data
        signal.notify_one();
    }
}
```

__Hàm xử lý dữ liệu__
```bash
void process_data() {
    while (1) {
        //delay a bit
        delay_ms(200);
        
        //unlock resources to proces data
        unique_lock<mutex> unilock(key);
        
        //wait until other thread fire notification
        signal.wait(unilock, []() {return IsdataRcv; });
        
        //reset read status and execute action -> print out notification
        cout << "data: " << sensor_data << endl;
        IsdataRcv = false;
        
        //unlock data when done processed
        unilock.unlock();
    }
}
```

__Xử lý trong luồng chính như sau__
```bash
int main() {
//tạo và chạy luồng đọc dữ liệu
    thread thread_read(sensor_read); 
//tạo và chạy luồng xử lý dữ liệu
    thread thread_process(process_data);

    thread_read.join();
    thread_process.join();
    return 0;
}
```

__Kết quả__

<p align = "center">
<img src = "https://github.com/user-attachments/assets/3f3af358-8131-4bf2-be35-711c19bc92f4" height = "250" width = "600">


## 2.3 LUỒNG BẤT ĐỒNG BỘ (Asynchrnous)
+ Là luồng cung cấp các cơ chế cho phép chạy Độc lập so với luồng chính

### a) So sánh với detach()

__Giống nhau__

đều chạy độc lập/song song với luông chính

__Khác nhau__

+ khi luồng chính kết thúc

=> detach() cũng sẽ kết thúc theo, không lấy được kết quả từ luồng chạy detach()

=> Luồng Async vẫn chạy và trả về kết quả  

### b) Cơ chế shared_future và future

+ Là những class template dùng để lưu trữ kết quả của luồng bất đồng bộ __nằm ở heap__
+ __future:__ đảm bảo không có luông nào truy cập đến kết quả này __(tương tự unique smart pointer)__
+ __shared_future:__  cho phép kết quả trả về được truy cập bởi nhiều luồngluồng
+ đọc về kết quả thông qua method __get()__ lúc này heap cũng được release luôn

__Cú pháp__

__future<T> async(launch_policy,callable&& func,Args&&... args);__

__+ launch_policy__ chế độ chạy (2 mode)
- async: tạo xong chạy luôn
- deferred: tạo xong đợi khi nào gọi mới chạy

__+ func__ hàm thực thi local/global 

__+ Args__ 

__Ví dụ về 1 luồng đọc dữ liệu và chạy song song với 1 luồng khác, và trả về kết quả sau khi tất cả các luồng hoàn thành__ 

__Khai báo thư viện và định nghĩa 1 số biến cần thiết__

```bash
#include <iostream>
#include <thread>
#include <mutex>
#include <atomic>
#include <chrono>
#include <future>
using namespace std;
atomic<int> sharedcnt(0); // shared variable used among multi-thread
atomic<int> sensor_data(0);
atomic<bool> IsdataRcv(false);
mutex key;                // permission mechanism to access shared resources among multi-thread
condition_variable signal;

void delay_ms(uint32_t time) {
    this_thread::sleep_for(chrono::milliseconds(time));
}
```


__Hàm để đọc dữ liệu__
```bash
int asyn_read(uint32_t limit) {
    for (int i = 0; i < limit; i++) {
        unique_lock<mutex> unilock(key);
        cout << "read sensor data -> count event: " << i << endl;
        unilock.unlock();
        delay_ms(100); 
    }
    sensor_data = rand() % 100; //generate random data
    cout << "done read data" << endl;
    return sensor_data;
}

__
```

__Tạo các luồng và chạy__

```bash
int main() {
    //store returned data at heap 
    int event_count = 0; //biến đếm số lần thực thi 
    future<int> unique_result = async(launch::async, asyn_read,event_count);   //does not allow other thread to access result      

    for (int i = 0; i < 15; i++) {
        unique_lock<mutex> unilock(key);
        cout << "main execute -> event count: " << i++ << endl;
        unilock.unlock();
        delay_ms(500);
    }
    cout << "sensor data: " << unique_result.get() << endl;

    return 0;
}
```

__Kết quả chạy chương trình__

<p align = "center">
<img src = "https://github.com/user-attachments/assets/cef5fae2-c16f-43e0-95d9-7dcb7d047423" height = "250" width = "600">



