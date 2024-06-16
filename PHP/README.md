# Performance with laravel

## Measuring performance

1. Throughput (biểu thị lượng công việc mà hệ thống có thể xử lý trong một khoảng thời gian nhất định.)

- Requests per second (RPS): Số lượng yêu cầu mà API có thể xử lý trong một giây.
- Transactions per second (TPS): Số lượng giao dịch (thường phức tạp hơn một yêu cầu đơn lẻ) mà API có thể xử lý trong một giây.
- Concurrent users: Số lượng người dùng có thể truy cập và sử dụng API đồng thời.

2. Response time (Thời gian phản hồi)

- Average response time: Thời gian trung bình để API xử lý và trả về kết quả cho một yêu cầu.
- Peak response time: Thời gian phản hồi lâu nhất trong một khoảng thời gian nhất định.
- Error rate: Tỷ lệ các yêu cầu bị lỗi so với tổng số yêu cầu.
- Size: tổng kích thước của HTTP response.

3. Latency (Độ trễ)

- Network latency: Thời gian để yêu cầu đi từ client đến server và phản hồi quay
- Processing latency: Thời gian để server xử lý yêu cầu và tạo phản hồi.

4. Resource utilization

- CPU usage: Phần trăm CPU được sử dụng bởi API.
- Memory usage: Lượng bộ nhớ được sử dụng bởi API.
- Disk I/O: Số lượng hoạt động đọc/ghi trên đĩa của API.
- Server uptime là khoảng thời gian mà một máy chủ (server) hoạt động liên tục và ổn định, có thể truy cập và phản hồi các yêu cầu từ người dùng hoặc các hệ thống khác.

**Các yếu tố cần đánh giá khi một api chậm**

- Number of database queries
- The execution time of database queries
- Which function takes a long time to finish its job?
- Which function uses more memory than it's supposed to? (chức năng nào sử dụng nhiều bộ nhớ hơn mức cần thiết)
- What parts of the system can be async? (những thành phần nào trong hệ thống có thể chạy không đồng bộ)
- and so on

### Phân biệt stateless và stateful

**Stateless**

- Đặc điểm:
  - Không lưu trữ thông tin về trạng thái của client trên server.
  - Mỗi yêu cầu được xử lý độc lập, không phụ thuộc vào các yêu cầu trước đó.
  - Server không cần biết lịch sử tương tác của client.

- Ưu điểm:
  - Đơn giản và dễ mở rộng: Vì không cần quản lý trạng thái, server dễ dàng xử lý nhiều yêu cầu đồng thời và có thể mở rộng dễ dàng hơn.
  - Độ tin cậy cao: Nếu một server gặp sự cố, yêu cầu có thể được chuyển hướng đến một server khác mà không ảnh hưởng đến trạng thái của client.

- Nhược điểm:
  - Không thể duy trì thông tin giữa các yêu cầu: Điều này có thể gây khó khăn cho việc thực hiện các tính năng như xác thực người dùng, giỏ hàng hoặc theo dõi phiên làm việc.
- Ví dụ:
  - Giao thức HTTP (Hypertext Transfer Protocol)
  - Các API RESTful (Representational State Transfer)
  - DNS (Domain Name System)

**Stateful:**

- Đặc điểm:
  - Lưu trữ thông tin về trạng thái của client trên server.
  - Các yêu cầu được xử lý dựa trên trạng thái hiện tại của client.
  - Server cần biết lịch sử tương tác của client để xử lý yêu cầu một cách chính xác.

- Ưu điểm:
  - Duy trì thông tin giữa các yêu cầu: Điều này cho phép thực hiện các tính năng phức tạp như xác thực người dùng, giỏ hàng, trò chuyện trực tuyến và các ứng dụng theo thời gian thực.
  - Cá nhân hóa trải nghiệm người dùng: Bằng cách theo dõi trạng thái, server có thể cung cấp nội dung và dịch vụ phù hợp với từng người dùng.

- Nhược điểm:
  - Phức tạp và khó mở rộng: Việc quản lý trạng thái đòi hỏi nhiều tài nguyên và có thể gây khó khăn cho việc mở rộng hệ thống.
  - Độ tin cậy thấp hơn: Nếu một server gặp sự cố, trạng thái của client có thể bị mất.

  ## Async workflows

  