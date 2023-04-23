# General note knowlegde

### Auto Scaling
 - AWS Auto Scaling giám sát các ứng dụng và tự động điều chỉnh số lượng instances để duy trì hiệu năng ổn định, tối ưu hoá chi phí vì sẽ chỉ trả phí theo on-demand
 - Health check + thay thế các unhealth instances
 - Nếu EC2 sử dụng EBS, auto scaling sẽ không attach được volumn EBS nếu ổ đĩa sắp hết dung lượng
 - Không tính phí
 - Khi sử dụng chung với ELB, cần đổi health check type từ EC2 -> ELB nếu không sẽ không tự replace unhealth instances
 - Chỉ sử dụng trên 1 region (bao gồm tất cả AZ), không thể sử dụng đối với multi region
 - Khi scaling in (giảm size) sẽ ưu tiên terminate những instances có launch configuration sớm nhất
 - Update launch configuration
   - config mới sẽ chỉ áp dụng cho instances được tạo mới sau đó => instances cũ sẽ không được áp dụng
   => Khi update launch configuration và cần đồng bộ lại launch configuration có thể làm theo cách sau:
      Update launch configuration => Tăng số lượng instance lên gấp đôi => VD hiện tại đang 4 instances => mở thêm 4 instances nữa
      => Sau đó đưa số lượng instances về lại 4 => Auto Scaling thực hiện scaling in => ưu tiên remove 4 instances cũ nhất => launch configuration sẽ mới nhất
   
   <img width="640" alt="Screen Shot 2023-03-05 at 11 05 10" src="https://user-images.githubusercontent.com/57032236/222941094-59da70d8-11eb-48a1-969f-df4310d7b791.png">
   
   <img width="645" alt="Screen Shot 2023-03-05 at 11 14 22" src="https://user-images.githubusercontent.com/57032236/222941340-feaa7424-4d74-4e7e-956c-dd7a4922a1b9.png">
<hr/>

### API Gateway
 - Để custom response trả về => sử dụng API Gateway mapping templates
 - Có thể sử dụng Lambda authorizer để uỷ quyền xác thực client call API Gateway, khi vượt qua xác thực Lambda authorizer client sẽ được cấp 1 token để thao tác với AWS resource sau đó
   ![image](https://user-images.githubusercontent.com/57032236/184393378-58ef62ce-ef23-456f-a2e6-3015906977c0.png)

<hr/>

### Beanstalk
 #### Hook
 
 ```
 # Commands
 
 # https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html#linux-commands 
 # You can use the commands key to execute commands on the EC2 instance. The commands run before the application and web server are set up and the application version file is extracted.
 commands:
   create_hello_world_file:
     command: touch hello-world.txt
     cwd: /home/ec2-user

 # Container Commands
 # https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/customize-containers-ec2.html#linux-container-commands

 # You can use the container_commands key to execute commands that affect your application source code. Container commands run after the application and web server have been set up and the application version archive has been extracted, but before the application version is deployed. 
 container_commands:
   modify_index_html:
     command: 'echo " - modified content" >> index.html'

   database_migration:
     command: 'echo "do a database migration"'
     # You can use leader_only to only run the command on a single instance
     leader_only: true
 ```
 - command: Chạy trước khi application được giải nén và web server được setup
 - container_commands: Chạy sau khi application được giải nén và web server được setup, tuy nhiên chạy trước khi app được deploy
   - NOTE:
     - Có thể sử dụng "leader_only" để chạy duy nhất 1 instance, ví dụ như case có 10 instances và cần thực hiện migrate database, việc thực hiện migrate cho DB chỉ cần chạy 1 lần duy nhất, không cần thiết chạy cả 10 lần => Sử dụng leader_only (chỉ có thể sử dụng với container_commands)
 
 #### Update Stragies
  - All at once (downtime)
  - Rolling
  - Rolling with additional batches (Giống rolling tuy nhiên khi instance thực hiện update sẽ có 1 instance mới thay thế vị trí đó => không bị giảm số lượng instances khi thực hiện update)
  - Immutable: Tạo mới tất cả instances sang một ASG mới => thực hiện update cho các instances mới này, khi thành công các instances sẽ thực hiện swap lại tất cả instances

 #### NOTE
  - Có thể có tối đa 1000 versions application, có thể sử dụng life cycle để thực hiện quản lý vòng đời của version
<hr/>

### Budget
 - Cần dữ liệu của 5 tuần để có thể dự báo ngân sách
<hr/>

### CDK
 - Sử dụng để định nghĩa các resources bằng các ngôn ngữ lập trình quen thuộc như: python, java, js,...
<hr/>

### CloudFormation
 - Viết file quản lý cơ sở hạ tầng, cung cấp và mô tả các tài nguyên AWS cũng như bên thứ 3. VD: Cung cấp VPC, subnet, EC2, ECS, CI/CD,...
   ```
   {
     "AWSTemplateFormatVersion" : "2010-09-09",
     "Description" : "A sample template",
     "Resources" : {
       "MyEC2Instance" : {
         "Type" : "AWS::EC2::Instance",
         "Properties" : {
           "ImageId" : "ami-2f726546",
           "InstanceType" : "t1.micro",
           "KeyName" : "testkey",
           "BlockDeviceMappings" : [
             {
               "DeviceName" : "/dev/sdm",
               "Ebs" : {
                 "VolumeType" : "io1",
                 "Iops" : "200",
                 "DeleteOnTermination" : "false",
                 "VolumeSize" : "20"
               }
             }
           ]
         }
       }
     }
   }

   ```
   Nhìn vào template trên sẽ hiểu được chúng ta định nghĩa ra 1 instance EC2 với AMI id là "ami-2f726546", instance type là "t1.micro", key pair là "testkey" và một EBS có type là "iop1",...
   ![image](https://user-images.githubusercontent.com/57032236/180631172-4de1791b-2f3b-4165-af90-8831378b90da.png)
   
 #### Stack
 - Là đơn vị dùng để gọi 1 nhóm các tài nguyên được định nghĩa bởi 1 template, chẳng hạn như khi có 1 file cloudFormation bao gồm các resources: ASG, ELB, RDS,...tạo các resources trên bằng AWS cloudFormation sẽ tương đương với 1 stack
 - Sử dụng output Export xuất giá trị đầu ra để có thể tái sử dụng ở 1 stack khác
 #### Function:
   ##### !GetAtt/Fn::GetAtt => Trả về giá trị của 1 attribute từ một tài nguyên trong template
     ```
     Resources:
      myELB:
        Type: AWS::ElasticLoadBalancing::LoadBalancer
        Properties:
          AvailabilityZones:
            - eu-west-1a
          Listeners:
            - LoadBalancerPort: '80'
              InstancePort: '80'
              Protocol: HTTP
      myELBIngressGroup:
        Type: AWS::EC2::SecurityGroup
        Properties:
          GroupDescription: ELB ingress group
          SecurityGroupIngress:
            - IpProtocol: tcp
              FromPort: 80
              ToPort: 80
              SourceSecurityGroupOwnerId: !GetAtt myELB.SourceSecurityGroup.OwnerAlias
              SourceSecurityGroupName: !GetAtt myELB.SourceSecurityGroup.GroupName
     ```
   ##### !Sub/Fn::Sub => Thay thế các biến trong chuỗi đầu vào bằng các giá trị chỉ định
     ```
     Name: !Sub 
      - 'www.${Domain}'
      - Domain: !Ref RootDomainName


     // OR this
     
     Name: Fn::Sub: "Hello ${AWS::Region}"
     ```
   ##### !Ref => Trả về giá trị của tham số hoặc tham chiếu đến tài nguyên được chỉ định
     - Sử dụng với parameters => Trả về gía trị của parameter
     - Sử dụng với resouces => Trả về physical ID của resouce tương ứng
     ```
     Resources:
      MyInstance:
        Type: AWS::EC2::Instance
        Properties:
          AvailabilityZone: us-east-1a
          ImageId: ami-009d6802948d06e52
          InstanceType: t2.micro
          SecurityGroups:
            - !Ref SSHSecurityGroup
            - !Ref ServerSecurityGroup
     ```
   ##### !FindInMap/Fn::FindInMap trả về giá trị tương ứng với các key (mapping là tập hợp các biến tĩnh trong template)
     ```
     Mappings: 
      RegionMap: 
        us-east-1: 
          HVM64: "ami-0ff8a91507f77f867"
          HVMG2: "ami-0a584ac55a7631c0c"
        us-west-1: 
          HVM64: "ami-0bdb828fd58c52235"
          HVMG2: "ami-066ee5fd4a9ef77f1"
     Resources: 
      myEC2Instance: 
        Type: "AWS::EC2::Instance"
        Properties: 
          ImageId: !FindInMap
            - RegionMap
            - !Ref 'AWS::Region'
            - HVM64
          InstanceType: m1.small
     ```
   ##### !ImportValue/Fn::ImportValue => Để import value từ stack khác vào
     ```
     Resources:
      myBucketResource:
        Type: AWS::S3::Bucket

      LambdaUsedToCleanUp:
        Type: Custom::cleanupbucket
        Properties:
          ServiceToken: !ImportValue EmptyS3BucketLambda
          BucketName: !Ref myBucketResource
     ```
   ##### !Join/Fn::Join => Để join 2 giá trị lại với nhau (như join string trong ruby)
     ```
     !Join [ ":", [ a, b, c ] ]
     
     // This will return "a:b:c"
     ```
   ##### !Base64/Fn::Base64 => trả về biểu diễn Base64 của chuỗi đầu vào. Hàm này thường được sử dụng để truyền dữ liệu đã mã hóa sang các phiên bản Amazon EC2 bằng thuộc tính UserData.
     ```
     Resources:
      MyInstance:
        Type: AWS::EC2::Instance
        Properties:
          AvailabilityZone: us-east-1a
          ImageId: ami-009d6802948d06e52
          InstanceType: t2.micro
          KeyName: !Ref SSHKey
          SecurityGroups:
            - !Ref SSHSecurityGroup
          # we install our web server with user data
          UserData: 
            Fn::Base64: |
              #!/bin/bash -xe
              yum update -y
              yum install -y httpd
              systemctl start httpd
              systemctl enable httpd
              echo "Hello World from user data" > /var/www/html/index.html
     ```
     NOTE good to know: UserData log sẽ được ghi vào /var/log/cloud-init-output.log
   ##### Conditions:
     - !Equals/Fn::Equals
     - !And/Fn::And
     - !If/Fn::If
     - !Not/Fn::Not
     - !Or/Fn::Or
   - ...
 #### Change sets & Drift detection
   - Change sets: Là tập hợp những thay đổi sắp tới SẮP được áp dụng cho template, nếu thấy được rồi thì có thể tiếp tục execute changes để thực hiện (có thể hình dung như git, trước khi commit sẽ được xem các file changed)
   - Drift detection: Là những thay đổi ĐÃ được thực hiện một cách THỦ CÔNG đối với cơ sở hạ tầng (Có thể hiểu là những thay đổi thủ công mà không thông qua re-build cloudFormation). Mục đích là để kiểm soát những thay đổi ngoài ý muốn, nhưng chỉ để xem sự thay đổi của resource, không thể force rollback về hay thay đổi bất cứ resource nào với drift detection
   
 #### Retaining data on delete
 Có thể set DeletionPolicy cho bất cứ resouce nào trong stack để kiểm soát việc gì sẽ xảy ra khi root stack bị xóa
   - Retain: Giữ lại resource, áp dụng với bất cứ resouce/nested stack nào
   - Snapshot: Thực hiện việc snapshot lại resource. Chỉ áp dụng với:
     - EBS volume, ElasticCache Cluster, ElasticCache ReplicationGroup
     - RDS DBInstance, RDS DBCluster, Redshift Cluster
   - Delete (setting mặc định nếu không chỉ định DeletionPolicy cụ thể cho resource)
     - Tuy nhiên đối với resouce là AWS::RDS::DBCluster thì default DeletionPolicy sẽ là Snapshot
     - Đối với resouce là S3 bucket thì default vẫn là delete tuy nhiên cần lưu ý là bắt buộc phải thực hiện empty bucket trước khi thực hiện xóa => CloudFormation sẽ không tự thực hiện empty bucket
    
 #### Terminaltion Protection on Stack
 Có thể setting "Enable termination protection" với giá trị là Enabled để chống việc xóa stack, chỉ có thể xóa stack khi disable "Enable termination protection"
 
 ![image](https://user-images.githubusercontent.com/57032236/223033459-da57166a-2970-40a9-b6ba-6f704ca2606d.png)
 
 #### Custom resource
  - Có thể định nghĩa custom resource, ý tưởng là sử dụng lambda để định nghĩa resouce đó
  - có thể áp dụng để giải quyết một số case như sau:
  - Định nghĩa 1 resource AWS mới chưa được CloudFormation cover
  - Định nghĩa tài nguyên On-Premise
  - Empty bucket S3 trước khi thực hiện xóa stack
  - Kéo AMI id
  - ...

 #### Stack policy
  - Có thể set stack policy để control hành động đối với stack, ví dụ
      ```
      {
          "Statement": [
              {
                  "Effect": "Allow",
                  "Action": "Update:*",
                  "Principal": "*",
                  "Resource": "*"
              },
              {
                  "Effect": "Deny",
                  "Action": "Update:*",
                  "Principal": "*",
                  "Resource": "LogicalResourceId/CriticalSecurityGroup"
              },
              {
                  "Effect" : "Deny",
                  "Action" : "Update:*",
                  "Principal": "*",
                  "Resource" : "*",
                  "Condition" : {
                    "StringEquals" : {
                      "ResourceType" : ["AWS::RDS::DBInstance"]
                    }
                  }
              }
          ]
      }
      ```
      => Để có thể thực hiện update resource "CriticalSecurityGroup" thì có 2 cách:
         - Sử dụng lệnh aws cloudformation set-stack-policy với --stack-policy-body để nhập chính sách muốn thay đổi HOẶC --stack-policy-url để chỉ định đường dẫn đến tệp policy => Sẽ sửa stack policy vĩnh viễn.
         - Sửa stack policy trong quá trình update stack resources => Chỉ sửa tạm thời tại thời điểm update resources => Sau khi xong xem lại stack policy vẫn sẽ như cũ

 #### CFN init
 - Có thể sử dụn cfn-init để thiết lập UserData theo một cách khác, ý tưởng là đưa các câu lệnh UserData vào metadata của resource => UserData sẽ tham chiếu đến đó

   ```
   Resources:
    MyInstance:
      Type: AWS::EC2::Instance
      Properties:
        AvailabilityZone: us-east-1a
        ImageId: ami-009d6802948d06e52
        InstanceType: t2.micro
        KeyName: !Ref SSHKey
        SecurityGroups:
          - !Ref SSHSecurityGroup
        # we install our web server with user data
        UserData: 
          Fn::Base64:
            !Sub |
              #!/bin/bash -xe
              # Get the latest CloudFormation package
              yum update -y aws-cfn-bootstrap
              # Start cfn-init
              /opt/aws/bin/cfn-init -s ${AWS::StackId} -r MyInstance --region ${AWS::Region} || error_exit 'Failed to run cfn-init'
      Metadata:
        Comment: Install a simple Apache HTTP page
        AWS::CloudFormation::Init:
          config:
            packages:
              yum:
                httpd: []
            files:
              "/var/www/html/index.html":
                content: |
                  <h1>Hello World from EC2 instance!</h1>
                  <p>This was created using cfn-init</p>
                mode: '000644'
            commands:
              hello:
                command: "echo 'hello world'"
            services:
              sysvinit:
                httpd:
                  enabled: 'true'
                  ensureRunning: 'true'
                  
   // Cần lưu ý rằng các câu lệnh UserData sẽ không ghi log theo như cách sử dụng trực tiếp UserData bằng hàm Fn::Base64 như ví dụ trên => xem log /var/log/cloud-init-output.log sẽ không ghi log các câu lệnh đó => Để xem log cfn-init thì sẽ xem ở /var/log/cfn-init.log (Tổng quan) hoặc /var/log/cfn-init-cmd.log (Chi tiết)
   ```
   
 #### CFN-signal
  - cfn-signal: Sử dụng để thông báo tín hiệu dạng như waitting condition

  ```
  Resources:
   MyInstance:
     Type: AWS::EC2::Instance
     Properties:
       AvailabilityZone: us-east-1a
       ImageId: ami-009d6802948d06e52
       InstanceType: t2.micro
       KeyName: !Ref SSHKey
       SecurityGroups:
         - !Ref SSHSecurityGroup
       # we install our web server with user data
       UserData: 
         Fn::Base64:
           !Sub |
             #!/bin/bash -xe
             # Get the latest CloudFormation package
             yum update -y aws-cfn-bootstrap
             # Start cfn-init
             /opt/aws/bin/cfn-init -s ${AWS::StackId} -r MyInstance --region ${AWS::Region}
             # Start cfn-signal to the wait condition
             /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackId} --resource SampleWaitCondition --region ${AWS::Region}
     Metadata:
       Comment: Install a simple Apache HTTP page
       AWS::CloudFormation::Init:
         config:
           packages:
             yum:
               httpd: []
           files:
             "/var/www/html/index.html":
               content: |
                 <h1>Hello World from EC2 instance!</h1>
                 <p>This was created using cfn-init</p>
               mode: '000644'
           commands:
             hello:
               command: "echo 'hello world'"
           services:
             sysvinit:
               httpd:
                 enabled: 'true'
                 ensureRunning: 'true'

   SampleWaitCondition:
     CreationPolicy:
       ResourceSignal:
         Timeout: PT2M
         Count: 1
     Type: AWS::CloudFormation::WaitCondition

     // Có thể thấy rằng EC2 chạy script UserData => chạy cfn-init => Chờ 1 instance EC2 trong vòng 2 phút (PT2M) thông báo tín hiệu tốt để tiếp tục tạo stack. Chỉ khi có signal tốt thì stack mới tiếp tục việc tạo còn không thì sẽ có trạng thái failure. Nhưng quan trọng là sẽ chỉ có ý nghĩa nếu đi kèm config enable rollback on failure => Có thể tưởng tượng rollback on failure giống như transaction => raise error thì sẽ rollback tất, còn nếu disable rollback on failure thì sẽ không rollback gì

     // Wait condition có thể sẽ không nhận được tín hiệu từ EC2 instance nên cần đảm bảo các điều kiện sau:
     // - Đảm bảo rằng AMI EC2 đang sử dụng đã được cài CloudFormation helper scripts, nếu chưa cài thì cần download
     // - Đảm bảo rằng câu lệnh cfn-init & cfn-signal thành công, nếu có lỗi thì có thể debug bằng cách xem log /var/log/cloud-init.log hoặc /var/log/cfn-init.log. Nhưng muốn giữ lại log và xem log để debug thì cần lưu ý rằng disabled rollback on failure nếu không thì CloudFormation sẽ xoá instance đó khi stack create fail
     // Điều quan trọng nữa là cần đảm bảo instance có kết nối internet. Nếu instance nằm trong private subnet thì cần sử dụng NAT gateway, nếu public thì có thể sử dụng luôn Internet gateway
  ```
  
 #### CFN hub:
   - WHAT: CFN init sẽ chỉ được chạy lần đâù khi tạo resource (CFN init sẽ lấy thông tin từ metadata để chạy), khi update metadata của resource thì stack CloudFormation sẽ không replace resource đó => CFN-hub sử dụng để thiết lập một schedule phát hiện sự thay đổi của metadata của resource, kết hợp với cfn hook để chỉ định làm 1 việc gì đó theo nhu cầu
   - VD:
   ```
    - 1 Stack có các resources:
      - 1 instance EC2
      - 1 parameter có tên là messageParam dạng String (ví dụ ban đầu truyền vào là 'hello world')
      - Sử dụng CFN-init để cài webserver nginx, sau khi cài nginx xong thì sẽ sửa file /var/www/index.html thành nội dung của messageParams

   - Sau khi create resource xong thì làm các step:
     - Mở web lên và quan sát (Khi này mở web lên sẽ có nội dung là 'hello world')
     - Update stack => truyền lại messageParam vào stack có nội dung là "hello world edited"
     - Mở lại web lên và quan sát vẫn sẽ thấy web có nội dung là "hello world"

   => Khi này cần đến CFN-hub bằng cách
     - Thiết lập cfn-hub.conf
     - Sau mỗi N (minutes) thì CFN-hub sẽ thực hiện check sự thay đổi của metadata => Nếu có sự thay đổi sẽ thực hiện các lệnh đã config trong cfn-auto-reloader.conf
     - Set lệnh cho cfn-auto-reloader.conf là cần release bản update => khi này web sẽ được cập nhật với nội dung "hello world edited"
  ```

   - Được cấu hình trong /etc/cfn/cfn-hub.conf
   - Default interval check sự thay đổi của resource trong metadata là 15 (minutes)
   - Sau khi kiểm tra định kỳ theo interval, nếu đã tìm thấy sự thay đổi (changes) thì CFN-hub sẽ chạy file /etc/cfn/hooks.d/cfn-auto-reloader.conf

 #### NOTE:
   - Khi update stack không thể thay đổi tên stack
   - Khi update stack failure => sẽ tự động rollback về version trước đó mà có trạng thái là hoạt động tốt
   - Đối với nested stacks (Stack trong stack) thì khi update stack con sẽ luôn thực hiện update stack cha
   - Có thể sử dụng "DependsOn" để chỉ định resouce phụ thuộc vào 1 resouce khác, tương tự như docker-compose
   - Đối với template có tạo resource cấp quyền ví dụ như IAM::Role, IAM::User,... thì cần xác nhận cụ thể cho phép CloudFormation có 1 trong 2 khả năng sau:
     - Nếu sử dụng IAM resources, có thể chỉ định 1 trong 2 <b>CAPABILITY_IAM</b> hoặc <b>CAPABILITY_NAMED_IAM</b>
     - Nếu là resources IAM custom name thì cần chỉ định <b>CAPABILITY_NAMED_IAM</b>
     - Nếu không chỉ định rõ capabilities (khả năng) này cho resources, CloudFormation sẽ trả về lỗi <b>InsufficientCapabilities</b>
   - Để định nghĩa 1 hàm lambda trong cloudFormation, có thể sử dụng 2 cách:
     - Viết lambda function inline, tuy nhiên template yml sẽ có giới hạn nên nếu làm cách này chỉ viết với hàm lambda đơn giản, nên sử dụng cách số 2
     - zip code function rồi đưa lên S3, ở template CloudFormation sử dụng !Sub để reference đến object S3 (function lambda zip) đó

     ```
     MyFunction:
     Type: AWS::Lambda::Function
     Properties:
       Code:
         S3Bucket: !Sub 'lambda-zips-${AWS::Region}'
       ...
     ```
<hr/>

### CloudFront
 - CloundFront route tất cả request đến primary origin, chỉ gửi request đến origin phụ khi một request đến origin chính không thành công
<hr/>

### Cognito User Pools
 - Cung cấp dịch vụ quản lý, đăng ký, đăng nhập user (Có thể hiểu usecase sử dụng cho 1 app serverless, chỉ sử dụng lambda, API gateway chẳng hạn)
 - Tích hợp sẵn, cung cấp giao diện đăng ký đăng nhập cho user, có thể custom lại view
 - Hỗ trợ đăng ký, đăng nhập với bên thứ 3 như Facebook, Google, Amazon hoặc Apple hoặc những bên khác sử cung cấp SAML
 - Quản lý được data users
 - Hỗ trợ MFA (xác thực đa yếu tố), xác thực qua điện thoại, email
 - Sau khi xác thực người dùng thành công sẽ trả về token JWT (JWT này sẽ có quyền thao tác với các resources AWS khác của mình tùy thuộc vào setting, ví dụ có thể cấp quyền cho JWT đó access S3,...)
 - Sử dụng được với javascript, Android, IOS
<hr/>

### CloudTrail
 - Theo dõi và ghi lại hoạt động của ACCOUNT đối với các resource AWS => khi cần check lại thay đổi resource theo scope USER NÀO thay đổi => nghĩ đến cloudtrail
 - Mặc định các log trong cloudtrail được mã hóa SSE
 - Mặc định cloudtrail sẽ chỉ track những actions ở bucket-level, nếu muốn track đến object-level thì cần bật S3 data events
 - NOTE
   - cloudtrail sẽ không ghi lại sự kiện 'CreateVolume' khi EBS được tạo trong quá trình launch EC2
<hr/>

### CloudWatch
 - EC2 monitoring
   + Default là 5 minutes
   + Có thể setting detail monitoring => 1 minutes (thêm phí)
   + Có thể dùng custom metric để put metric theo nhu cầu (VD: 1s, 5s, 10s,...)
<hr/>

### CodeBuild
 - What: Code build được sử dụng với mục đích build, test version code nào đó
 - Có thể sử dụng GitHub, GitHub Enterprise, BitBucket, AWS CodeCommit hoặc Amazon S3 để làm source
 - Có thể sử lý đồng thời nhiều bản build
 - Environment variables: Có 3 dạng
   + Plain text: Dạng string thường
   + SSM
   + Secret manager
 - Artifact: Là đầu ra của codebuild sau khi CodeBuild thực hiện build code => sẽ được lưu trữ ở đâu đó, vd như S3
<hr/>

### CodeCommit
 - connect: Sử dụng https hoặc ssh
   + Không thể sử dụng được ssh đối với root account
   + Có thể sử dụng HTTPS đối với root account nhưng NOT RECOMMENDED
   + Nên sử dụng IAM user để connect
<hr/>

### CodeDeploy
 #### Basic
 - Hỗ trợ auto deploy
 - Sử dụng file appspec.yml/appspec.yaml trong root-directory
 - Cần cài CodeDeploy Agent (Install docs: https://docs.aws.amazon.com/codedeploy/latest/userguide/codedeploy-agent-operations-install.html)
 - Hỗ trợ deploy đối với
   - EC2 / On-premises
   - ECS
   - Lambda

 #### Các chiến lược deploy (deployment strategy)
   - In-place deployment
     - Application trong nhóm target deploy sẽ bị dừng và deploy code, settings,... mới nhất rồi sau đó khởi động lại
     - Chỉ hoạt động đới với EC2 / On-premises
   - Blue-Green deployment: Giải quyết vấn đề downtime, mục đích và ý tưởng là sử dụng 2 tài nguyên phần cứng song song giống hệt nhau để tạo ra 2 môi trường etc staging, production,... (blue và green).
     - VD
       - User đang sử dụng version code 1 (Blue)
       - Version mới được deploy lên môi trường giống hệt blue => môi trường green
       - Sau khi deploy xong nếu check OK => route traffic qua môi trường mới (green)
       ![image](https://user-images.githubusercontent.com/57032236/180630810-05112a08-df77-4f7b-88c4-c0237e582d35.png)

     - Ưu điểm:
       - Zero downtime
       - Dễ dàng rollback vì đã có sẵn môi trường blue
 #### Deployment configuration
   - For EC2 / On-premise

     |Deployment configuration|Description|
     | ------------- | ------------- |
     |CodeDeployDefault.AllAtOnce|Thực hiện việc deploy cho tất cả instances cùng 1 lúc. Nếu ít nhất 1 instance deploy thành công => Trạng thái tổng thể của deployment đó sẽ hiển thị là <b>SUCCESS</b>, Nếu không có bất cứ 1 instance nào deploy thành công => Trạng thái tổng thể sẽ là <b>FAILURE</b>. Ví dụ có 9 instances, deploy 1 cái thành công, 8 cái thất bại => trạng thái tổng thể là |
   - For ECS
     - Khi thực hiện deploy đối với ECS, deployment configuration sẽ quyết định việc chuyển traffic từ version cũ sang version mới như thế nào. Có 3 kiểu là: canary, linear và all-at-once. Đối với canary và linear có thể tạo custom configuration

     |Deployment configuration|Description|
     | ------------- | ------------- |
     |CodeDeployDefault.ECSLinear10PercentEvery1Minutes|Chuyển 10% traffic mỗi 1 phút cho đến khi chuyển toàn bộ traffic sang version mới|
     |CodeDeployDefault.ECSLinear10PercentEvery3Minutes|Chuyển 10% traffic mỗi 3 phút cho đến khi chuyển toàn bộ traffic sang version mới|
     |CodeDeployDefault.ECSCanary10Percent5Minutes|Chuyển 10% traffic lần đầu tiên, sau 5 phút sẽ chuyển nốt 90% traffic còn lại|
     |CodeDeployDefault.ECSCanary10Percent15Minutes|Chuyển 10% traffic lần đầu tiên, sau 15 phút sẽ chuyển nốt 90% traffic còn lại|
     |CodeDeployDefault.ECSAllAtOnce|Chuyển toàn bộ traffic 1 lúc|
   - For ECS (deploy thông qua CloudFormation blue/green strategy)
     - Tương tự như ECS thường nhưng KHÔNG tạo được custom configuration
   - For Lambda
     - Khi thực hiện deploy đối với Lambda, deployment configuration sẽ quyết định việc chuyển traffic từ function lambda version cũ sang version mới như thế nào. Có 3 kiểu là: canary, linear và all-at-once. Đối với canary và linear có thể tạo custom configuration

     |Deployment configuration|Description|
     | ------------- | ------------- |
     |CodeDeployDefault.LambdaCanary10Percent5Minutes|Chuyển 10% traffic lần đầu tiên, sau 5 phút sẽ chuyển nốt 90% còn lại|
     |CodeDeployDefault.LambdaCanary10Percent10Minutes|Chuyển 10% traffic lần đầu tiên, sau 10 phút sẽ chuyển nốt 90% còn lại|
     |CodeDeployDefault.LambdaCanary10Percent15Minutes|Chuyển 10% traffic lần đầu tiên, sau 15 phút sẽ chuyển nốt 90% còn lại|
     |CodeDeployDefault.LambdaCanary10Percent30Minutes|Chuyển 10% traffic lần đầu tiên, sau 30 phút sẽ chuyển nốt 90% còn lại|
     |CodeDeployDefault.LambdaLinear10PercentEvery1Minute|Chuyển 10% traffic mỗi 1 phút cho đến khi chuyển xong toàn bộ|
     |CodeDeployDefault.LambdaLinear10PercentEvery2Minutes|Chuyển 10% traffic mỗi 2 phút cho đến khi chuyển xong toàn bộ|
     |CodeDeployDefault.LambdaLinear10PercentEvery3Minutes|Chuyển 10% traffic mỗi 3 phút cho đến khi chuyển xong toàn bộ|
     |CodeDeployDefault.LambdaLinear10PercentEvery10Minutes|Chuyển 10% traffic mỗi 10 phút cho đến khi chuyển xong toàn bộ|
     |CodeDeployDefault.LambdaAllAtOnce|Chuyển toàn bộ traffic 1 lúc sang lambda function version mới |
     
 #### Lifecycle Event
   - Hooks
   ![image](https://user-images.githubusercontent.com/57032236/183232015-edf2a8ba-0642-4e7b-8ee2-7df9945ee86e.png)
   - Environment Variable Avaibility for Hooks
     - APPLICATION_NAME
     - DEPLOYMENT_ID
     - DEPLOYMENT_GROUP_NAME
     - DEPLOYMENT_GROUP_ID
     - LIFECYCLE_EVENT
     - For case bundle from Amazon S3
       - BUNDLE_BUCKET
       - BUNDLE_KEY
       - BUNDLE_VERSION
       - BUNDLE_ETAG
     - For case bundle from Amazon S3
       - BUNDLE_COMMIT
 #### Trigger, log, event
 - Có thế kết hợp với AWS CloudWatch (setting trong AWS CloudWatch) để thông báo khi deployment chuyển state (failure, success,...), 1 số usecase như:
   - Gửi message slack thông báo trạng thái deployment
   - call lambda function làm gì đó
   - Kết hợp với AWS CloudWatch alarm để tự động hóa tắt, khởi động,... lại instances
 - Có thể ghi log vào CloudWatch log bằng cách cài CloudWatch Logs agent cho instances EC2, không thể ghi log deployment trực tiếp vào CloudWatch log nếu không sử dụng agent
 - Có thể sử dụng deployment trigger để trigger đến AWS SNS (setting ngay trong codedeploy)
 #### Rollback
   - Có 2 loại rollback là: Manual Rollbacks và Automatic Rollbacks
     - Manual Rollbacks (Khi checked "Disable rollbacks" trong mục "Rollbacks" trong deployment group)
     - Automatic Rollbacks (Khi không check "Disable rollbacks" trong mục "Rollbacks" trong deployment group), có thể chọn 1 trong 2 option hoặc cả 2
       - "Roll back when a deployment fails" => Khi deploy chuyển state sang failure => rollback
       - "Roll back when alarm thresholds are met" => Khi thỏa mãn điều kiện alarm => rollback
          - VD: Khi deploy nếu CPU tăng quá 80% và mong muốn rollback => setting alarm => rollback khi thỏa mãn điều kiện CPU > 80%
          - Alarm được chọn ngay trong deployment group (nhưng tạo alarm thì cần tạo ở CloudWatch Alarm)
       - Khi rollback sẽ rollback về revision gần nhất mà có trạng thái là SUCCESS
         revision_1 (SUCCESS) -> revision_2 (FAILURE) -> revision_3 (FAILURE) -> current_revision
         => Khi rollback sẽ rollback về revision_1
 #### Use CodeDeploy with On-Premises instances
   - Step 1: Config và đăng ký các instances đó với CodeDeploy
     - Để đăng ký instances với CodeDeploy có 2 cách
       - Cách 1: Sử dụng ARN của IAM user (Not recommended)
         - Sử dụng AWS CLI chạy lệnh [register](https://docs.aws.amazon.com/cli/latest/reference/deploy/register.html), chỉ nên sử dụng lệnh này đối với môi trường không phải production vì kém bảo mật do sử dụng credentials tĩnh vĩnh viễn
         - Sử dụng AWS CLI chạy lệnh [register-on-premises-instance](https://docs.aws.amazon.com/cli/latest/reference/deploy/register-on-premises-instance.html), do là đăng ký thủ công nên sẽ thích hợp với trường hợp số lượng ít instances
       - Cách 2: Sử dụng ARN của IAM role
         - Sử dụng AWS CLI chạy lệnh [register-on-premises-instance](https://docs.aws.amazon.com/cli/latest/reference/deploy/register-on-premises-instance.html) và sử dụng credentials tạm thời để xác thực bằng AWS Security Token Service (AWS STS). Cách này mức bảo mật cao nhất vì chỉ sử dụng credentials tạm thời, khi hết hạn sẽ cần làm mới => Thích hợp với production ở mọi cấp độ
   - Step 2: Deploy
   * DOCS: https://docs.aws.amazon.com/codedeploy/latest/userguide/on-premises-instances-register.html
<hr/>

### CodeBuild
- Hỗ trợ define các step buildcode
- Sử dụng file buildspec.yml trong root-directory
- Sử dụng local build bằng CodeBuild Agent để: test, debug
<hr/>

### CodePipeline
- Là một service CI/CD, hiểu nôm na là kết hợp giữa source code (AWS CodeCommit, Github,...), CI (AWS CodeBuild, jenkins,...), CD (AWS CodeDeploy,...) giúp tạo nên 1 luồng hoàn chỉnh để tạo nên 1 hệ thống CI/CD. Cụ thể hơn thì CodePipeline là tập hợp các stage (các stage này có thể custom, tuỳ biến theo nhu cầu), mỗi stage là 1 tập hợp các action mong 

![image](https://k21academy.com/wp-content/uploads/2021/03/php-project-release-pipeline.png)

- Mỗi stage sẽ tạo ra "artifacts", artifact đó sẽ được sử dụng cho stage tiếp theo
- Actions
  - CodePipeline bao gồm các action
   - Code
   - Build
   - Test
   - Deploy
   - Approval
   - Invoke
  - Có 2 cách thực thi của action:
    - Sequential (Chạy tuần tự): Xong action trên mới chạy action dưới
    - Parallel (Song song): Các action ngang hàng nhau => Chạy song song không đồng bộ
  - Thứ tự chạy, cách thực thi là song song hay tuần tự sẽ được quyết định bơi runOrder, thứ tự cao sẽ chạy sau thứ tự trước, thứ tự bằng nhau sẽ chạy song song. VD: 1 -> 2 && 2 -> 3
 
  <img width="1109" alt="Screen Shot 2023-03-02 at 22 28 51" src="https://user-images.githubusercontent.com/57032236/222473874-de8caaad-e926-46a6-8370-927d98032bbc.png">
  => Deploy và uploadToS3 là action parallel
  
  <img width="1109" alt="Screen Shot 2023-03-02 at 22 32 36" src="https://user-images.githubusercontent.com/57032236/222474317-c22f206d-61ba-456c-850d-acebb45d8dbc.png">
  => Manual approve phải success thì action DeployToProduction mới được phép thực 

![image](https://d2908q01vomqb2.cloudfront.net/7719a1c782a1ba91c031a682a0a2f8658209adbf/2018/05/24/Screen-Shot-2018-05-24-at-9.03.04-AM.png)
<hr/>

### DynamoDB
 - Sử dụng global table nếu có người dùng phân phối toàn cầu => giảm khoảng cách vật lý giữa client và DynamoDB endpoint => Giảm độ trễ
 - Eventually consistent VS Strongly consistent
   - #### Eventually consistent (Đọc nhất quán cuối cùng)
     - Response có thể sẽ không phải data mới nhất, có thể hiểu rằng response sẽ là data tại lúc call, trong quá trình call, data có được thay đổi cũng sẽ không được trả về data mới nhất => nhanh hơn so với strongly consistent
   - #### Strongly consistent (Đọc nhất quán mạnh mẽ)
     - Trả về dữ liệu được cập nhật mới nhất, tuy nhiên sẽ đi kèm với 1 số nhược điểm:
       - Có độ trễ cao hơn so với eventually consistent
       - Không hỗ trợ đối với global secondary indexes (chỉ mục thứ cấp toàn cầu)
       - Sử dụng nhiều read capacity hơn so với eventually consistent
     => Để improve performance đối với DynamoDB, luôn hướng tới sử dụng Eventually consistent bất cứ khi nào có thể
 - DynamoDB mặc định sử dụng Eventually consistent, trừ khi setting khác
 - Các câu lệnh GetItem, Query, Scan cho phép thêm option ConsistentRead = true để sử dụng Strongly Consistent trong quá trình thao tác.
 - #### Amazon DynamoDB Accelerator (DAX)
   - Là bộ nhớ đệm, có khả năng sử dụng cao, được thiết kế riêng cho DynamoDB => dùng để improve performance khi hệ thống NẶNG VỀ READ
   - Cải thiện performance lên đến 10 lần - từ mili second -> micro second ngay cả khi có hàng triệu request mỗi giây
   - Tính phí theo giờ và các instance DAX không cần cam kết dài hạn
 - #### request options:
   - ProjectionExpression: Sử dụng để filter attributes thay vì get all attributes
   - FilterExpression: Thêm điều kiện lọc records
 - Cân nhắc sử dụng Global table để improve performance khi có user ở nhiều nơi trên khắp thế giới, có thể chỉ định region table DynamoDB khả dụng => giảm khoảng cách vật lý đối với client => giảm độ trễ
 - WCU & RCU
   - WCU: total_item * (item_size/1KB) = WCU
     VD:
        - Để write 10 items/giây, mỗi item = 2KB => 10 * (2 / 1) = 20 => Cần 20 WCU/s
        - Để write 6 items/giây, mỗi item = 4.5KB => 6 * (5 / 1) = 30 => Cần 30 WCU/s
   - RCU:
     - 1 RCU = 1 strongly consistent read (đối với mỗi item size = 4KB)
     - 1 RCU = 2 eventually consistent read (đối với mỗi item size = 4KB)
     VD:
        - Để sử dụng 10 strongly consistent read / giây đối với mỗi item 4KB
          => 10 * (4 / 4) = 10 RCU/s
        - Để sử dụng 10 strongly consistent read / giây đối với mỗi item 6KB
          => 10 * (8 / 4) = 20 RCU/s (Làm tròn size item lên, 4 = 4, 6 = 8, 9 = 12, 10 = 12)

        - Để sử dụng 16 eventually consistent read / giây đối với mỗi item 12KB
          => (16 / 2) * (12 / 4) => 24 RCU/s
<hr/>

### EC2
 - Dedicated instance vs Dedicated host
   - Dedicated instance (phiên bản chuyên dụng): Chuyên dụng riêng cho 1 khách hàng, được ngăn cách với các instance khác về mặt vật lý ở cấp phần cứng, kể cả có liên kết đến cùng 1 tài khoản thanh toán. Tuy nhiên các instance này có thể chia sẻ phần cùng với các instance khác nếu chung 1 tài khoản
   - Dedicated host (Máy chủ chuyên dụng): Là một máy chủ vật lý riêng biệt
 
   => Sự khác nhau giữa dedicated instance và dedicated host
   ![image](https://user-images.githubusercontent.com/57032236/180655410-dc9f4b59-1483-4684-bd89-cc12fc1a911c.png)
 - Có thể enable/disable DeleteOnTermination đối với EBS để quy định khi terminate instance có xoá ổ EBS hay không. Hành động này có thể thực hiện được kể cả khi EC2 running
<hr/>

### Elastic load balancer
 - Khi tạo load balancer => cần tạo target group => cần chỉ định target type => Khi tạo xong target group KHÔNG THỂ thay đổi target type
 - #### Target types
   - instance: chỉ định bởi các ID instance
   - ip: Chỉ định bởi các IP (Có thể đặt range dải mạng, etc: 10.0.0.0/8)
   - Lambda: Mục tiêu là 1 hàm lambda
 - Sử dụng header X-Forwarded-For để xem IP của client request nếu sử dụng load balancer
 - Error Code:
   - 500: Lỗi server
   - 503(Service unavailable): Báo chưa có target (target group chưa add target)
   - 504: Timeout (không kết nối được với target khi hết thời gian timeout)
   - 403: bị chặn bởi AWS WAF
 - Khi ELB được tạo, nó sẽ nhận được tên DNS công khai, tên DNS này không thay đổi, nhưng public IP có thể thay đổi => nên sử dụng DNS thay vì public IP
 - Khi muốn phân tích về IP client request cũng như độ trễ request thì có thể sử dụng ALB access logs
 - ELB giao tiếp với các EC2 instances bằng cách sử dụng IP private
 - Để tránh việc re-authenticate cho client khi sử dụng ELB (nhiều instance sẽ không cùng session nếu không sử dụng chung 1 nơi lưu trữ session), ta có thể enable sticky sessions, khi nhận request lần đầu từ client nếu chưa có ghi nhận nào trước đó, nó sẽ tạo ra 1 cookie có tên là AWSALB, sau đó khi client cũ đó gửi request lại, nó sẽ routing cho request đó đến instance trước đó đã xử lý request để tránh re-authenticate
<hr/>

### Elastic Beanstalk
 - Sử dụng để triển khai, mở rộng các web application được viết bằng Java, .NET, PHP, Node.js, Ruby, Python, Go và Docker trên những máy chủ: Apache, Nginx, Passenger và IIS
 - Chỉ cần upload code lên mà không cần lo về vấn đề deployment, performance về mặt infra, monitoring,...
 - Vẫn có quyền access vào các resources
 - Không tính thêm phí, chỉ tính phí các tài nguyên sử dụng như: EC2, ECS,...
 #### Config
  - Có 3 cách config Beanstalk
    - Sử dụng ebextensions/
    - Sử dụng saved config
    - Config trực tiếp sử dụng CLI, console, SDK,...

   Thứ tự ưu tiên apply vào EB sẽ là: Setting trực tiếp > Saved configurations > Config files (.ebextensions/) > default value
 #### Deployment stragies
   - All-at-Once: Tất cả instances trong 1 lần
     ![image](https://user-images.githubusercontent.com/57032236/180630279-cb57e0fb-59c5-48c7-add6-2fb153e2117c.png)
     
   - Rolling: Deploy thành nhiều đợt cuốn chiếu
     ![image](https://user-images.githubusercontent.com/57032236/180630281-b6908b64-4d07-49dd-bd16-f2ca3db200e3.png)
     
   - Rolling with Additional Batch: Tách ra thành nhiều lô, lô đầu tiên sẽ tạo ra số instance EC2 mới thay vì các instances EC2 đã có (1 lô có thể chỉ định số lượng instances).
     ![image](https://user-images.githubusercontent.com/57032236/180630288-3cb97a08-0658-4619-849a-97208d6994f1.png)

   - Immutable: Triển khai toàn bộ instance mới, sau đó sẽ swap instances mới với instance cũ
     ![image](https://user-images.githubusercontent.com/57032236/180630302-81cb5f2e-a36f-4b76-a8a9-20b80820e4e0.png)

   - Traffic Splitting: Deploy theo chiến lược Immutable tuy nhiên sẽ cắt số traffic nhất định đến instances mới để thử nghiệm, nếu hoạt động tốt sẽ chuyển 100% lưu lượng sang instances mới
   - Blue/Green (đọc bên dưới trong AWS CodeDeploy)
 - Đặt config theo quy ước đặt tên: .ebextensions/<mysettings>.config
 - Nếu đặt các service trong ebextensions => khi deploy lại sẽ bị restart lại => data cũ sẽ không được giữ lại
   => Muốn giữ lại data cũ cần tách riêng service đó ra rồi tham chiếu vào Elastic Beanstalk
   VD: Khi init RDS trong .ebextensions/ => khi deploy lại thì RDS cũng sẽ bị restart => để tránh trường hợp này thì nên tạo RDS riêng rồi ánh xạ host & port trong Beanstalk. Trong trường hợp init trong .ebextensions/ mà muốn giữ lại data cũ thì cần snapshot lại RDS
<hr/>

### Elastic Container Registry (ECR)
 - Là một dịch vụ để lưu trữ, quản lý các image docker tương tự dockerhub
<hr/>   

### ECS
 - Dùng để quản lý các container docker
 - Để config ECS, viết config trong file /etc/ecs/ecs.config
   - ECS_ENABLE_TASK_IAM_ROLE: Sử dụng để kích hoạt IAM role cho container
<hr/>

### EBS
 - Có thể hiểu EBS như 1 ổ đĩa thêm cho EC2
 - Nếu bật chế độ Encryption by default, từ sau đó trở đi các EBS mới được tạo ra sẽ được mã hóa theo mặc định
   - Mã hóa theo mặc định là theo region, không thể chỉ định đặc biệt EBS nào được mã hóa EBS nào không trong cùng 1 region
 - EBS hỗ trợ cả encrypt on the fly & encrypt at rest
<hr/>
 
### EFS
 - Là service lưu trữ dạng file (tệp)
 - EFS Standard - IA: tự động tiết kiệm chi phí lưu trữ đối với các tệp ít được truy xuất
<hr/>
 
### ElasticCache
 - Strategies
   - Lazy: Chỉ cached khi cần thiết, data có thể bị cũ
     ```
     // *****************************************
     // function that returns a customer's record.
     // Attempts to retrieve the record from the cache.
     // If it is retrieved, the record is returned to the application.
     // If the record is not retrieved from the cache, it is
     //    retrieved from the database, 
     //    added to the cache, and 
     //    returned to the application
     // *****************************************
     get_customer(customer_id)

         customer_record = cache.get(customer_id)
         if (customer_record == null)

             customer_record = db.query("SELECT * FROM Customers WHERE id = {0}", customer_id)
             cache.set(customer_id, customer_record)

         return customer_record
     ```
   - Writethrough: Khi có sự kiện ghi, đồng thời ghi đè vào data cũ ở bộ nhớ đệm + update vào database => Dữ liệu luôn luôn mới nhất
     ```
     // *****************************************
     // function that saves a customer's record.
     // *****************************************
     save_customer(customer_id, values)

         customer_record = db.query("UPDATE Customers WHERE id = {0}", customer_id, values)
         cache.set(customer_id, customer_record)
     return success
     ```
   - TTL: set time to live cho cache
     ```
     // *****************************************
     // function that saves a customer's record.
     // The TTL value of 300 means that the record expires
     //    300 seconds (5 minutes) after the set command 
     //    and future reads will have to query the database.
     // *****************************************
     save_customer(customer_id, values)

         customer_record = db.query("UPDATE Customers WHERE id = {0}", customer_id, values)
         cache.set(customer_id, customer_record, 300)

         return success
     ```
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
 - Có thể authenticate database bằng IAM database authentication đối với 2 loại DB (Xác thực DB bằng chuỗi authen mà không cần mật khẩu, mỗi chuối này có lifetime = 15 minutes)
   - RDS MySQL
   - RDS PostGreSQL
<hr/>

### Inspector
 - Là một dịch vụ đánh giá bảo mật tự động giúp cải thiện tính bảo mật và tuân thủ của các application được deploy trên AWS
 - Đánh giá về:
   - exposure (mức độ phơi nhiễm)
   - vulnerabilities (lỗ hổng bảo mật)
   - deviations (độ sai lệch so với best practice)
<hr/>

### Kinesis
 - Hỗ trợ việc thu thập, phân tích dữ liệu real-time
 - Xử lý dữ liệu ở bất kỳ quy mô nào
 - Nếu muốn xử lý case đầu vào kinesis tăng 1 cách đột ngột thì có thể config retry (with exponential backoff) => điều này sẽ retry push data vào kinesis nếu bị lỗi
 - Nếu lượng data đầu vào nhiều thì cần tăng số lượng shards, có thể hiểu mỗi shards như phân đoạn dữ liệu, càng nhiều shard càng chứa được nhiều data để xử lý
 - Mỗi 1 shard tương ứng tối đa 1 EC2 đóng vai trò làm worker
 - Đối với các region US East (N. Virginia), US West (Oregon), and Europe (Ireland). Mỗi region tối đa 500 shards/account AWS, các region khác tối đa 200 shards/account AWS
   ![image](https://user-images.githubusercontent.com/57032236/183232267-35364256-6300-48ba-83a3-9cc79b066615.png)
 - Hỗ trợ thu thập dữ liệu như:
   - Âm thanh
   - Log
   - Video
   - click chuột
 - #### Amazon Kinesis Video Streams
   - Thu thập, xử lý và lưu trữ dữ liệu video từ nguồn phát để phục vụ cho việc phát lại, phân tích và machine learning đến AWS
   - Use case:
     - Smart home: Kết hợp với các thiết bị gia đình có trang bị camera như chuông, camera, webcam,...Sau đó có thế sử dụng dữ liệu để tạo ra các app thông minh gia đình (Ví dụ: Tương tác với chuông cửa hỗ trợ camera từ điện thoại di động của bạn)
     ![image](https://user-images.githubusercontent.com/57032236/180633334-2c375e6e-66db-442e-8317-ab877fada119.png)

     - Smart city: Kết hợp với camera tại đèn giao thông, bãi đỗ xe, trung tâm mua sắm,... để thu thập dữ liệu, có thể sử dụng để phân tích góp phần nào hạn chế tội phạm, vi phạm
     ![image](https://user-images.githubusercontent.com/57032236/180633309-39229ec8-47c9-4397-8db2-d8d0eacf9a14.png)
   
     - Tự động hóa công nghiệp: Có thể sử dụng để thu thập dữ liệu như tín hiệu RADAR, LIDAR, một vài dữ liệu ở thiết bị công nghiệp khác. Từ đó có tể phân tích dữ liệu bằng machine learning để sử dụng, ví dụ như dự báo tuổi thọ của van, đặt lịch thay thế linh kiện trước => giảm thiểu thời gian dừng hoạt động và lỗi dây chuyền sản xuất
     ![image](https://user-images.githubusercontent.com/57032236/180633355-05fc0169-ba43-452a-adce-ceaa63b83b8b.png)

 - #### Kinesis Data Streams
   - Thu thập log, dữ liệu cảm biến, dữ liệu nhấp chuột,...để phân tích, trả về output trong thời gian thực, ví dụ như bảng xếp hạng hoặc có thể chuyển data vào các hồ dữ liệu khác
   - Phân tích theo thời gian thực trong vài giây bằng lambda hoặc Amazon Kinesis Data Analystics
   ![image](https://user-images.githubusercontent.com/57032236/180633089-65f0f824-6fd7-4398-baae-5d3cba13b574.png)

 - #### Kinesis Data Firehose
   - Sử dụng để tải các stream data vào hồ dữ liệu, kho chứa hoặc dịch vụ phân tích khác theo thời gian thực (hiểu đơn giản là dịch vụ chuyển tiếp data cho kenesis)
   - Cho phép thu thập, chuyển đổi và tải data vào đích đến nào đó
   ![image](https://user-images.githubusercontent.com/57032236/180633192-cef46368-4269-4213-99ac-b05c814da870.png)
   - Usecase
     - Chuyển đổi, truyền data vào S3, hồ dữ liệu khác sang các định dạng khác mà không cần xây quy trình xử lý khác
     
 - #### Kinesis Data Analytics
   - Phân tích data cho dữ liệu phát trực tuyến trong thời gian thực bằng cách sử dụng Apache Flink
   ![image](https://user-images.githubusercontent.com/57032236/180633606-7d2ca76e-d305-48fd-8464-d1139db6e426.png)
<hr/>

### KMS
 - Sử dụng để tạo, quản lý các khóa
 - Tích hợp với CloudTrail để ghi lại nhật ký sư dụng khóa
 - Không hỗ trợ rotate
 - Hỗ trợ 3 loại CMK (Customer master key)
   - Customer managed key: Do chính mình tạo, sở hữu và quản lý toàn quyền
   - AWS managed key: Khóa do AWS thay mặt mình tạo và quản lý bởi AWS
   - AWS owned key: Là các khóa do AWS sở hữu và được sử dụng chung đối với nhiều tài khoản AWS (sẽ không hiển thị trên console)
   
 ![image](https://user-images.githubusercontent.com/57032236/180655975-5eb54e40-f48b-469a-9a4d-fd2681396288.png)
<hr/>

### Lambda
 - Càng tăng RAM thì CPU càng tăng (không thể tách biệt 1 trong 2, min = 128MB, max = 10GB)
 - Để cải thiện perfomance có thể
   - Tách biệt phần kết nối DB khỏi hàm(ví dụ có 10 request liên tục, nếu nhét connect DB vào trong hàm thì có mỗi lần xử lý sẽ phải connect lại, còn nếu tách ra ngoài thì chỉ cần connect 1 lần, rồi 9 lần còn lại chỉ cần xử lý logic sau đó mà k cần kết nối lại DB)
 - Khi thay đổi nội dung hàm lambda, cần thực hiện deploy lại nếu không sẽ vẫn chạy code cũ
 - Lambda chỉ support image linux-based khi muốn chạy container
 - Muốn test container trên lambda có thể sử dụng Lambda Runtime Interface Emulator
 - Lambda chỉ sử dụng được image docker được lưu ở ECS trên cùng 1 account
 - Tổng size environment variable không được vượt quá 4KB, không limit số lượng
 - Có thể sử dụng ổ đĩa tạm thời /tmp để lưu trữ dữ liệu (disk size = 512MB), sau khi lambda kết thúc chương trình chạy => ổ đĩa này sẽ được
 - Giới hạn concurrency/AWS region là 1000 (của all lambda function chứ không phải của 1 function)
   => vì thế có thể sẽ xảy ra trường hợp ví dụ như, có 3 function A, B, C. A và B sử dụng hết concurrency => khi C được gọi sẽ bị exception. Để giải quyết trường hợp này thì có thể set reserve concurrency (quy định số concurrency được sử dụng cho func), khi set như này thì sẽ không có func C sẽ không bị A và B chiếm concurrency.
   => Hoặc nếu thực sự cần thiết thì có thể raise ticket AWS support để tăng hạn nghạch concurrency lên
   
 #### Lambda layer
  - Khi code hàm lambda mà cần sử dụng thư viện gì đó thì có thể sử dụng lambda layer, kiểu dạng đóng gói rồi mang lên lambda, khi nào cần thì import vào
  - Có thể upload dạng bằng dạng file zip, file zip này có thể tải trực tiếp từ local lên hoặc lấy từ S3 về
 #### Version
  - 1 version là 1 snap shot của function tại thời điểm tạo, config và code của version đó là bất biến, không thể thay đổi.
 #### Aliases
  - Dùng để đặt tên cho 1 version lambda, ví dụ
   - $LASTEST -> DEV
   - Version2 -> TEST
   - Version1 -> PROD
  - Có thể chia traffic cho alias, ví dụ alias TEST có thể trỏ 20% đến version2 và 80% đến version1
<hr/>

### Organizations Service Control Policy (SCP)
 - Sử dụng để xác định quyền TỐI THIỂU cho các tài khoản trong organization hoặc Orgazination Unit (Đơn vị con của organization)
 - Chỉ sử dụng để hạn chế quyền, không dùng để cung cấp quyền
<hr/>

### Parameter Store
 - Cũng có thể sử dụng để lưu trữ các giá trị như SETTINGS, mật khẩu database tuy nhiên sẽ lưu dưới dạng THAM SỐ để có thể tham chiếu trên EC2, ECS, Lambda,...
 - Không hỗ trợ rotate
 - Sử dụng KMS để encrypt/decrypt
 - Standard parameter max size = 4KB, Advance parameter max size = 8KB
 - Được thiết kế để sử dụng cho nhiều usecase hơn ví dụ như setting URL, database name,...
 - Có thể lưu được dưới dạng plaintext, secret string
 - Không thể truy cập từ tài khoản AWS khác
 - Free khi sử dụng standard parameter, nếu sử dụng advance thì sẽ tính phí dựa trên số lượt gọi API
<hr/>

### Permissions boundary
 - Sử dụng để xác định quyền tối đa được phép đối với 1 thực thể IAM (user/role)
 - Chỉ sử dụng để hạn chế quyền, không dùng để cung cấp quyền
<hr/>

### RDS (Relationship database service)
 - là một tập hợp các dịch vụ database
   - Aurora
     - MySQL
     - PostgreSQL
   - MySQL
   - MariaDB
   - PostgreSQL
   - Oracle
   - SQL Server
   - Outposts (Cho phép toàn quyền quản lý database RDS ở môi trường On-Premises)
 - RDS Multi-AZ
   - áp dụng các bản patch cập nhật OS bằng cách upgrade ở standby -> chuyển standby thành primary -> upgrade old primary -> chuyển old primary thành standby
   - Ưu điểm
     - Tăng độ sẵn sàng (các AZ độc lập về mặt vật lý -> nếu chẳng may có AZ lỗi thì vẫn có bản dự phòng)
     - Nâng cao độ bền (Sao chép đồng bộ giữa nhiều AZ -> luôn được cập nhật theo phiên bản chính)
     - Độ sẵn sàng cao (Chuyển đổi CSDL dự phòng nếu có vấn đề xảy ra trong vòng 60s mà không cần can thiệp thủ công)
 - RDS hỗ trợ automatic backups, tuy nhiên thời gian lưu trữ chỉ từ 0 - 35 ngày, nếu cần lưu trữ lâu dài và lên lịch snapshot theo 1 thời gian biểu cố định thì có thể dùng cron job trong CloudWatch để lên job sử dụng lambda để thực hiện snapshot
 - Automatic backup chỉ backup trên 1 AZ, không hỗ trợ backup trên nhiều vùng
<hr/>

### Route 53
 - Dịch vụ dùng để định tuyến, phân giải domain sang dạng IP để điều hướng request
 - routing policy
   - Simple routing policy: Định tuyến bình thường, không có gì đặc biệt
   - Failover routing policy: Có thể hiểu là chủ động đặt routing khi tài nguyên unhealth
   - Geolocation routing policy: Định tuyến dựa trên khu vực địa lý của client
   - Geoproximity routing policy: Định tuyến dựa trên bản đồ tự định nghĩa
     ![image](https://user-images.githubusercontent.com/57032236/180654793-fa8c2c9b-6410-4299-876c-acec38213f4f.png)
   - Latency routing policy: Định tuyến dựa trên độ trễ
   - IP-based routing policy: Định tuyến dựa trên IP client
   - Multivalue answer routing policy
   - Weighted routing policy: Định tuyến dựa trên trọng số, ví dụ có 2 môi trường => có thể định tuyến 30% đến môi trường 1, 70% đến môi trường 2 (% là tùy ý)
<hr/>

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
   - Usecase: Nếu hệ thống quan trọng về thông lượng, không quan trọng về thứ tự message => Sử dụng standard
     VD: Có background job gửi email về chương trình giảm giá sản phẩm, khuyến mãi,...
 - #### Types
   - Standard
     - Không giới hạn thông lượng (throughput) (Không giới hạn số lượt gọi API SendMessage, ReceiveMessage, DeleteMessage trong 1s)
     - Đôi khi thứ tự delivery message sẽ khác với thứ tự đẩy vào queue
   - FIFO
     - Thông lượng cao nhưng có giới hạn
       - Nếu sử dụng batch (Thao tác với nhiều SQS message 1 lúc đối với các API SendMessageBatch, ReceiveMessage, DeleteMessageBatch) hỗ trợ tối đa tương tác với 3000 messages mỗi giây, mỗi API tương đương 10 messages => tương đương 300 lệnh gọi API mỗi giây. Để tăng hạn ngạch (quota) có thể tạo ticket support
       - Nếu không sử dụng batch => tối đa 300 lệnh gọi API mỗi giây (SendMessage, ReceiveMessage, DeleteMessage)
     - Áp dụng cơ chế FIRST IN - FIRST OUT => message vào queue trước sẽ được delivery trước
     - Usecase: Nếu hệ thống cần quan tâm đến thứ tự xử lý message
       VD: Khi web cho phép người dùng đăng ký khóa học, cần xử lý logic đăng ký tài khoản người dùng trước khi đăng ký khóa học
 - Sau khi tạo SQS queue => không thể thay đổi queue type
<hr/>
 
### S3:
 - Đối với event notification, nếu không bật chế độ version thì khi 2 request đều ghi vào 1 đối tượng => có thể sẽ chỉ có 1 event notification được trigger
 - Encryption
   - Client-side encryption: Object được mã hóa trước khi gửi lên S3
   - Server-side encryption: Việc mã hóa object được diễn ra ở phía server S3
     - SSE-S3: Mã hóa và khóa được quản lý bởi S3, mỗi object được mã hóa bằng 1 khóa duy nhất, sau đó mã hóa khóa đó bằng 1 khóa gốc khác, sử dụng tiêu chuẩn AES-256
     - SSE-KMS: Mã hóa ở phía S3, tuy nghiên sử dụng khóa KMS do chúng ta quản lý
     - SSE-C: Vẫn là mã hóa phía server tuy nhiên key là do client gửi lên (PHẢI SỬ DỤNG HTTPS)
 - S3 SELECT: hỗ trợ truy vấn select từ 1 tập objects, ví dụ có 1 list các file CSV, có thể chỉ định muốn lấy 3/10 columns trong list csv chỉ định đó
 - Khi object cần upload S3 đạt đến mức 100MB thì AWS khuyến cáo nên sử dụng multi upload
 - Versioning
   - Khi versioning được bật, những object trước đó sẽ có version ID = null
   - versioning được kích hoạt ở level bucket (all object in bucket), không thể chỉ áp dụng cho 1 số objects/folders chỉ định
 - Để phân trang có thể sử dụng combo: --starting-token & --max-items
 - Khi vừa xoá bucket xong mà sử dụng lệnh get list buckets, bucket vừa bị xoá vẫn có thể xuất hiện trong list trong 1 thời gian nhất định
<hr/>

### Serverless Application Model (SAM)
 - Sử dụng để định nghĩa, build các ứng dụng không máy chủ, nó cung cấp cú pháp viết tắt để diễn đạt các hàm, API, database và ánh xạ event
 ![image](https://user-images.githubusercontent.com/57032236/180653207-b8ee7264-5d25-4384-8084-478ce8b202de.png)

- SAM hỗ trợ các resource types
  - AWS::Serverless::Api

  - AWS::Serverless::Application

  - AWS::Serverless::Function

  - AWS::Serverless::HttpApi

  - AWS::Serverless::LayerVersion

  - AWS::Serverless::SimpleTable

  - AWS::Serverless::StateMachine
<hr/>

### Serverless Application Repository (SAR)
 - Là một kho các ứng dụng serverless có sẵn, có thể tái sử dụng lại
<hr/>

### Secret Manager
 - Sử dụng để quản lý các bí mật cần thiết để TRUY CẬP CÁC ỨNG DỤNG, dịch vụ và tài nguyên AWS, ví dụ như mật khẩu database, API key,...
 - Hỗ trợ automatic rotate (Xoay định kỳ)
 - Truy xuất dữ liệu trong AWS Secret Manager bằng cách call API
 - Max size = 64KB
 - Sử dụng KMS để encrypt/decrypt
 - Có thể truy cập từ tài khoản AWS khác
<hr/>
 
### Step Functions
 - Llà dịch vụ điều phối serverless cho phép tích hợp với các hàm AWS Lambda và các dịch vụ AWS khác để xây dựng các workflow phức tạp bằng cách chia nhỏ các bước, định nghĩa các step cần làm gì khi success, khi failed

 ![image](https://user-images.githubusercontent.com/57032236/233834210-85037aea-ca6b-46af-a5c5-8c19c59cde03.png)

 Như ví dụ này thì có thể gộp tất cả vào 1 hàm lambda, tuy nhiên hạn chế sẽ là:

 - Maximum timeout của 1 hàm lambda là 15 minutes => Hạn chế về mặt thời gian
 - Nếu gộp chung lại thành 1 hàm => hàm to => phức tạp
 => Để giải quyết thì sẽ dựng 1 StepFunction như ảnh mô tả

 ![image](https://user-images.githubusercontent.com/57032236/183231878-c24a7454-7b4d-4b55-88a1-cbbf004b94b7.png)

#### Usecase:
 ##### Function orchestration / Phối hợp chức năng
  ![image](https://user-images.githubusercontent.com/57032236/233834445-86794394-35ce-46fb-9bbe-8e8cac83d1d7.png)

  Tạo 1 flow bao gồm các hàm lambda, hàm trước gọi hàm sau, hàm sau gọi hàm sau nữa. Bằng cách này có thể thấy kết quả của từng step -> dễ dang debug xem sai ở step nào

 ##### Branching / Phân nhánh chức năng
  ![image](https://user-images.githubusercontent.com/57032236/233834542-1918861a-c811-478e-b651-103926520ded.png)

  VD: 1 user gửi request đến hệ thống yêu cầu tăng hạn mức thẻ tín dụng
      - Trong trường hợp nằm trong được phép thì có thể cho phép StepFunction call 1 hàm lambda tự động duyệt yêu cầu.
      - Còn nếu trường hợp vượt quá mức được phép duyệt tự động thì có thể gọi 1 hàm lambda khác gửi yêu cầu đó đến admin để duyệt thủ công.

 ##### Error handling / Xử lý lỗi
  ![image](https://user-images.githubusercontent.com/57032236/233834761-e1f92c28-3b9e-4209-9c59-95a94e500ba5.png)

  - Retry:
    VD: 1 khách hàng yêu cầu tên người dùng. Lần đầu tiên, yêu cầu của khách hàng không thành công. Sử dụng câu lệnh Thử lại, bạn có thể yêu cầu Step Functions thử lại yêu cầu của khách hàng. Lần thứ hai, yêu cầu của khách hàng thành công.
  - Catch:
    VD: Trong trường hợp sử dụng tương tự, khách hàng yêu cầu tên người dùng không khả dụng. Sử dụng câu lệnh Catch, Step Functions đề xuất tên người dùng khả dụng.

 ##### Human in the loop
  ![image](https://user-images.githubusercontent.com/57032236/233835150-7395ede6-fa46-483d-aa34-23fcdc419019.png)

   VD: Sử dụng một app ngân hàng, một user gửi tiền cho một người bạn. User đó chờ email xác nhận. Với 1 callback và task token, Step Function yêu cầu Lambda gửi tiền người bạn của user đó và báo cáo lại khi bạn của user nhận được. Sau khi Lambda báo cáo lại rằng bạn của user đã nhận được tiền, Step Functions chuyển sang bước tiếp theo trong workflow, đó là gửi cho user đó 1 email xác nhận.

 ##### Parallel processing / Tiến trình song song
  ![image](https://user-images.githubusercontent.com/57032236/233835237-4a5940c1-334c-406f-80c8-f80ae208fbfb.png)

  VD: Một user chuyển đổi tệp video thành năm độ phân giải màn hình khác nhau để người xem có thể xem video trên nhiều thiết bị. Sử dụng Parallel state, Step Functions nhập tệp video, Lambda có thể xử lý tệp đó thành năm độ phân giải màn hình cùng một lúc.

 ##### Dynamic parallelism / Song song động
  ![image](https://user-images.githubusercontent.com/57032236/233835327-48c6ccff-b3d2-468e-8c3e-d14be0b82b4d.png)

  VD: Một khách hàng đặt ba mặt hàng và bạn cần chuẩn bị từng mặt hàng để giao. Bạn kiểm tra tình trạng sẵn có của từng mặt hàng, tập hợp từng mặt hàng rồi đóng gói từng mặt hàng để giao. Khi sử dụng Map state, Step Functions để Lambda xử lý song song từng mặt hàng của khách hàng. Sau khi tất cả các mặt hàng của khách hàng của bạn được đóng gói để giao, Step Functions sẽ chuyển sang bước tiếp theo trong workflow, đó là gửi cho khách hàng một email xác nhận có thông tin theo dõi.
<hr/>

### X-Ray
- Là service hỗ trợ theo dấu và cho ra cái nhìn tổng quát khi ứng dụng sử dụng micro service (Ví dụ như web sử dụng các micro services => X-Ray sẽ chạy qua các micro service và tổng hợp lại cho ra 1 sơ đồ, từ đó có cái nhìn tổng quan)
![image](https://user-images.githubusercontent.com/57032236/180803943-97e0828c-c6e3-4233-bdba-41670e5bf4f8.png)
- Có thể sử dụng debug, tìm ra nguyên nhân gốc rễ
- Tương thích với EC2, ECS, Lambda, SQS, SNS và Elastic Beanstalk
- Sử dụng các ngôn ngữ: Java, Node.js, .NET
- Cần phải enable X-Ray sampling (active tracing) để có thể hoạt động
<hr/>

### Trusted advisor
 - Là một công cụ trực tuyến cung cấp phương pháp cải thiện in real-time theo best practice của AWS về:
   - cost optimization (tối ưu hóa chi phí)
   - security (bảo mật)
   - fault tolerance (khả năng chịu lỗi)
   - service limits (giới hạn dịch vụ một cách đơn giản nhất)
   - and performance improvement (cải thiện hiệu suất).
<hr/>

### NOTE SOMETHINGS
 - Có thể xem các thông tin của instance: subnet-id,...
   => access url này http://169.254.169.254/latest/meta-data/instance-id
<hr/>
