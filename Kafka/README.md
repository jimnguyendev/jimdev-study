# KAFKA

## Tại sao lại học kafka

Khi nào nên dùng ?

1. event_bus
2. observer_pattern
3. stream

3 cái này là 1 combo thường đi cùng nhau. Cơ bản nó thay đổi mindset communicate giữa các services. 

Trước thường là pull based, server mở sẵn listen, client cần dữ liệu thì request lên kéo về. Giờ có trend push based, client mới là thằng mở sẵn lắng nghe (observer), server có event thì bắn vào broker (event bus), broker sẽ có những logic riêng dispatch đến các client đang lắng nghe. 

Để làm được việc lắng nghe liên tục thì request dạng truyền thống đóng mở mỗi lần cần dùng như http/1 không hợp lý lắm, nên việc giữ connect luôn live (streaming) là cần thiết, khi các app ready sẽ luôn giữ connect stable để trao đổi data qua lại realtime.

Việc khi nào nên dùng thì tuỳ thuộc vào design của hệ thống & năng lực của team: 

- Có khả năng maintain được broker hay không, redis/kafka/rabbitmq/socket server/etc… không phải ai cũng control được nó hoàn toàn nhất là khi chạm đến những pain point của nó.
- Việc giữ streaming pipeline khác biệt so với với http/1 server truyền thống, nhất là khi tải cao, phải master được các khái niệm mới như connection pool, heart beat, idle, keep alive,...

### Event streaming

- Streaming: a continuous flow of data (event) in real-time

> Là một luồng dữ liệu liên tục theo thời gian thực.

Streaming (cơ chế move data) != Stream Processing (cơ chế xử lý trên luồng dữ liệu)

### Topic và Partition

1. Topic
   - Là một luồng event không có điểm dừng
   - Producer sẽ ghi message vào topic và consumers sẽ lấy mess ra để đọc
   - Một cụm kafka có thể có nhiều topic
2. Partition
  - Một topic là một nhóm các partition
  - Là một tập các record mess đc sắp xếp theo thứ tự và không có điểm dừng
  - Consumer sẽ đọc đúng thứ tự mess trong partition
  - Thứ tự ghi ở đầu producer sẽ được duy trì nếu chỉ ghi vào một đầu partition.
  - Nhiều producer cùng push message vào 1 partition có thể không đảm bảo thứ tự vì network. 

[Topic và Partition docs](https://docs.google.com/presentation/d/1ekuBrktYhX__y3tOg5U-O-VLPR1p3ZYcAILJxqQ5BJQ/edit?pli=1#slide=id.g26a54071665_0_158)

### Offset và Record

- Offset là một id duy nhất của một record trong partition
- Record is aka a message, an event and **immutable**
  
[Offset và Record docs](https://docs.google.com/presentation/d/1ekuBrktYhX__y3tOg5U-O-VLPR1p3ZYcAILJxqQ5BJQ/edit#slide=id.g26a54071665_0_0)

### Producer

- Nếu mà ko khai báo số partition trong record mà key != null, thì key sẽ được hash để tìm ra partition
- Nếu thay đổi số lượng partition thì record có cùng key có thể sẽ không nằm cùng trên một partition

[Producer docs](https://docs.google.com/presentation/d/1ekuBrktYhX__y3tOg5U-O-VLPR1p3ZYcAILJxqQ5BJQ/edit#slide=id.g26a54071665_0_0)

[Producer advanced docs](https://docs.google.com/presentation/d/1R4aSz2sOVY_nzzfx9mCkEzfFWO5wv2wXSV5SSdX5w2Y/edit#slide=id.g2bec0addcfd_0_0)

- ACK trong kafka được hiểu là đã nhận message và đã xử lý lưu trữ thành công ở đầu producer
- Lần đầu gửi message và các lần retry được coi là các lần gửi thử (attempt).
- Khoảng thời gian giữa các lần gửi thử (attempt) mặc định là 100 ms (được cấu hình bởi retries.backoff.ms).
- Nếu lần đầu gửi message lỗi thì producer sẽ đợi 100 ms để retry.
- Để enable.idempotence cần đảm bảo cả 3 điều kiện sau:
acks = all
retries > 0
0 < max.in.flight.requests.per.connection <= 5
- Trong trường hợp retries=INT_MAX, producer có thể sẽ không retries đến giá trị đó. mà mặc định sau khi gọi hàm send() được 2 phút (delivery.timeout.ms), hàm send() gửi message sẽ dừng lại.

> Note: max.in.flight.requests.per.connection trong Kafka là một tham số cấu hình quan trọng của Producer (bộ phận sản xuất) trong Kafka, nó kiểm soát số lượng yêu cầu (request) tối đa có thể được gửi đi mà chưa nhận được phản hồi từ broker (bộ phận môi giới) cho mỗi kết nối.
### Consumer

- Có nhiệm vụ là đọc message từ topic và sử dụng **pull/poll** 
- **1 partition được consume bởi tối đa 1 consumer.**
- **1 consumer có thể consume nhiều partitions.**
  - Case 1: số partitions = số consumers. 1 consumer sẽ consume 1 partition.
  - Case 2: X partitions < Y consumers. Lúc đó sẽ có (Y - X) consumers ở trạng thái idle (rảnh). Tuy nhiên, trong một số trường hợp, người ta có thể tạo một vài idle consumers. Để nếu 1 consomer chết thì sẽ có 1 idle consumer vào thế chỗ luôn. Hạn chế downtime của partition đó. Và một vài idle consumers không hoạt động, có tốn tài nguyên nhưng không đáng kể.
  - Case 3: số partitions > số consumers. Lúc đó sẽ có những consumer consume nhiều hơn 1 partition. Kafka sẽ cố gắng đảm bảo: 1 là không có partition nào được consume, hoặc 2 là tất cả partitions đều được consume.

[Consumer docs](https://docs.google.com/presentation/d/1ekuBrktYhX__y3tOg5U-O-VLPR1p3ZYcAILJxqQ5BJQ/edit#slide=id.g26a54071665_0_0)

[Consumer advanced docs](https://docs.google.com/presentation/d/1Mut3SCKKnse4Fvd64H3iVLltjoiggruNWlpr0AsIXgo/edit#slide=id.g26ab4aed02b_0_0)

> Việc commit offset trước và sau khi xử lý mess là do comsumer quyết định.

- Delivery Guarantee là lời đảm bảo của hệ thống về cách hệ thống truyền tải dữ liệu có ổn định, tin cậy, bị mất mát hay không ?
    - At-Most-Once: tối đa 1 lần
    - At-Least-Once: tối thiểu 1 lần
    - Exactly-Once: bản thân message broker ko thể đảm bảo Exactly-Once cần thêm consumer tham gia để xử lý để đạt được.

### Replication

- Số replica gọi là replication factor sẽ được cấu hình trên topic.
- Record từ leader partition sẽ được replicated một cách bất đồng bộ cho các follower

### Serialization

- Serialization: dùng để convert chuyển đổi object
- Parsing: đầu vào thường là dạng text (string) -> dạng có cấu trúc.
- Marshaling: thường đề cập đến giao tiếp thay vì lưu trữ (RPC)

[Serialization docs](https://docs.google.com/presentation/d/1T2sgz-ROXhPVGgimlkFTStYRTGii4v_bCWslg5LFlls/edit#slide=id.g2c7830e68fb_0_0)


### Demo

[Fraud Detection System
](https://docs.google.com/presentation/d/1fn5PsAc4ZuOjnM4eoNQQhWf9NOKhN7K9v_zMLT_jg78/edit#slide=id.g2c8a8ddc061_0_0)

### Group Membership & Partition Assignment

[docs](https://docs.google.com/presentation/d/1jeP_HmVas5-CrU5D7K3Ac6Orsm7x0ae-ZREM4xGZIK0/edit#slide=id.g2c3c50e36fa_0_1)

### Liveness

- liveness: đảm bảo hệ thống hoạt động xuôn sẻ
- Safety: đảm bảo ko có vấn đề ảnh hưởng xấu đến hệ thống

[docs](https://docs.google.com/presentation/d/1_iZMhTvUIgrkNAJCE0PgAwOdkHfR0Q_HWk6KlkWYmz0/edit)