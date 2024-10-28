
- [[#Table Of Content|Table Of Content]]
- [[#Big Data là gì?|Big Data là gì?]]
- [[#Hadoop và các thành phần|Hadoop và các thành phần]]
	- [[#Hadoop và các thành phần#Tổng quan về kiến trúc hadoop|Tổng quan về kiến trúc hadoop]]
		- [[#Tổng quan về kiến trúc hadoop#1. Kiến trúc HDFS|1. Kiến trúc HDFS]]
		- [[#Tổng quan về kiến trúc hadoop#2. Hadoop cluster:|2. Hadoop cluster:]]
		- [[#Tổng quan về kiến trúc hadoop#3. MapReduce Framework Hadoop|3. MapReduce Framework Hadoop]]
- [[#Kiến trúc HDFS|Kiến trúc HDFS]]
		- [[#Tổng quan về kiến trúc hadoop#1. NameNode|1. NameNode]]
		- [[#Tổng quan về kiến trúc hadoop#2. DataNode|2. DataNode]]
		- [[#Tổng quan về kiến trúc hadoop#3. Secondary NameNode|3. Secondary NameNode]]
		- [[#Tổng quan về kiến trúc hadoop#4. Block trong HDFS|4. Block trong HDFS]]
		- [[#Tổng quan về kiến trúc hadoop#5. Rack Awareness|5. Rack Awareness]]
- [[#Đọc ghi dữ liệu trên HDFS|Đọc ghi dữ liệu trên HDFS]]
		- [[#Tổng quan về kiến trúc hadoop#1. Ghi dữ liệu trên HDFS|1. Ghi dữ liệu trên HDFS]]
		- [[#Tổng quan về kiến trúc hadoop#2. Đọc dữ liệu trong HDFS|2. Đọc dữ liệu trong HDFS]]

## Big Data là gì?
- Big data là một tập dữ liệu lớn không thể xử lý bằng các phương pháp truyền thống.
- 3 đặc tính của Big Data:
	- Volume: Kích thước, độ lớn của dữ liệu.
	- Velocity: tốc độ xử lý dữ liệu (có dữ liệu rồi, phải xử lý dữ liệu đó thì nó mới có ý nghĩa)
	- Variety: tính đa dạng của dữ liệu (unstructured , voice, video, ...)

## Hadoop và các thành phần
- Hadoop xuất phát từ google file system, sau này được phát triển thành hệ thống Hadoop (HDFS)

![[Pasted image 20241028220255.png]]

### Tổng quan về kiến trúc hadoop

https://demanejar.github.io/posts/hadoopo-ecosystem/

![[Pasted image 20241028220512.png]]

Apache Hadoop là 1 framework hỗ trợ cho việc lưu trữ và xử lý dữ liệu phân tán trên nhiều máy. Nó được thiết kế để có khả năng scale trên hàng ngàn máy (node).

Thành phần:
- Hadoop Common: các utils hỗ trợ Hadoop module
- HDFS: Hệ thống file phân tán
- Hadoop YARN: Framework dùng cho lập trình job và quản lý tài nguyên hệ thống
- MapReduce: Hệ thống xử lý dữ liệu của Hadoop

#### 1. Kiến trúc HDFS
Hadoop Distributed File System (HDFS): là một hệ thống file phân tán được thiết kế để có thể chạy trên các hệ thống phần cứng thông thường.
- Khả năng chịu lỗi cao (fault-tolerant)
- Deploy trên các phần cứng thông thường
- Khả năng truy cập (high throughput)
- Khả năng scale tốt khi muốn mở rộng hệ thống

Kiến trúc HDFS:
![[Pasted image 20241028221349.png]]

Gồm 2 thành phần chính là Namenode và Datanode theo cơ chế master-slave. Namenode đóng vai trò là người quản lý toàn bộ hoạt động.

#### 2. Hadoop cluster:
![[Pasted image 20241028221542.png]]

Trong hệ thống có hàng ngàn máy (node), các node sẽ được sắp xếp theo từng rack 
![[Pasted image 20241028221642.png]]

#### 3. MapReduce Framework Hadoop

MapReduce là framework dùng để xử lý dữ liệu lớn có thể lên tới hàng petabyte một cách song song trên hàng ngàn node với tốc độ cao. MapReduce gồm 2 pha chính là map và reduce.
 ![[Pasted image 20241028221757.png]]


## Kiến trúc HDFS

#### 1. NameNode

Lưu trữ thông tin metadata (fsimage, edit logs). Trong đó:
- fsimage quản lý thông tin về location của data.
- edit logs quản lý thao tác đọc ghi dữ liệu của hệ thống, client hoặc thao tác như replicate data.

Quản lý user truy cập vào hệ thống (gán quyền cho user truy cập vào file). 

Giao tiếp với client cho thao tác đọc ghi dữ liệu.

Cung cấp dịch vụ đăng ký DataNode mới trong cụm. Ex: Khi một máy muốn kết nối vào cụm, nó phải đăng ký một dịch vụ trong NameNode để thông báo rằng muốn đăng ký vào cụm đó, NameNode sẽ cung cấp thông tin cần thiết về server mà client cần.

Nhận heartbeat của DataNode. Ex: Trong 3 phút, DataNode không gửi heartbeat về cho NameNode, DataNode sẽ gần như là mặc định cái DataNode đã bị fail và sẽ thực hiện copy data từ DataNode fail này sang DataNode khác.

Xác định location của file, replicated file.

*# Kiến trúc NameNode trong Hadoop version 1 được coi là Single Point Failure (khi có lỗi xảy ra, phải khởi động lại bằng tay, phải lấy thông tin từ SecondaryNameNode để khôi phục lại)*

Trong kiến trúc Hadoop version 2, Hadoop đảm bảo được high availability. Cho phép cài thêm NameNode ở trạng thái StandBy, khi NameNode ở trạng thái Active bị lỗi thì NameNode ở tráng thái StandBy sẽ được chỉnh sang Active và thay thế cho NameNode lỗi kia.
#### 2. DataNode

Cung cấp các block để lữu trữ file.

Giao tiếp với client cho thao tác đọc ghi.

Khởi tạo và xóa block data.

Replicate data trong cụm.

Giữ liên lạc với NameNode thông qua việc gửi heartbeat theo định kỳ.

#### 3. Secondary NameNode

Lưu các thông tin trên fsimage của NameNode theo định kỳ.

Phục vụ cho việc khôi phục lại thông tin metadata trên NameNode khi gặp sự cố.

![[Pasted image 20241028223417.png]]

#### 4. Block trong HDFS

Trong Hadoop version 1: kích thước của block là 64MB.
Trong Hadoop version 2: kích thước của block là 128MB và có thể lên tới 256MB.

![[Pasted image 20241028223953.png]]

Các block được lữu trữ trên DataNode và được duplicate.

#### 5. Rack Awareness

Làm tăng khả năng trao đổi dữ liệu, dễ dàng cho việc quản lý.

Không có quá 1 block trên 1 node, không có quá 2 block trên cùng 1 rack. Ex: Khi Rack - 1 lỗi, thì vẫn còn dữ liệu trên Rack - 2.

➡ Fault tolerant

![[Pasted image 20241028224144.png]]

*# Kích thước của block trong HDFS có ảnh hưởng như thế nào đến việc lưu trữ dữ liệu và xử lý dữ liệu trên HDFS?*

**Kích thước block lớn** phù hợp khi xử lý các tập dữ liệu lớn và giúp giảm tải cho NameNode.

**Kích thước block nhỏ** thì linh hoạt hơn cho các tác vụ xử lý nhỏ, nhưng cần cân nhắc để tránh tăng quá mức lượng metadata.

- Khi kích thước block lớn, một tệp sẽ được chia thành ít block hơn, do đó hệ thống sẽ cần ít metadata (dữ liệu mô tả) hơn để lưu trữ thông tin về các block. Điều này giúp tiết kiệm bộ nhớ ở NameNode, giảm tải việc quản lý metadata và tăng hiệu suất.
- Kích thước block lớn cũng giúp giảm bớt số lượng các yêu cầu truy xuất tới các block khác nhau khi xử lý dữ liệu. Do đó, các tác vụ xử lý sẽ ít phải chuyển đổi giữa các block, cải thiện hiệu suất khi chạy các ứng dụng lớn. Tuy nhiên, nếu block quá lớn, việc truyền tải và xử lý mỗi block cũng sẽ lâu hơn, nhất là khi chỉ cần xử lý một phần nhỏ của dữ liệu trong block.
- Kích thước block nhỏ dẫn đến việc chia dữ liệu thành nhiều block hơn, nghĩa là NameNode sẽ cần nhiều metadata hơn để quản lý các block này. Điều này có thể gây áp lực lên bộ nhớ và giảm hiệu suất của NameNode nếu lượng dữ liệu lớn.
- Với block nhỏ, hệ thống có thể xử lý từng phần dữ liệu nhỏ hơn một cách linh hoạt, phù hợp với những trường hợp chỉ cần truy xuất một phần nhỏ của dữ liệu. Tuy nhiên, việc xử lý nhiều block cũng tạo ra nhiều yêu cầu I/O và overhead, khiến cho quá trình xử lý tổng thể bị chậm hơn.

## Đọc ghi dữ liệu trên HDFS

#### 1. Ghi dữ liệu trên HDFS

1. Client gửi request ghi file đến cho hệ thống qua các `DistributedFileSystem` APIs.
2. Sau đó, `DistributedFileSystem` sẽ tạo ra 1 yêu cầu RPC gửi tới cho NameNode, NameNode cấp lại location có thể ghi file vào.
3. Client sẽ thực hiện ghi file thông qua `FSDataOutputStream`, dữ liệu client muốn ghi sẽ được đẩy vào queue, DataStreamer sẽ lấy data trong queue để ghi vào DataNode. 
4. Dữ liệu sẽ được ghi trên 1 DataNode và hệ thống sẽ tự động replicate data thành `n` bảng khác lưu trên DataNode khác nhau.
5. Sau khi ghi xong trên `n` bảng, DataNode sẽ gửi lại message `acknowledgement` để xác nhận cho `FSDataOutputStream`.
6. Client sẽ thực hiện close file vừa ghi.
7. Hệ thống `DistributedFileSystem` sẽ báo lại cho NameNode về việc ghi xong dữ liệu.


![[Pasted image 20241028225325.png]]

#### 2. Đọc dữ liệu trong HDFS

1. Client gửi request đọc file đến cho hệ thống thông qua các `DistributedFileSytem` APIs.
2. Sau đó `DistributedFileSystem` sẽ tạo ra 1 yêu cầu RPC gửi tới cho NameNode để lấy về location của block.
3. `DistributedFileSystem` sẽ sinh ra `FSDataInputStream` lưu các location gần với cả client nhất để đọc dữ liệu.
4. Dữ liệu sẽ được đọc trong các DataNode, khi đọc xong block sẽ chuyển sang block khác.
5. Trong trường hợp block đang đọc bị lỗi, hệ thống sẽ chuyển sang đọc dữ liệu của block đó trên DataNode khác gần nhất.
6. Khi đọc xong, client sẽ gửi lệnh close file đến cho hệ thống.

![[Pasted image 20241028230215.png]]


*# Hệ thống xử lý như thế nào khi client đang ghi dữ liệu thì DataNode fail?*

Block hiện tại sẽ được ghi mới lại trên 1 DataNode khác hoạt động bình thường khác, còn block bị ghi một phần trên DataNode bị hỏng sẽ được remove đi khi DataNode nó được khôi phục.

![[Pasted image 20241028231246.png]]