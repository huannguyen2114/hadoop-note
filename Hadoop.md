- [Table Of Content](#table-of-content)
- [Big Data là gì?](#big-data-là-gì)
- [Hadoop và các thành phần](#hadoop-và-các-thành-phần)
  - [Tổng quan về kiến trúc hadoop](#tổng-quan-về-kiến-trúc-hadoop)
    - [1. Kiến trúc HDFS](#1-kiến-trúc-hdfs)
    - [2. Hadoop cluster](#2-hadoop-cluster)
    - [3. MapReduce Framework Hadoop](#3-mapreduce-framework-hadoop)
- [Kiến trúc HDFS](#kiến-trúc-hdfs)
  - [1. NameNode](#1-namenode)
  - [2. DataNode](#2-datanode)
  - [3. Secondary NameNode](#3-secondary-namenode)
  - [4. Block trong HDFS](#4-block-trong-hdfs)
  - [5. Rack Awareness](#5-rack-awareness)
- [Đọc ghi dữ liệu trên HDFS](#đọc-ghi-dữ-liệu-trên-hdfs)
  - [1. Ghi dữ liệu trên HDFS](#1-ghi-dữ-liệu-trên-hdfs)
  - [2. Đọc dữ liệu trong HDFS](#2-đọc-dữ-liệu-trong-hdfs)

## Big Data là gì?
- Big data là một tập dữ liệu lớn không thể xử lý bằng các phương pháp truyền thống.
- 3 đặc tính của Big Data:
  - Volume: Kích thước, độ lớn của dữ liệu.
  - Velocity: Tốc độ xử lý dữ liệu (có dữ liệu rồi, phải xử lý dữ liệu đó thì nó mới có ý nghĩa).
  - Variety: Tính đa dạng của dữ liệu (unstructured, voice, video, ...).

## Hadoop và các thành phần
- Hadoop xuất phát từ Google File System, sau này được phát triển thành hệ thống Hadoop (HDFS).

![Hadoop Image](Pasted%20image%20241028220255.png)

### Tổng quan về kiến trúc hadoop

[Hadoop Ecosystem Overview](https://demanejar.github.io/posts/hadoopo-ecosystem/)

![Hadoop Architecture](Pasted%20image%20241028220512.png)

Apache Hadoop là một framework hỗ trợ lưu trữ và xử lý dữ liệu phân tán trên nhiều máy. Nó được thiết kế để có khả năng scale trên hàng ngàn máy (node).

Thành phần:
- Hadoop Common: Các utils hỗ trợ Hadoop module.
- HDFS: Hệ thống file phân tán.
- Hadoop YARN: Framework dùng cho lập trình job và quản lý tài nguyên hệ thống.
- MapReduce: Hệ thống xử lý dữ liệu của Hadoop.

#### 1. Kiến trúc HDFS
Hadoop Distributed File System (HDFS): Một hệ thống file phân tán được thiết kế để có thể chạy trên các hệ thống phần cứng thông thường.
- Khả năng chịu lỗi cao (fault-tolerant).
- Deploy trên các phần cứng thông thường.
- Khả năng truy cập (high throughput).
- Khả năng scale tốt khi muốn mở rộng hệ thống.

Kiến trúc HDFS:

![HDFS Architecture](Pasted%20image%20241028221349.png)

Gồm 2 thành phần chính là NameNode và DataNode theo cơ chế master-slave. NameNode đóng vai trò là người quản lý toàn bộ hoạt động.

#### 2. Hadoop cluster

![Hadoop Cluster](Pasted%20image%20241028221542.png)

Trong hệ thống có hàng ngàn máy (node), các node sẽ được sắp xếp theo từng rack.

![Rack Awareness](Pasted%20image%20241028221642.png)

#### 3. MapReduce Framework Hadoop
MapReduce là framework dùng để xử lý dữ liệu lớn có thể lên tới hàng petabyte một cách song song trên hàng ngàn node với tốc độ cao. MapReduce gồm 2 pha chính là map và reduce.

![MapReduce](Pasted%20image%20241028221757.png)

## Kiến trúc HDFS

#### 1. NameNode
- Lưu trữ thông tin metadata (fsimage, edit logs).
- Quản lý user truy cập vào hệ thống (gán quyền cho user truy cập vào file).
- Giao tiếp với client cho thao tác đọc ghi dữ liệu.
- Nhận heartbeat của DataNode.
- Xác định location của file, replicated file.

> *Trong kiến trúc Hadoop version 2, Hadoop đảm bảo high availability với một NameNode ở trạng thái StandBy để thay thế khi NameNode Active gặp lỗi.*

#### 2. DataNode
- Cung cấp các block để lưu trữ file.
- Giao tiếp với client cho thao tác đọc ghi.
- Khởi tạo và xóa block data.
- Replicate data trong cụm.
- Giữ liên lạc với NameNode thông qua việc gửi heartbeat theo định kỳ.

#### 3. Secondary NameNode
- Lưu các thông tin trên fsimage của NameNode theo định kỳ.
- Phục vụ cho việc khôi phục lại thông tin metadata trên NameNode khi gặp sự cố.

![Secondary NameNode](Pasted%20image%20241028223417.png)

#### 4. Block trong HDFS
- Hadoop version 1: kích thước của block là 64MB.
- Hadoop version 2: kích thước của block là 128MB và có thể lên tới 256MB.

![HDFS Block Size](Pasted%20image%20241028223953.png)

Các block được lưu trữ trên DataNode và được duplicate.

#### 5. Rack Awareness
- Tăng khả năng trao đổi dữ liệu, dễ dàng cho việc quản lý.
- Không có quá 1 block trên 1 node, không có quá 2 block trên cùng 1 rack.

> *Fault tolerant: Khi Rack-1 lỗi, vẫn còn dữ liệu trên Rack-2.*

![Rack Awareness Diagram](Pasted%20image%20241028224144.png)

> **Kích thước block lớn** phù hợp khi xử lý các tập dữ liệu lớn và giúp giảm tải cho NameNode.
> 
> **Kích thước block nhỏ** thì linh hoạt hơn cho các tác vụ xử lý nhỏ, nhưng cần cân nhắc để tránh tăng quá mức lượng metadata.

## Đọc ghi dữ liệu trên HDFS

#### 1. Ghi dữ liệu trên HDFS
1. Client gửi request ghi file đến cho hệ thống qua các `DistributedFileSystem` APIs.
2. `DistributedFileSystem` gửi yêu cầu RPC tới NameNode.
3. Client ghi file thông qua `FSDataOutputStream`.
4. Dữ liệu sẽ được ghi trên 1 DataNode và replicate thành `n` bảng trên DataNode khác.
5. DataNode gửi lại message `acknowledgement`.
6. Client thực hiện close file vừa ghi.
7. `DistributedFileSystem` báo lại cho NameNode về việc ghi xong dữ liệu.

![Write Data on HDFS](Pasted%20image%20241028225325.png)

#### 2. Đọc dữ liệu trong HDFS
1. Client gửi request đọc file qua `DistributedFileSytem` APIs.
2. `DistributedFileSystem` gửi yêu cầu RPC tới NameNode để lấy location của block.
3. Dữ liệu được đọc trong các DataNode, và chuyển sang block khác khi đọc xong.
4. Nếu block đang đọc bị lỗi, hệ thống chuyển sang DataNode khác gần nhất.
5. Khi đọc xong, client gửi lệnh close file.

![Read Data on HDFS](Pasted%20image%20241028230215.png)

> *Khi DataNode fail trong lúc client ghi dữ liệu, block hiện tại sẽ được ghi mới lại trên DataNode khác, còn block bị ghi một phần sẽ được remove khi DataNode khôi phục lại.*

![Failover Process](Pasted%20image%20241028231246.png)
