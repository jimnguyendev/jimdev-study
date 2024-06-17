# Redis

## REDIS và 3 sự cố phổ biến khi sử dụng

### Cache avalanche

#### Nguyên nhân

Cache Avalanche là một vấn đề xảy ra khi một lượng lớn dữ liệu trong cache hết hạn đồng thời, dẫn đến việc tất cả các yêu cầu đều đổ dồn về cơ sở dữ liệu gốc, gây quá tải và ảnh hưởng đến hiệu suất hệ thống.

**Giải pháp**

**1. Sử Dụng Khóa hoặc Hàng Đợi**

```java
// Ví dụ mã nguồn Java cho việc sử dụng khóa
public Users getByUserId(Long id) {
    // Tạo khóa cache dựa trên id của người dùng
    String key = this.getClass().getName() + "-" + Thread.currentThread().getStackTrace()[1].getMethodName() + "-id" + id;

    // Kiểm tra dữ liệu người dùng có sẵn trong cache hay không
    String userJson = redisService.getString(key);
    if (!StringUtils.isEmpty(userJson)) {
        // Nếu dữ liệu tồn tại trong cache, chuyển đổi từ JSON sang đối tượng Users và trả về
        Users users = JSONObject.parseObject(userJson, Users.class);
        return users;
    }

    Users user = null;
    try {
        // Mở khóa để tránh xung đột khi truy vấn vào cơ sở dữ liệu
        lock.lock();

        // Thực hiện truy vấn vào cơ sở dữ liệu để lấy thông tin người dùng
        user = userMapper.getUser(id);

        // Lưu thông tin người dùng vào cache
        redisService.setString(key, JSONObject.toJSONString(user));
    } catch (Exception e) {
        // Xử lý ngoại lệ nếu có
    } finally {
        // Giải phóng khóa sau khi hoàn thành truy vấn
        lock.unlock();
    }

    // Trả về thông tin người dùng
    return user;
}

```

**Lưu ý:** Việc sử dụng khóa hàng đợi chỉ nhằm giảm áp lực lên cơ sở dữ liệu và không cải thiện `throughput` của hệ thống. Trong trường hợp có nhiều yêu cầu cùng thời điểm, quá trình tái tạo bộ đệm có thể khiến khóa bị chặn, gây ra thời gian chờ cho người dùng.

**2. Cung Cấp Thời Gian Hết Hạn Ngẫu Nhiên cho Các Giá Trị**

```php
use Illuminate\Support\Carbon;
use Illuminate\Support\Facades\Cache;

function storeRandomExpiration($key, $value, $minMinutes, $maxMinutes)
{
    $ttl = rand($minMinutes, $maxMinutes); // Chọn ngẫu nhiên TTL trong khoảng minMinutes và maxMinutes
    Cache::put($key, $value, Carbon::now()->addMinutes($ttl));
}

// Sử dụng
storeRandomExpiration('product_123', $productData, 10, 30); // TTL ngẫu nhiên từ 10 đến 30 phút

```

**Ưu điểm**

- Tránh Cache Stampede: Các giá trị cache sẽ không hết hạn đồng thời, giảm tải cho hệ thống khi nhiều yêu cầu cố gắng cập nhật lại cache cùng lúc.
- Linh hoạt: Bạn có thể dễ dàng điều chỉnh khoảng thời gian hết hạn bằng cách thay đổi các tham số $minMinutes và $maxMinutes.

**3. Xây Dựng Kiến Trúc Bộ Đệm Đa Cấp**
Dưới đây là một ví dụ về cách sử dụng nhiều tầng cache trong Laravel, kết hợp cả cache trong bộ nhớ (in-memory cache) và cache trên đĩa (file-based cache):

```php
use Illuminate\Support\Facades\Cache;

function getCachedData($key) {
    // Kiểm tra trong bộ nhớ cache trước
    if (Cache::has($key)) {
        return Cache::get($key);
    }

    // Nếu không có trong bộ nhớ cache, kiểm tra trong file cache
    $value = Cache::store('file')->get($key);
    if ($value !== null) {
        // Lưu lại vào bộ nhớ cache để lần sau truy cập nhanh hơn
        Cache::put($key, $value, now()->addMinutes(5));
        return $value;
    }

    // Nếu không có trong cả hai cache, truy xuất dữ liệu từ nguồn gốc (ví dụ: database)
    $value = getDataFromDatabase($key);

    // Lưu vào cả hai cache
    Cache::store('file')->put($key, $value, now()->addHours(1)); // Lưu vào file cache với TTL dài hơn
    Cache::put($key, $value, now()->addMinutes(5)); // Lưu vào bộ nhớ cache với TTL ngắn hơn

    return $value;
}
```

### 2. Thâm Nhập Bộ Đệm (Cache Penetration)

Khi một key không tồn tại trong bộ đệm, nhưng Redis không lưu giá trị null vào cache, điều này khiến mọi yêu cầu truy cập cơ sở dữ liệu.

**Giải pháp**

```java
public String getByUsers2(Long id) {
    // Khởi tạo key để lưu vào Redis
    String key = this.getClass().getName() + Thread.currentThread().getStackTrace()[1].getMethodName() + "-id:" + id;

    // Kiểm tra xem tên người dùng có tồn tại trong Redis không
    String userName = redisService.getString(key);

    // Nếu không tìm thấy trong Redis
    if (StringUtils.isEmpty(userName)) {
        // In ra thông báo bắt đầu gửi yêu cầu tới cơ sở dữ liệu
        System.out.println("##### Bắt đầu gửi yêu cầu tới cơ sở dữ liệu ******");

        // Lấy thông tin người dùng từ cơ sở dữ liệu
        Users user = userMapper.getUser(id);

        // Khởi tạo giá trị để lưu vào Redis
        String value = null;

        // Kiểm tra xem người dùng có tồn tại không
        if (user != null) {
            // Nếu tồn tại, lấy tên của người dùng
            value = user.getName();
        }

        // Lưu giá trị vào Redis
        redisService.setString(key, value);

        // Trả về tên người dùng
        return value;
    } else {
        // Nếu tìm thấy trong Redis, trả về giá trị đó
        return userName;
    }
}

```

### Cache Breakdown

Xảy ra khi một mục trong cache hết hạn hoặc bị xóa và đồng thời có nhiều yêu cầu truy cập vào mục đó. Do không tìm thấy dữ liệu trong cache, tất cả các yêu cầu đều phải truy cập trực tiếp vào cơ sở dữ liệu gốc, gây áp lực lớn và làm giảm hiệu suất hệ thống.

### Hệ thống sập đột ngột

Khi Redis đột ngột sập trong quá trình tính toán stock của đơn hàng, việc xử lý tình huống này cần sự kết hợp giữa các biện pháp phòng ngừa và khôi phục để đảm bảo tính nhất quán dữ liệu và trải nghiệm người dùng.

**Các biện pháp phòng ngừa:**

- **Sử dụng Redis Cluster:** Cấu hình Redis Cluster để đảm bảo tính sẵn sàng cao (high availability) và khả năng chịu lỗi (fault tolerance). Khi một node Redis gặp sự cố, các node khác vẫn có thể tiếp tục hoạt động.
- **Sao lưu dữ liệu thường xuyên:** Thực hiện sao lưu dữ liệu Redis định kỳ để có thể khôi phục lại dữ liệu trong trường hợp xảy ra sự cố.
- **Sử dụng cơ chế lưu trữ dữ liệu bền vững:** Kích hoạt AOF (Append Only File) hoặc RDB (Redis Database) persistence để đảm bảo dữ liệu không bị mất khi Redis khởi động lại.
- **Giám sát chặt chẽ:** Sử dụng các công cụ giám sát để theo dõi hoạt động của Redis và phát hiện sớm các dấu hiệu bất thường.
- **Xây dựng kịch bản xử lý sự cố:** Chuẩn bị sẵn các kịch bản xử lý sự cố Redis để có thể phản ứng nhanh chóng và hiệu quả khi sự cố xảy ra.

### Redis

- Là một cơ sở liệu key - value in memory, được sử dụng rộng rãi theo nhiều hướng.

-> Mục đích: đảm bảo hiệu xuất cao và tính ổn định của hệ thống

#### String

Lưu trữ tối đa 512M -> triển khai SDS

- embstring (<= 44 bytes)
- raw (> 44 bytes)
- int (integer)

#### Hash (thuật toán hash table)

Redis Hashes là một kiểu dữ liệu trong Redis cho phép lưu trữ nhiều cặp trường (field) và giá trị (value) trong một key duy nhất. Nó hoạt động tương tự như một từ điển (dictionary) hoặc một bảng băm (hash table) trong các ngôn ngữ lập trình khác


#### List (link list)


