# note-aws-associate-level

SQS:
 - Thời gian lưu trữ message có thể setting: 1 - 14 days. Mặc định là 4 ngày
 - messageVisibleTimeout: Để tránh không cho nhiều consumer xử lý cùng 1 message thì có messageVisibleTimeout, có thể hiểu đây là thời gian tối đa consumer được phép xử lý message (mặc định là 30s, minimum = 0, maximum = 12h)

Auto Scaling:
 - Health check + replace unhealth instance
 
S3:
 - Đối với event notification, nếu không bật chế độ version thì khi 2 request đều ghi vào 1 đối tượng => có thể sẽ chỉ có 1 event notification được trigger
