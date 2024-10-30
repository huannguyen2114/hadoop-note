## Nhân ma trận

Giả sử ta có một ma trận (M) kích thước (p x q), với phần tử ở hàng (i) và cột (j) được ký hiệu là (m\_{ij}), và một ma trận (N) kích thước (q x r) với phần tử ở hàng (j) và cột (k) được ký hiệu là (n\_{jk}). Khi đó, tích của (M) và (N) là ma trận (P = MN) có kích thước (p x r), với phần tử ở hàng (i) và cột (k) được ký hiệu là (p\_{ik}), trong đó:

````latex
P(i, k) = m_{ij} x n_{jk}
````

![Pasted image 20241030123848.png](image/Pasted%20image%2020241030123848.png)

## Data Model

Chúng ta biểu diễn ma trận (M) dưới dạng một quan hệ (M(I, J, V)), với các bộ dữ liệu (i, j, m\_{ij}), và ma trận (N) dưới dạng một quan hệ (N(J, K, W)), với các bộ dữ liệu (j, k, n\_{jk}). Hầu hết các ma trận đều thưa nên có một lượng lớn ô có giá trị bằng 0. Khi biểu diễn ma trận theo dạng này, chúng ta không cần lưu các mục có giá trị bằng 0, giúp tiết kiệm một lượng lớn không gian lưu trữ. 

Dữ liệu đầu vào cho các tệp, chúng ta lưu trữ ma trận (M) và (N) trên HDFS theo định dạng sau:

*M, i, j, m_ij}*

````
 M,0,0,10.0
 M,0,2,9.0
 M,0,3,9.0
 M,1,0,1.0
 M,1,1,3.0
 M,1,2,18.0
 M,1,3,25.2
 ....
````

*N, j, k, n_jk}*

````
 N,0,0,1.0
 N,0,2,3.0
 N,0,4,2.0
 N,1,0,2.0
 N,3,2,-1.0
 N,3,6,4.0
 N,4,6,5.0
 N,4,0,-1.0
 ....
````

## MapReduce

Chúng ta sẽ viết các hàm Map và Reduce để xử lý các tệp đầu vào. Hàm Map sẽ tạo ra các cặp **key, value** từ dữ liệu đầu vào như được mô tả trong Thuật toán 1. Hàm Reduce sử dụng đầu ra của hàm Map, thực hiện các phép tính và tạo ra các cặp **key, value** như được mô tả trong Thuật toán 2. Tất cả các đầu ra sẽ được ghi vào HDFS.

![Pasted image 20241030124555.png](image/Pasted%20image%2020241030124555.png)

## Ví dụ

Giả sử ta có 2 ma trận M(2x3) và N(3x2) như sau:

![Pasted image 20241030124814.png](image/Pasted%20image%2020241030124814.png)

Tích P của MxN sẽ như sau:

![Pasted image 20241030124901.png](image/Pasted%20image%2020241030124901.png)

### Map

Với ma trận M, sau giải thuật 1 (trong Map Function) sẽ tạo ra 1 cặp (key, value) như bên dưới:

![Pasted image 20241030125046.png](image/Pasted%20image%2020241030125046.png)

Với ma trận  N, sau giải thuật 2 (trong Map Function) sẽ tạo ra 1 cặp (key, value) như bên dưới:

![Pasted image 20241030125226.png](image/Pasted%20image%2020241030125226.png)

Sau khi thực hiện thao tác kết hợp (combine operation), tác vụ map sẽ trả về các cặp **key, value** như sau:

![Pasted image 20241030125320.png](image/Pasted%20image%2020241030125320.png)

Lưu ý rằng các mục có cùng **key** sẽ được nhóm trong cùng một chỗ, thao tác này được thực hiện bởi framework. 
Đầu ra này sẽ được lưu trữ trong HDFS và cung cấp cho tác vụ reduce làm đầu vào.

### Reduce

Tác vụ reduce (reduce task) nhận các cặp $key, value$ làm đầu vào và xử lý từng khóa một. Đối với mỗi khóa, nó chia các giá trị thành hai danh sách riêng biệt cho M và N. Đối với khóa (1,1), giá trị là danh sách \[(M,1,1), (M,2,2), (M,3,3), (N,1,a), (N,2,c), (N,3,e)\].

Tác vụ reduce (reduce task) sắp xếp các giá trị bắt đầu bằng M vào một danh sách và các giá trị bắt đầu bằng N vào một danh sách khác như sau:

* **Danh sách M**: Chứa tất cả các giá trị bắt đầu bằng M.
* **Danh sách N**: Chứa tất cả các giá trị bắt đầu bằng N.

### Ví dụ:

Đối với khóa (1,1) với giá trị là danh sách:

* **Giá trị**: \[(M,1,1), (M,2,2), (M,3,3), (N,1,a), (N,2,c), (N,3,e)\]

**Kết quả:**

* **Danh sách M**: \[(M,1,1), (M,2,2), (M,3,3)\]
* **Danh sách N**: \[(N,1,a), (N,2,c), (N,3,e)\]

![Pasted image 20241030125543.png](image/Pasted%20image%2020241030125543.png)

Sau đó, tác vụ giảm (reduce task) tính tổng của tích mijm\_{ij}mij​ và njkn\_{jk}njk​ cho mỗi jjj như sau:

### Công thức:

* Đối với mỗi giá trị trong danh sách M, ký hiệu là mijm\_{ij}mij​:
  * mijm\_{ij}mij​ là giá trị từ danh sách M với chỉ số iii.
* Đối với mỗi giá trị trong danh sách N, ký hiệu là njkn\_{jk}njk​:
  * njkn\_{jk}njk​ là giá trị từ danh sách N với chỉ số jjj và kkk.

### Quy trình:

1. **Khởi tạo biến tổng**: Bắt đầu với tổng bằng 0.
1. **Lặp qua từng chỉ số j**:
   * Lấy giá trị mijm\_{ij}mij​ từ danh sách M.
   * Lấy giá trị njkn\_{jk}njk​ từ danh sách N.
   * Tính tích mij×njkm\_{ij} \times n\_{jk}mij​×njk​ và cộng vào tổng.

### Ví dụ:

Giả sử:

* **Danh sách M**: \[(M,1,1), (M,2,2), (M,3,3)\]
* **Danh sách N**: \[(N,1,a), (N,2,c), (N,3,e)\]

**Kết quả**:

* Tính tổng cho mỗi jjj:
  * Tổng=∑(mij×njk)\text{Tổng} = \sum (m\_{ij} \times n\_{jk})Tổng=∑(mij​×njk​) cho tất cả các jjj có giá trị tương ứng trong danh sách M và N.

**Lưu ý**: Các giá trị từ danh sách M và N cần phải được xác định chính xác (có thể là các số thực hoặc số nguyên) để thực hiện phép nhân và cộng.

![P(1,1) = 1a+2c+3e](https://s0.wp.com/latex.php?latex=P%281%2C1%29+%3D+1a%2B2c%2B3e&bg=f7f7f7&fg=242424&s=0&c=20201002)

Cùng một phép toán sẽ được áp dụng cho tất cả các mục đầu vào của tác vụ reduce (reduce task).

![Pasted image 20241030125717.png](image/Pasted%20image%2020241030125717.png)

Output nhận được:
![Pasted image 20241030125730.png](image/Pasted%20image%2020241030125730.png)

## Code

Phát triển các lớp mapper và reducer dưới dạng Map.java và Reduce.java, cũng như lớp ứng dụng chính gọi là MatrixMultiply.java. Như thấy trong code của `MatrixMultiply.java` dưới đây, trong main, các tham số cấu hình đang được thiết lập cũng như các thư mục đầu vào/đầu ra của tác vụ MapReduce.

![Pasted image 20241030125804.png](image/Pasted%20image%2020241030125804.png)

Class `Mapper` kế thừa class `org.apache.hadoop.mapreduce.Mapper` và thực hiện tác vụ map được mô tả trong Thuật toán 1, đồng thời tạo ra các cặp $(key, value)$ từ các tệp đầu vào như được thể hiện trong code dưới đây:

![Pasted image 20241030125810.png](image/Pasted%20image%2020241030125810.png)

Class `Reducer` kế thừa class `org.apache.hadoop.mapreduce.Reducer` và thực hiện tác vụ reduce được mô tả trong Thuật toán 2, đồng thời tạo ra các cặp $(key, value)$ cho ma trận tích sản phẩm, sau đó ghi output của nó lên HDFS như được thể hiện trong code dưới đây:

![Pasted image 20241030125818.png](image/Pasted%20image%2020241030125818.png)

## Kết quả:

Tính thủ công ta dễ dàng có:
![Pasted image 20241030130352.png](image/Pasted%20image%2020241030130352.png)

Ta có input cho ma trận A:

````shell
A,0,0,1
A,0,1,2
A,0,2,3
A,1,0,4
A,1,1,5
A,1,2,6
````

Ta có input cho ma trận B:

````
B,0,0,10
B,0,1,11
B,1,0,20
B,1,1,22
B,2,0,30
B,2,1,33
````

Đưa file ma trận lên HDFS:

![Pasted image 20241030131129.png](image/Pasted%20image%2020241030131129.png)

Thực hiện chạy chương trình bằng lệnh:

````bash
hadoop jar target/MatrixMultiply.jar com.lendap.hadoop.MatrixMultiply matrix_input matrix_output
````
