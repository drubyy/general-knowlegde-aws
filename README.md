# note-aws-developer-associate-level

### SQS
 - Thời gian lưu trữ message có thể setting: 1 - 14 days. Mặc định là 4 ngày
 - Có thể chỉ định số lượng message tối đa được phép lấy trong 1 lần (max = 10)
 
 - #### messageVisibleTimeout
   Để tránh không cho nhiều consumer xử lý cùng 1 message thì có messageVisibleTimeout, có thể hiểu đây là thời gian tối đa consumer được phép xử lý message (default = 30s, minimum = 0, maximum = 12h)
 - #### delay queue
   Để set thời gian delay cho message enqueue => sử dụng delayQueue (default = 0s, minimum = 0s, maximum = 15 minutes)
 => Sự khác biệt giữa messageVisibleTimeout và delayQueue là:
   - messageVisibleTimeout sẽ bị ẩn đối với các consumer khác MỖI LẦN có 1 consumer nhận message
   - delayQueue sẽ ẩn message đối với TẤT CẢ consumer lần đầu tiên nó được đưa vào queue

 - #### Size message
   - Tối thiểu là 1 byte (1 ký tự)
   - Max size sẽ là 256KB
   - Có thể sử dụng Amazon SQS Extended Client Library for Java để tăng kích thước tối đa lên 2GB
<hr/>
 
### Elastic load balancer
 - Khi tạo load balancer => cần tạo target group => cần chỉ định target type => Khi tạo xong target group KHÔNG THỂ thay đổi target type
 - #### Target types
   - instance: chỉ định bởi các ID instance
   - ip: Chỉ định bởi các IP (Có thể đặt range dải mạng, etc: 10.0.0.0/8)
   - Lambda: Mục tiêu là 1 hàm lambda
 - Sử dụng header X-Forwarded-For để xem IP của client request nếu sử dụng load balancer
<hr/>

### Auto Scaling
 - Health check + thay thế các unhealth instances
 - Không tự động attach volumn nếu ổ đĩa sắp hết dung lượng
<hr/>
 
### S3:
 - Đối với event notification, nếu không bật chế độ version thì khi 2 request đều ghi vào 1 đối tượng => có thể sẽ chỉ có 1 event notification được trigger
<hr/>

### IAM
 - #### IAM access advisor
   - Sử dụng để scan / phân tích permission lần cuối được sử dụng bởi user / role là khi nào, từ đó có thể thấy được các quyền dư thừa => loại bỏ quyền dư thừa không sử dụng
 - #### IAM access analyzer
   - Sử dụng để phân tích các tài nguyên trong Organization hoặc trong tài khoản AWS để phát hiện các rủi ro bảo mật về chia sẻ tài nguyên ngoài ý muốn.
   - VD: 1 bucket S3 hoặc 1 role IAM chia sẻ với 1 thực thể khác bên ngoài => Thực thể đó có thể sử dụng để truy cập vào tài nguyên
 - Có thể sử dụng IAM để làm trình quẩn lý chứng chỉ (SSL/TLS) khi cần sử dụng HTTPS trong region không được ACM (AWS Certificate Manager) hỗ trợ
   - Chỉ sử dụng được đối với chứng chỉ SSL/TLS bên ngoài, không sử dụng được đối với chứng chỉ do ACM cung cấp
   - Không thể quản lý chứng chỉ ở bảng điều khiển
<hr/>

### DynamoDB
 - Sử dụng global table nếu có người dùng phân phối toàn cầu => giảm khoảng cách vật lý giữa client và DynamoDB endpoint => Giảm độ trễ
 - #### Eventually consistent
   - Response có thể sẽ không phải data mới nhất, có thể hiểu rằng response sẽ là data tại lúc call, trong quá trình call, data có được thay đổi cũng sẽ không được trả về data mới nhất => nhanh hơn so với strongly consistent
 - #### Strongly consistent
   - Trả về dữ liệu được cập nhật mới nhất, tuy nhiên sẽ đi kèm với 1 số nhược điểm:
     - Có độ trễ cao hơn so với eventually consistent
     - Không hỗ trợ đối với global secondary indexes (chỉ mục thứ cấp toàn cầu)
     - Sử dụng nhiều read capacity hơn so với eventually consistent
 - DynamoDB mặc định sử dụng Eventually consistent, trừ khi setting khác
 - Các câu lệnh GetItem, Query, Scan cho phép thêm option ConsistentRead = true để sử dụng Strongly Consistent trong quá trình thao tác.
 - Amazon DynamoDB Accelerator (DAX)
   - Là bộ nhớ đệm, có khả năng sử dụng cao, được thiết kế riêng cho DynamoDB
   - Cải thiện performance lên đến 10 lần - từ mili second -> micro second ngay cả khi có hàng triệu request mỗi giây
   - Tính phí theo giờ và các instance DAX không cần cam kết dài hạn
   
### ECS
 - Để config ECS, viết config trong file /etc/ecs/ecs.config
   - ECS_ENABLE_TASK_IAM_ROLE: Sử dụng để kích hoạt IAM role cho container
<hr/>   

### EBS
 - Nếu bật chế độ Encryption by default, từ sau đó trở đi các EBS mới được tạo ra sẽ được mã hóa theo mặc định
   - Mã hóa theo mặc định là theo region, không thể chỉ định đặc biệt EBS nào được mã hóa EBS nào không trong cùng 1 region
<hr/>

### AWS Secret Manager
 - Sử dụng để quản lý các bí mật cần thiết để truy cập các ứng dụng, dịch vụ và tài nguyên AWS, ví dụ như mật khẩu database, API key,...
 - Cho phép xoay vòng
 - Truy xuất dữ liệu trong AWS Secret Manager bằng cách call API
<hr/>

### AWS Trusted advisor
 - Là một công cụ trực tuyến cung cấp phương pháp cải thiện in real-time theo best practice của AWS về:
   - cost optimization (tối ưu hóa chi phí)
   - security (bảo mật)
   - fault tolerance (khả năng chịu lỗi)
   - service limits (giới hạn dịch vụ một cách đơn giản nhất)
   - and performance improvement (cải thiện hiệu suất).
<hr/>

### AWS Inspector
 - Là một dịch vụ đánh giá bảo mật tự động giúp cải thiện tính bảo mật và tuân thủ của các application được deploy trên AWS
 - Đánh giá về:
   - exposure (mức độ phơi nhiễm)
   - vulnerabilities (lỗ hổng bảo mật)
   - deviations (độ sai lệch so với best practice)
<hr/>

### AWS CDK
 - Sử dụng để định nghĩa các resources bằng các ngôn ngữ lập trình quen thuộc như: python, java, js,...
<hr/>

### AWS Budget
 - Cần dữ liệu của 5 tuần để có thể dự báo ngân sách
<hr/>
