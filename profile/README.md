Kiến trúc microservice là một phương pháp thiết kế phần mềm trong đó ứng dụng được cấu trúc dưới dạng một tập hợp các dịch vụ nhỏ, độc lập. Mỗi dịch vụ này thực hiện một chức năng kinh doanh cụ thể và có thể được phát triển, triển khai, vận hành và mở rộng một cách độc lập. Đối với một ứng dụng tương tự Grab, chúng ta cần xem xét các dịch vụ chính sau đây:

![system-design by shyamzzp](https://raw.githubusercontent.com/shyamzzp/interview/main/docs/system-design/_img/uber/2022-06-25-15-29-30.png)

Quy trình làm việc (workflow) mô tả ở đây liên quan đến việc kết nối giữa hành khách và tài xế thông qua một dịch vụ gọi xe. Dưới đây là diễn giải chi tiết của quá trình này:

1. **Bob gửi yêu cầu đi xe**: Bob, người dùng dịch vụ, gửi yêu cầu đi xe đến Dịch vụ Ghép Đôi (Ride Matching service). Bob mong muốn tìm được một tài xế phù hợp để thực hiện chuyến đi.

2. **Dịch vụ Ghép Đôi liên lạc với Dịch vụ Định Vị**: Sau khi nhận được yêu cầu từ Bob, Dịch vụ Ghép Đôi sẽ liên hệ với Dịch vụ Định Vị để tìm kiếm tất cả các tài xế đang có mặt và sẵn sàng trong cùng khu vực.

3. **Xếp hạng và lọc tài xế**: Dịch vụ Ghép Đôi sẽ thực hiện các bước xếp hạng và lọc để chọn ra những tài xế phù hợp nhất với yêu cầu của Bob. Các tiêu chí có thể bao gồm vị trí của tài xế, đánh giá của tài xế, loại xe, và các yếu tố khác.

4. **Gửi thông báo đến tài xế**: Sau khi đã lọc và chọn được các tài xế phù hợp, Dịch vụ Ghép Đôi sẽ sử dụng Dịch vụ Thông Báo để gửi yêu cầu đi xe đến các tài xế đã chọn. Tài xế có thể chọn chấp nhận hoặc từ chối yêu cầu này.

5. **Xử lý yêu cầu của tài xế**: Khi một tài xế chấp nhận yêu cầu, Dịch vụ Ghép Đôi sẽ nhận thông tin và tiếp tục gửi chi tiết chuyến đi đến Dịch vụ Quản Lý Chuyến Đi (Trip Management service).

6. **Giám sát chuyến đi**: Dịch vụ Quản Lý Chuyến Đi sẽ theo dõi tiến trình của chuyến đi, đảm bảo mọi thứ diễn ra suôn sẻ. Họ cũng sẽ phát sóng vị trí của tài xế và hành khách cho tất cả các bên liên quan trong chuyến đi, giúp mọi người cập nhật thông tin về chuyến đi.

Quy trình này giúp đảm bảo chuyến đi được an toàn, thuận tiện và phù hợp với nhu cầu của người dùng.

### 1. Thiết kế kiến trúc tổng quan

Trước khi đi vào chi tiết, cần xác định một số thành phần chính của ứng dụng như sau:

- **API Gateway:** Là điểm nhập duy nhất vào hệ thống, xử lý việc chuyển tiếp yêu cầu đến các microservices phù hợp, và cung cấp một lớp bảo mật.
- **Service Discovery:** Quản lý việc phát hiện và đăng ký các microservice trong hệ thống.
- **Load Balancer:** Phân phối tải và yêu cầu đến các instances của microservice.
- **Centralized Configuration:** Quản lý cấu hình chung cho tất cả microservices.
- **Distributed Tracing and Logging:** Theo dõi và ghi nhận các yêu cầu xuyên suốt hệ thống.
- **Authentication and Authorization:** Xác thực và phân quyền người dùng.

### 2. Thiết kế các Microservice chính

Dưới đây là một số microservice cần thiết cho ứng dụng tương tự Grab:

#### 2.1 User Service

- **Chức năng:** Quản lý thông tin người dùng, bao gồm khách hàng và tài xế.
- **APIs:** Đăng ký, đăng nhập, cập nhật thông tin cá nhân, quản lý điểm thưởng, v.v.

#### 2.2 Driver Service

- **Chức năng:** Quản lý thông tin và trạng thái của tài xế.
- **APIs:** Cập nhật vị trí, trạng thái sẵn sàng nhận chuyến, lịch sử chuyến đi, đánh giá tài xế, v.v.

#### 2.3 Ride Matching Service

- **Chức năng:** Ghép chuyến xe dựa trên vị trí của khách hàng và tài xế.
- **APIs:** Tạo yêu cầu chuyến đi, hủy yêu cầu, tìm kiếm tài xế phù hợp, thông báo cho tài xế, v.v.

#### 2.4 Trip Management Service

- **Chức năng:** Quản lý các chi tiết của chuyến đi.
- **APIs:** Theo dõi trạng thái chuyến đi, lưu trữ lộ trình, tính toán giá tiền, v.v.

#### 2.5 Payment Service

- **Chức năng:** Xử lý các giao dịch thanh toán.
- **APIs:** Thanh toán hóa đơn, quản lý phương thức thanh toán, phát hành hoá đơn, v.v.

#### 2.6 Notification Service

- **Chức năng:** Gửi thông báo đến người dùng.
- **APIs:** Gửi SMS, email, và thông báo qua app.

### 3. Dữ liệu và Lưu Trữ

Mỗi microservice sẽ có cơ sở dữ liệu riêng (database per service) để đảm bảo tính độc lập và dễ dàng mở rộng. Các loại cơ sở dữ liệu có thể bao gồm:

- **SQL Database** (ví dụ: PostgreSQL, MySQL) choviệc lưu trữ dữ liệu cấu trúc như thông tin người dùng, thông tin tài xế.
- **NoSQL Database** (ví dụ: MongoDB, Cassandra) cho dữ liệu không cấu trúc hoặc semi-cấu trúc, như thông tin chuyến đi, lịch sử đơn hàng.

### 4. Bảo mật

Bảo mật là một yếu tố quan trọng trong kiến trúc microservices:

- **Authentication:** Sử dụng OAuth2 và JWT để xác thực người dùng.
- **Authorization:** Áp dụng RBAC (Role-Based Access Control) để quản lý quyền truy cập theo vai trò người dùng.
- **Secure Communication:** Sử dụng HTTPS và TLS để đảm bảo an toàn cho dữ liệu truyền giữa các services.

### 5. DevOps và Môi trường Vận Hành

Việc triển khai và vận hành các microservices cần được tự động hóa để tối ưu hóa hiệu quả:

- **Containerization:** Sử dụng Docker để đóng gói ứng dụng và các phụ thuộc của nó.
- **Orchestration:** Sử dụng Kubernetes để quản lý và tự động hóa việc triển khai, mở rộng và quản lý các container.
- **CI/CD:** Thiết lập các pipeline Continuous Integration và Continuous Deployment để tự động hóa việc kiểm tra và triển khai code mới.
- **Monitoring and Logging:** Sử dụng Prometheus và Grafana cho monitoring; ELK Stack (Elasticsearch, Logstash, Kibana) hoặc Loki cho logging để theo dõi và phân tích các vấn đề.

### 6. Tính Năng Phục Hồi và Khả Năng Mở Rộng

- **Circuit Breaker:** Sử dụng mô hình Circuit Breaker để ngăn các lỗi trong một phần của hệ thống lan rộng.
- **Rate Limiting:** Áp dụng giới hạn tốc độ để bảo vệ các services khỏi lưu lượng quá tải.
- **Auto-scaling:** Thiết lập tự động mở rộng quy mô dựa trên nhu cầu thực tế sử dụng.

### 7. Kết luận

Việc thiết kế một kiến trúc microservices cho một ứng dụng đặt xe như Grab đòi hỏi sự cân nhắc kỹ lưỡng về việc phân chia chức năng, quản lý dữ liệu, bảo mật, và vận hành. Mỗi service cần được thiết kế để hoạt động độc lập nhưng vẫn có thể tương tác hiệu quả với các services khác. Điều này không chỉ giúp tăng cường khả năng mở rộng và bảo trì của ứng dụng mà còn cải thiện trải nghiệm người dùng cuối.


Khi triển khai một dự án lớn như ứng dụng đặt xe dựa trên kiến trúc microservices, việc quản lý mã nguồn thông qua các repositories riêng biệt cho mỗi microservice và các thành phần hỗ trợ là rất quan trọng. Dưới đây là danh sách các git repositories có thể được thiết lập cho các thành phần khác nhau của dự án:

### 1. Core Microservices

Mỗi microservice chính sẽ có một repository riêng để quản lý mã nguồn, phụ thuộc, và cấu hình đặc thù cho dịch vụ đó.

- **User Service**
  - `git-repo-url/user-service`
- **Driver Service**
  - `git-repo-url/driver-service`
- **Ride Matching Service**
  - `git-repo-url/ride-matching-service`
- **Trip Management Service**
  - `git-repo-url/trip-management-service`
- **Payment Service**
  - `git-repo-url/payment-service`
- **Notification Service**
  - `git-repo-url/notification-service`

### 2. Infrastructure and Configuration

Repositories dành cho việc quản lý cấu hình và hạ tầng của toàn bộ hệ thống.

- **API Gateway Configuration**
  - `git-repo-url/api-gateway`
- **Service Discovery Configuration**
  - `git-repo-url/service-discovery`
- **Central Configuration**
  - `git-repo-url/central-config`
- **Infrastructure as Code**
  - `git-repo-url/infrastructure`
    - Repository này chứa các kịch bản và mô tả cho việc triển khai hạ tầng sử dụng công cụ như Terraform hoặc AWS CloudFormation.

### 3. Common Libraries and Services

Repositories cho các thư viện chung và các microservices hỗ trợ nhỏ hơn.

- **Common Libraries**
  - `git-repo-url/common-libraries`
    - Chứa các thành phần tái sử dụng, như utilities và helper functions dùng chung cho nhiều services.
- **Authentication Service**
  - `git-repo-url/auth-service`
    - Dành cho quản lý xác thực và phân quyền.

### 4. DevOps and CI/CD

Repositories chứa các kịch bản và cấu hình cho DevOps và quy trình CI/CD.

- **CI/CD Configuration**
  - `git-repo-url/ci-cd-config`
    - Chứa các cấu hình cho Jenkins, GitLab CI/CD, hoặc các công cụ CI/CD khác.
- **Monitoring and Logging**
  - `git-repo-url/monitoring-logging`
    - Chứa cấu hình cho Prometheus, Grafana, ELK Stack, v.v.

### 5. Frontend

Nếu ứng dụng có các thành phần frontend (ví dụ như web app, mobile app), chúng cũng nên được quản lý trong repositories riêng.

- **Web Application**
  - `git-repo-url/web-app`
- **Mobile Application**
  - `git-repo-url/mobile-app`
    - Có thể chia thành hai repositories riêng biệt cho Android và iOS nếu cần.

### 6. Documentation and Miscellaneous

Repository cho tài liệu và các nội dung khác.

- **Documentation**
  - `git-repo-url/documentation`
    - Chứa tài liệu về dự án, thiết kế, API docs, v.v.

### 7. Data Science and Analytics (nếu có)

- **Analytics Service**
  - `git-repo-url/analytics-service`
    - Dành cho việc phân tích dữ liệu và báo cáo.

Địa chỉ `git-repo-url` trong các ví dụ trên phải được thay thế bằng URL thực tế của mỗi repository trên server Git của bạn (ví dụ như GitHub, GitLab, Bitbucket, v.v.). Việc tổ chức này giúp dễ dàng quản lý, phát triển, và triển khai từng phần của hệ thống một cách độc lập.

# CS / RS
Dưới đây là bảng cấu trúc chi phí (Cost Structure) và dòng doanh thu (Revenue Stream) của Uber đã được cập nhật với các ghi chú tương ứng:

### Cấu Trúc Chi Phí (Cost Structure)

| **Hạng Mục**                       | **Mô Tả**                                                                 | **Ghi chú** |
|------------------------------------|---------------------------------------------------------------------------|-------------|
| Chi Phí Nghiên Cứu và Phát Triển   | Chi phí liên quan đến việc cải tiến ứng dụng, phát triển công nghệ mới.  | TBD         |
| Chi Phí Marketing và Quảng Cáo     | Chi phí cho các chiến dịch quảng cáo, khuyến mãi, tiếp thị.              | TBD         |
| Chi Phí Nhân Sự                    | Lương và phúc lợi cho nhân viên, bao gồm cả tài xế và nhân viên văn phòng.| TBD         |
| Chi Phí Hạ Tầng                    | Chi phí duy trì và phát triển cơ sở hạ tầng công nghệ, máy chủ, trung tâm dữ liệu.| Không       |
| Chi Phí Pháp Lý và Tuân Thủ        | Chi phí liên quan đến việc tuân thủ các quy định pháp luật, bảo hiểm.    | Không       |
| Chi Phí Vận Hành                   | Chi phí liên quan đến hoạt động hàng ngày như hỗ trợ khách hàng, quản lý tài xế.| TBD         |
| Chi Phí Khấu Hao và Bảo Trì        | Chi phí liên quan đến khấu hao và bảo trì tài sản cố định.                | TBD         |

### Dòng Doanh Thu (Revenue Stream)

| **Nguồn Doanh Thu**                | **Mô Tả**                                                                 | **Ghi chú** |
|------------------------------------|---------------------------------------------------------------------------|-------------|
| Phí Chuyến Đi                      | Phí nhận từ mỗi chuyến đi, gồm phần trăm của Uber từ tổng giá trị chuyến đi.| TBD         |
| Phí Dịch Vụ Cao Cấp                | Doanh thu từ dịch vụ cao cấp như Uber Black, UberXL.                       | TBD         |
| Phí Hủy Chuyến                     | Phí phạt khi khách hàng hủy chuyến sau khi đã đặt.                         | TBD         |
| Phí Chờ                             | Phí tính thêm khi tài xế phải chờ đợi khách hàng lâu.                      | TBD         |
| Quảng Cáo Trên Ứng Dụng            | Doanh thu từ quảng cáo trên ứng dụng Uber.                                | TBD         |
| Quan Hệ Đối Tác và Liên Kết        | Doanh thu từ các chương trình hợp tác, liên kết với các công ty khác.     | TBD         |
| Dịch Vụ Giao Hàng                  | Doanh thu từ dịch vụ giao hàng như Uber Eats.                             | TBD         |
| Các Khoản Phụ Thu                  | Các khoản phụ thu khác như phí cầu đường, phí sân bay.                    | TBD         |

Hy vọng bảng này cung cấp cái nhìn tổng quan về cấu trúc chi phí và dòng doanh thu của Uber với các ghi chú cập nhật!
