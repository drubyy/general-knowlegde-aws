# note-aws-developer-associate-level

### SQS
 - Thời gian lưu trữ message có thể setting: 1 - 14 days. Mặc định là 4 ngày
 
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
 
### Elastic load balancer
 - Khi tạo load balancer => cần tạo target group => cần chỉ định target type => Khi tạo xong target group KHÔNG THỂ thay đổi target type
 - #### Target types
   - instance: chỉ định bởi các ID instance
   - ip: Chỉ định bởi các IP (Có thể đặt range dải mạng, etc: 10.0.0.0/8)
   - Lambda: Mục tiêu là 1 hàm lambda

### Auto Scaling
 - Health check + replace unhealth instance
 
### S3:
 - Đối với event notification, nếu không bật chế độ version thì khi 2 request đều ghi vào 1 đối tượng => có thể sẽ chỉ có 1 event notification được trigger
 
### IAM
 - #### IAM access advisor
   - Sử dụng để scan / phân tích permission lần cuối được sử dụng bởi user / role là khi nào, từ đó có thể thấy được các quyền dư thừa => loại bỏ quyền dư thừa không sử dụng
 - #### IAM access analyzer
   - Sử dụng để phân tích các tài nguyên trong Organization hoặc trong tài khoản AWS để phát hiện các rủi ro bảo mật về chia sẻ tài nguyên ngoài ý muốn.
   - VD: 1 bucket S3 hoặc 1 role IAM chia sẻ với 1 thực thể khác bên ngoài => Thực thể đó có thể sử dụng để truy cập vào tài nguyên
 - Có thể sử dụng IAM để làm trình quẩn lý chứng chỉ (SSL/TLS) khi cần sử dụng HTTPS trong region không được ACM (AWS Certificate Manager) hỗ trợ
   - Chỉ sử dụng được đối với chứng chỉ SSL/TLS bên ngoài, không sử dụng được đối với chứng chỉ do ACM cung cấp
   - Không thể quản lý chứng chỉ ở bảng điều khiển
   
### AWS Secret Manager
 - Sử dụng để quản lý các bí mật cần thiết để truy cập các ứng dụng, dịch vụ và tài nguyên AWS, ví dụ như mật khẩu database, API key,...
 - Cho phép xoay vòng
 - Truy xuất dữ liệu trong AWS Secret Manager bằng cách call API
   
### AWS Trusted advisor
 - Là một công cụ trực tuyến cung cấp phương pháp cải thiện in real-time theo best practice của AWS về:
   - cost optimization (tối ưu hóa chi phí)
   - security (bảo mật)
   - fault tolerance (khả năng chịu lỗi)
   - service limits (giới hạn dịch vụ một cách đơn giản nhất)
   - and performance improvement (cải thiện hiệu suất).
   
### AWS Inspector
 - Là một dịch vụ đánh giá bảo mật tự động giúp cải thiện tính bảo mật và tuân thủ của các application được deploy trên AWS
 - Đánh giá về:
   - exposure (mức độ phơi nhiễm)
   - vulnerabilities (lỗ hổng bảo mật)
   - deviations (độ sai lệch so với best practice)
 
### AWS CDK
 - Sử dụng để định nghĩa các resources bằng các ngôn ngữ lập trình quen thuộc như: python, java, js,...
 
### AWS Budget
 - Cần dữ liệu của 5 tuần để có thể dự báo ngân sách
