---
title: "SAP Load Testing: Cach tiep can Serverless voi AWS"
date: 2025-08-07
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

*by **Michele Donna** and **Francesco Bersani** | on **07 AUG 2025** | in* [Amazon Athena](https://aws.amazon.com/athena/), [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), [Amazon Managed Grafana](https://aws.amazon.com/grafana/), [Open Source](https://aws.amazon.com/blogs/awsforsap/category/open-source/), [SAP on AWS](https://aws.amazon.com/sap/), [Serverless](https://aws.amazon.com/serverless/)

## **Giới thiệu (Introduction)**

Việc thực hiện Load Testing đầy đủ cho hệ thống SAP là yếu tố lớn bảo đảm hệ thống đáp ứng được kỳ vọng về performance và reliability khi ở mức tải đỉnh. Những kịch bản thường cần Load Testing gồm: rollout công ty/quốc gia mới, nâng cấp phần mềm từ ECC lên S/4HANA, vá ứng dụng (ví dụ support packages), dự án chuyển đổi S/4HANA hoặc migrate sang SAP RISE.

Để vận hành ổn định sau các thay đổi quy mô lớn như vậy, khuyến nghị thực hiện load test trước khi cutover lên production để tránh các vấn đề liên quan đến performance. Với blog này, bạn sẽ học cách triển khai và sử dụng một nền tảng load test trên AWS để inject nhiều loại tải vào hệ thống SAP ERP có thể triển khai on-premise hoặc trên RISE.

### **SAP Load Testing là gì?**

Load Testing trong SAP là quá trình có hệ thống inject các loại tải lên hệ thống để đo hành vi dưới điều kiện tải nặng. Quy trình mô phỏng nhiều concurrent users truy cập đồng thời, thực thi các SAP transactions, xử lý lượng dữ liệu lớn, kiểm thử các business-critical processes đồng thời đo response times và resource utilization nhằm đánh giá hiệu năng đúng cách.

### **Tầm quan trọng trọng yếu của Load Testing**

Từ góc độ business continuity, Load Testing giúp tránh sập hệ thống vào giờ cao điểm, bảo đảm các quy trình chốt sổ cuối tháng/cuối năm chạy trơn tru, duy trì năng suất và sự hài lòng của người dùng.  
 Về risk mitigation, Load Testing nhận diện performance bottlenecks trước khi ảnh hưởng vận hành, giúp tránh downtime tốn kém và giảm rủi ro lỗi xử lý dữ liệu.  
 Về resource optimization, nó xác định yêu cầu phần cứng tối ưu, hỗ trợ capacity planning, tìm vùng cần performance tuning.  
 Về cost savings, Load Testing đúng cách giúp tránh over-provisioning, giảm chi phí bảo trì bất ngờ và tối thiểu gián đoạn kinh doanh.  
 Ngược lại, thiếu Load Testing dễ dẫn tới system failures trong kỳ kinh doanh quan trọng, mất doanh thu, giảm năng suất, ảnh hưởng danh tiếng và tăng chi phí bảo trì.

### **Các thách thức của Load Testing truyền thống**

Hệ sinh thái Load Testing đã thay đổi đáng kể những năm gần đây. Dù các bộ công cụ lâu đời (ví dụ bộ test automation của Tricentis—bao gồm LoadRunner) vẫn phổ biến trong hệ sinh thái SAP, cách tiếp cận truyền thống kéo theo nhiều yếu tố khó biện minh: chi phí license/hạ tầng/kỹ năng chuyên gia cao, triển khai vận hành phức tạp.

### **Load Testing kiểu Serverless với AWS (Serverless Load Testing with AWS)**

Cách tiếp cận hiện đại dịch chuyển sang kiến trúc Serverless với dịch vụ gốc AWS: [AWS Lambda](https://aws.amazon.com/lambda/), [Amazon EventBridge](https://aws.amazon.com/eventbridge/), [AWS Batch](https://aws.amazon.com/batch/), [AWS Fargate](https://aws.amazon.com/fargate/), [AWS Step Functions](https://aws.amazon.com/step-functions/), [AWS Systems Manager](https://aws.amazon.com/systems-manager/) và [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/).  
 Kết hợp Step Functions để orchestrate test scenarios và [Amazon S3](https://aws.amazon.com/s3/)/[Amazon DynamoDB](https://aws.amazon.com/dynamodb/) để lưu kết quả giúp tạo framework tự động có thể mô phỏng hàng nghìn concurrent users và cung cấp performance metrics chi tiết.

Lợi ích lớn của Serverless Load Testing gồm: pay-per-use (nhiều dịch vụ nằm trong AWS Free Tier), zero infrastructure maintenance, [khả năng automatic scaling](https://aws.amazon.com/blogs/awsforsap/using-aws-to-enable-sap-application-auto-scaling/) và giảm mạnh độ phức tạp khâu cài đặt/thực thi test.

---

### **Khi nào nên thực hiện Load Testing (When to Perform Load Tests)**

Kiểm thử tải (Load testing) trong SAP là một phần quan trọng của kiểm thử phi chức năng mà khách hàng SAP cần xem xét. Việc kiểm thử tải nên được thực hiện trước khi triển khai chính thức hệ thống SAP mới, sau các bản nâng cấp hoặc vá lỗi lớn, khi thêm quy trình nghiệp vụ mới, trước các giai đoạn cao điểm như kết thúc năm tài chính, và khi có kế hoạch mở rộng kinh doanh.

Các sự kiện kinh doanh được lên kế hoạch — như chốt sổ tài chính cuối tháng hoặc cuối năm — thường tạo ra tải hệ thống bất thường, đòi hỏi phải đánh giá hiệu năng chủ động để đảm bảo hoạt động diễn ra trơn tru. Bên cạnh đó, khi doanh nghiệp có kế hoạch tích hợp thêm đơn vị kinh doanh mới hoặc triển khai quy trình phức tạp hơn, việc kiểm thử tải trở nên thiết yếu nhằm xác minh khả năng của hệ thống trong việc xử lý khối lượng và độ phức tạp tăng thêm.

Để duy trì hiệu năng ổn định, các tổ chức nên tích hợp kiểm thử tải vào chiến lược Quản lý Thay đổi (Change Management). Cách tiếp cận có hệ thống này giúp phát hiện sớm tác động hiệu năng tiềm ẩn từ các thay đổi ứng dụng trước khi ảnh hưởng đến môi trường vận hành thật. Bằng cách tích hợp khả năng kiểm thử tự động, doanh nghiệp có thể xác thực hiệu năng hệ thống hiệu quả hơn, đồng thời xây dựng nền tảng vững chắc cho chất lượng trong các lần nâng cấp và tích hợp tương lai.

Cách tiếp cận chủ động này trong kiểm thử hiệu năng giúp đảm bảo độ tin cậy của hệ thống và trải nghiệm người dùng tối ưu trong toàn bộ môi trường SAP

---

## **Kiến trúc giải pháp**

### **SAP trên AWS (Native)**

[![Figure 1: SAP on AWS Native architecture](https://d2908q01vomqb2.cloudfront.net/17ba0791499db908433b80f37c5fbc89b870084b/2025/07/29/lt-Native-1024x309.jpg)](https://d2908q01vomqb2.cloudfront.net/17ba0791499db908433b80f37c5fbc89b870084b/2025/07/29/lt-Native.jpg)
Hình 1: SAP on AWS Native architecture

### RISE with SAP
[![Figure 2: RISE architecture](https://d2908q01vomqb2.cloudfront.net/17ba0791499db908433b80f37c5fbc89b870084b/2025/07/29/lt-RISE-1024x309.jpg)](https://d2908q01vomqb2.cloudfront.net/17ba0791499db908433b80f37c5fbc89b870084b/2025/07/29/lt-RISE.jpg)

Hình 2: RISE architecture

**Lưu ý:** Có thể cần **điều chỉnh code** để **trích xuất OS metrics** từ các hệ thống SAP chạy trong **RISE**. ([Amazon Web Services, Inc.](https://aws.amazon.com/blogs/awsforsap/sap-load-testing-a-serverless-approach-with-aws/))

---

## **Kịch bản kiểm thử (Testing Scenarios)**

### **Môi trường RISE**

### **Kiểm thử ứng dụng** 

[![Figure 3: RISE load test types](https://d2908q01vomqb2.cloudfront.net/17ba0791499db908433b80f37c5fbc89b870084b/2025/07/29/image-16.png)](https://d2908q01vomqb2.cloudfront.net/17ba0791499db908433b80f37c5fbc89b870084b/2025/07/29/image-16.png)

#### **HANA Database Load Testing**

[Giải pháp mã nguồn mở](http://c) này tạo workload database thực tế cho SAP HANA, inject các thao tác insert/update/delete quy mô lớn bằng k6-sql và khả năng scripting của k6. Ví dụ:

import sql from "k6/x/sql";

import driver from "k6/x/sql/driver/hdb";

import secrets from 'k6/secrets';

const db \= sql.open(driver, \`hdb://${username}:${password}@${hanaHost}:${hanaPort}\`);

export function setup() {

  db.exec(\`

  CREATE COLUMN TABLE test\_table (

    A INT GENERATED BY DEFAULT AS IDENTITY,

    B TEXT,

    C TEXT,

    D TEXT

    );

  \`);

}

export default function () {

  // first insert

  let insert\_result \= db.exec(\`

    INSERT INTO test\_table (B, C, D) 

    VALUES ('test', 'test', 'test');

  \`);

  console.log("Row inserted");

  // then select

  let selectResult \= db.query(\`

    SELECT \* FROM test\_table 

    LIMIT 10;

  \`);

  console.log(\`Read ${selectResult.length} rows\`);

}

export function teardown() {

   db.exec(\`DROP TABLE test\_table;\`);

   db.close();

 }

Khối monitoring (dựa trên HANA native và Amazon CloudWatch) tập trung vào transaction throughput và response times, thu thập mẫu tiêu thụ tài nguyên chi tiết trên toàn hệ thống; phân tích error rates và performance bottlenecks, cung cấp insight về memory/CPU impact.

#### **Load Testing cho SAP Fiori và IDoc**

Một framework dựa trên [k6](https://k6.io/) cho IDoc processing khối lượng lớn, bảo đảm khả năng trao đổi tài liệu nghiệp vụ mạnh mẽ. Hỗ trợ dynamic IDoc payload generation cho nhiều loại tài liệu (ví dụ sales orders — xem [code repository](https://github.com/aws-samples/sample-sap-load-testing-a-serverless-approach-with-aws/blob/main/assets/samples/IdocScenario/sample_idoc_ID1.xml) để lấy ví dụ), mô hình tải ramp-up, sustained và peak. Tích hợp sâu AWS CloudWatch để thu thập metrics và threshold-based validation.

Tương tự, có thể mô phỏng song song tương tác người dùng SAP Fiori để kiểm thử end-user experience khi hệ thống vào trạng thái tải nặng.

Monitoring tập trung vào hai trục: thứ nhất là Fiori access/IDoc processing performance — theo dõi throughput, processing times, hành vi queue dưới tải, mẫu lỗi và xác nhận end-to-end. Thứ hai là System impact analysis — đo SAP response times, database performance, network utilization và mẫu resource consumption tổng thể.

Ví dụ script **JavaScript** inject **sales order IDoc** hoặc mô phỏng tương tác Fiori:
```javascript
import http from 'k6/http';

import { check, sleep } from 'k6';

import { Rate } from 'k6/metrics';

import encoding from 'k6/encoding';

import secrets from 'k6/secrets';

//get information from secret

const username \= await secrets.get('username');

const password \= await secrets.get('password');

const sapClient \= await secrets.get('sapClient');

//get baseUrl from environment variable

const sapBaseUrl \= \_\_ENV.SAP\_BASE\_URL

//set the sap client in url parameter

let sapClientStringParameter=""

if (sapClient.match(/^\[0-9\]{3}$/)) {

    sapClientStringParameter=\`?sap-client=${sapClient}\`

}

// define your url path

const urlPath="/sap/bc/idoc\_xml"

//build the final url

const url \= \`${sapBaseUrl}${urlPath}${sapClientStringParameter}\`;

//start your load test logic

const xmlfile \= open('./sample\_idoc\_ID1.xml');

const todayDate \= new Date().toISOString().slice(0, 10);

const newDate \= todayDate.replace("-","")

export const successRate \= new Rate('success');

export const options \= {

  vus: 5,

  duration: '60s',

  insecureSkipTLSVerify: true,

};

export default function () {

    const data \= getAndConvertIdocXml();

    const credentials \= \`${username}:${password}\`;

    const encodedCredentials \= encoding.b64encode(credentials);

    const httpOptions \= {

        headers: {

          Authorization: \`Basic ${encodedCredentials}\`,

          "Content-Type": 'text/xml'

        },

      };

  check(http.post(url, data, httpOptions), {

    'status is 200': (r) \=\> r.status \== 200,

  }) || successRate.add(1);

  sleep(5);

}

function getAndConvertIdocXml() {

    let result \= xmlfile.replace("{{GENERATED\_IDOC\_NUMBER}}", Math.floor(Math.random() \* 100000000000000))

    result  \= result.replace("{{GENERATED\_MESSAGE\_ID}}", Math.floor(Math.random() \* 100000000000000))

    return result

  }
```
Một cấu trúc XML IDoc mẫu có thể được tạo thông qua giao dịch WE19 (công cụ kiểm thử IDoc). Sau khi IDoc được tạo, tệp XML cần được chuyển đổi thành mẫu (template) với các chuỗi giữ chỗ (string placeholder) `GENERATED_IDOC_NUMBER`, `GENERATED_MESSAGE_ID` để được điền tại thời điểm chạy (runtime) khi kịch bản kiểm thử tải được thực thi.

Lưu ý: hãy đảm bảo bạn đã thực hiện [cấu hình ALE](https://help.sap.com/docs/SUPPORT_CONTENT/abap/3353526372.html) chính xác — chẳng hạn như sender/receiver và partner number/port được đặt đúng giá trị phù hợp với hệ thống SAP của bạn — để xử lý IDoc inbound thành công với [status 53](https://help.sap.com/docs/SUPPORT_CONTENT/abap/3353525129.html) (Application document posted).

Cả hai phương pháp kiểm thử đều mang lại lợi ích đáng kể nhờ mô phỏng quy trình nghiệp vụ thực tế và xây dựng được các kịch bản kiểm thử có khả năng mở rộng. Hệ thống giám sát toàn diện, kết hợp với các công cụ mã nguồn mở tiết kiệm chi phí, giúp tích hợp sâu với các dịch vụ AWS nhằm tăng cường khả năng quan sát và kiểm soát. 

---

## **Môi trường SAP trên AWS (Native Environment)**

Ngoài các tùy chọn môi trường RISE, các bài kiểm thử sau đây có thể được thực hiện khi vận hành SAP trực tiếp (natively) trên AWS.

### **Kiểm thử hạ tầng (Infrastructure Testing)**

[![Figure 4: infrastructure load test types](https://d2908q01vomqb2.cloudfront.net/17ba0791499db908433b80f37c5fbc89b870084b/2025/07/29/image-1-27-1024x515.png)](https://d2908q01vomqb2.cloudfront.net/17ba0791499db908433b80f37c5fbc89b870084b/2025/07/29/image-1-27.png)

Hình 4: infrastructure load test types

Hình mô tả 3 kịch bản chính tập trung vào các khía cạnh hạ tầng khác nhau:

### **Compute Testing Scenario**

Kịch bản này đánh giá khía cạnh tính toán của hệ thống SAP thông qua stress testing có hệ thống bằng các công cụ ở cấp hệ điều hành. Để mô phỏng tải CPU, tương tự AWS Fault Injection Simulator (FIS), có thể dùng các công cụ như stress-ng để tạo các mẫu sử dụng CPU chính xác. Ví dụ: dùng stress-ng để duy trì mức tải CPU 75% trên mọi lõi, hoặc thực hiện memory stress testing để tiêu thụ 80% RAM khả dụng. Có hướng dẫn chi tiết trong phần tutorial.

Kết hợp các công cụ này với chỉ số hiệu năng gốc của SAP (ví dụ thu thập bởi sapsocol hoặc Workload Monitor) sẽ cung cấp cái nhìn toàn diện khi so sánh instance type giữa Intel và AMD, từ đó xác định cấu hình tính toán tối ưu cho từng SAP workload cụ thể. Phương pháp kiểm thử bao gồm: tăng tải dần (gradual load increments), duy trì giai đoạn sử dụng cao (sustained high utilization), và theo dõi hành vi hệ thống dưới các điều kiện computational stress khác nhau.

### **Storage Testing Scenario**

Kịch bản này tập trung tối ưu hiệu năng lưu trữ thông qua bộ kiểm thử toàn diện Flexible I/O (FIO). FIO cho phép đo đạc chính xác đặc tính hiệu năng lưu trữ qua nhiều mô hình kiểm thử như Random Read/Write Testing, Sequential Read Performance và IOPS Testing cho EBS volumes. Để biết thêm chi tiết về FIO benchmarking trên AWS, tham khảo liên kết hướng dẫn.

Các bài FIO được chạy trên nhiều loại EBS volume (ví dụ gp3 so với io2) để phân tích:

* Khả năng thông lượng tối đa (maximum throughput capabilities)

* Hiệu năng IOPS dưới các loại workload khác nhau

* Mẫu độ trễ (latency patterns) tại các queue depth khác nhau

* Hành vi của hệ thống lưu trữ dưới sustained load

### **Networking Testing Scenario**

Kịch bản này kiểm tra hiệu năng mạng và khả năng kết nối bằng cách đo các tham số mạng quan trọng ảnh hưởng tới hiệu năng hệ thống SAP. Tính năng trọng yếu là mô phỏng độ trễ mạng (network delay) bằng các lệnh hệ điều hành như tc (traffic control) trên Linux. Cách này cho phép kiểm soát chính xác điều kiện mạng bằng cách tạo độ trễ nhân tạo, packet loss và giới hạn băng thông.

Ví dụ, dùng các lệnh như tc qdisc có thể mô phỏng các điều kiện mạng thực tế, bao gồm:

* Tạo độ trễ cụ thể (ví dụ 300ms delay)

* Mô phỏng packet loss (ví dụ 5% packet loss)

* Tạo điều kiện bandwidth throttling (giới hạn băng thông)

* Áp dụng jitter trong truyền thông mạng

Những mô phỏng suy giảm chất lượng mạng này giúp cung cấp insight giá trị về cách hệ thống SAP ứng xử dưới nhiều điều kiện mạng khác nhau. Cách tiếp cận này hỗ trợ tổ chức đánh giá độ bền vững (resilience) của hệ thống SAP trước các vấn đề mạng và tối ưu cấu hình mạng tương ứng. Kịch bản này cũng có thể dùng để bổ sung độ trễ giữa các hệ thống SAP liên kết (ví dụ ERP ↔ BW) và đo lường quy trình data extraction, bằng cách mô phỏng một dự án migration trong đó ERP chạy on-premise còn BW được migrate lên AWS.

Những mô phỏng suy giảm chất lượng mạng này giúp cung cấp insight giá trị về cách hệ thống SAP **ứng xử dưới nhiều điều kiện mạng** khác nhau. Cách tiếp cận này hỗ trợ tổ chức đánh giá **độ bền vững (resilience)** của hệ thống SAP trước các vấn đề mạng và tối ưu cấu hình mạng tương ứng.  
 Kịch bản này cũng có thể dùng để **bổ sung độ trễ** giữa các hệ thống SAP liên kết (ví dụ **ERP ↔ BW**) và đo lường quy trình **data extraction**, bằng cách mô phỏng một dự án **migration** trong đó **ERP chạy on-premise** còn **BW được migrate lên AWS**.

---

## **Giám sát Load Test**

### **Tùy chọn giám sát thời gian thực (Real-time Monitoring Options)**

#### **CloudWatch Dashboards**

Các bảng điều khiển tùy chỉnh của Amazon CloudWatch cung cấp cái nhìn hợp nhất về các chỉ số hiệu năng quan trọng, kết hợp dữ liệu hạ tầng, dữ liệu hiệu năng ứng dụng và các chỉ số kiểm thử tùy chỉnh thành các biểu đồ trực quan toàn diện. Các chỉ số hiệu năng chính được tổ chức thành các nhóm logic: SAP technical metrics như Dialog và Database Response Time, số lượng system dumps, số lượng người dùng đang hoạt động dành cho nhóm vận hành SAP, và OS metrics như CPU, mức sử dụng bộ nhớ, IOPS lưu trữ và thông lượng dành cho nhóm vận hành hạ tầng. Các bảng điều khiển này có khả năng cảnh báo tự động dựa trên các ngưỡng được định sẵn, cho phép phản ứng chủ động với các vấn đề hiệu năng trong quá trình kiểm thử. Việc lưu trữ dữ liệu lịch sử giúp phân tích xu hướng và so sánh hiệu năng giữa các lần kiểm thử khác nhau, mang lại những hiểu biết giá trị cho việc lập kế hoạch dung lượng và tối ưu hóa hệ thống..

[![Figure 5: CloudWatch dashboard metrics during load tests](https://d2908q01vomqb2.cloudfront.net/17ba0791499db908433b80f37c5fbc89b870084b/2025/08/05/Screenshot-2025-08-05-at-15.01.21-1024x465.png)](https://d2908q01vomqb2.cloudfront.net/17ba0791499db908433b80f37c5fbc89b870084b/2025/08/05/Screenshot-2025-08-05-at-15.01.21.png)

#### **Amazon Managed Grafana (tùy chọn)**

Là một lựa chọn thay thế hoặc bổ trợ cho CloudWatch dashboards, Amazon Managed Grafana mang đến khả năng trực quan hóa nâng cao và các tính năng phân tích chuyên sâu hơn. Dịch vụ này cải thiện trải nghiệm giám sát thông qua khả năng tương quan dữ liệu nâng cao và các chỉ số tùy chỉnh, cung cấp nhiều tùy chọn hiển thị phong phú với cả bảng điều khiển dựng sẵn và bảng điều khiển tùy chỉnh.

Amazon Managed Grafana hỗ trợ việc tổng hợp chỉ số xuyên tài khoản và xuyên vùng (cross-account, cross-region), cho phép giám sát toàn diện các môi trường SAP phức tạp. Cơ chế kiểm soát truy cập theo nhóm và chia sẻ bảng điều khiển giúp tăng cường khả năng cộng tác giữa các nhóm liên quan. Ngoài ra, tích hợp gốc với các dịch vụ AWS (trong kịch bản này là Amazon CloudWatch) và các nguồn dữ liệu bên ngoài giúp mở rộng năng lực giám sát.

Khả năng khám phá chỉ số theo thời gian thực (real-time metric exploration) và phân tích tức thời (ad-hoc analysis) hỗ trợ việc điều tra nhanh các vấn đề hiệu năng, được bổ sung bằng khả năng triển khai bảng điều khiển tự động thông qua hạ tầng dưới dạng mã (infrastructure as code). Các kênh cảnh báo và thông báo tích hợp sẵn giúp đảm bảo phản hồi kịp thời với các bất thường về hiệu năng.

Sự kết hợp giữa CloudWatch metrics và các biểu đồ trực quan của Grafana tạo nên một hệ sinh thái giám sát mạnh mẽ, hỗ trợ cả giám sát vận hành thời gian thực lẫn phân tích hiệu năng dài hạn, từ đó giúp đưa ra các quyết định tối ưu hóa hệ thống SAP dựa trên dữ liệu..

[![Figure 6: Grafana dashboard metrics during load tests](https://d2908q01vomqb2.cloudfront.net/17ba0791499db908433b80f37c5fbc89b870084b/2025/07/29/image-3-18-1024x473.png)](https://d2908q01vomqb2.cloudfront.net/17ba0791499db908433b80f37c5fbc89b870084b/2025/07/29/image-3-18.png)
Hình 6: Grafana dashboard metrics trong Load Testing  
Trong cả hai trường hợp, SAP NetWeaver specific metrics có thể thu thập vào CloudWatch bằng open-source solution được liên kết trong bài.

---

## **Quy trình Load Testing (Load Testing Workflow)**

### **Luồng Orchestration và thực thi (Orchestration and Execution Flow)**

Quy trình này tuân theo một chuỗi sự kiện có cấu trúc giữa các dịch vụ AWS để thực hiện và giám sát các bài kiểm thử tải của hệ thống SAP:

* Test Initiation: Một Step Function khởi tạo quy trình kiểm thử dựa trên dữ liệu đầu vào từ người dùng trong giao diện web chuyên dụng, điều phối toàn bộ quá trình thông qua state machine để kiểm soát việc thực thi và giám sát kiểm thử. Payload đầu vào xác định kịch bản nào sẽ được chạy (SAP, HANA hoặc Infrastructure).

* Test Execution: Các hàm Lambda chuyên biệt, xác thực thông qua AWS Secrets Manager, thực thi các kịch bản kiểm thử tải. Các hàm này chứa logic và các tham số cụ thể cho từng loại kịch bản kiểm thử.

* System Access: AWS Systems Manager và Fargate cung cấp quyền truy cập an toàn vào môi trường SAP trong VPC, cho phép thực hiện kiểm thử trên:  
  * SAP Application Servers cho kiểm thử ở cấp ứng dụng  
  * SAP HANA database cho kiểm thử ở cấp cơ sở dữ liệu  
  * Cấp hệ điều hành để kiểm thử CPU, bộ nhớ, lưu trữ và mạng

* Monitoring and Data Collection:

  * Amazon CloudWatch thu thập và xử lý các chỉ số hiệu năng từ quá trình kiểm thử, tập hợp dữ liệu từ toàn bộ các thành phần SAP và hạ tầng liên quan.

* Realtime Visualization Options:

  * CloudWatch dashboards cho việc giám sát thời gian thực  
  * Amazon Managed Grafana (tùy chọn) cho khả năng trực quan hóa và phân tích nâng cao, có thể tích hợp với AWS IAM Identity Center (kế thừa AWS Single Sign-On) để xác thực người dùng

* Long Term Data Processing & Analysis:  
  * Các chỉ số hiệu năng được lưu trữ trong Amazon S3  
  * AWS Glue xử lý và chuyển đổi dữ liệu  
  * Amazon Athena cho phép phân tích kết quả dựa trên truy vấn SQL  
  * Amazon QuickSight (tùy chọn) tạo biểu đồ và phân tích trực quan hiệu năng

Để cải thiện và đơn giản hóa trải nghiệm người dùng cuối, một ứng dụng React đã được phát triển, trong đó người dùng được xác thực an toàn qua [Amazon Cognito](https://aws.amazon.com/it/cognito/) có thể trực tiếp khởi tạo kiểm thử tải và theo dõi trạng thái. Ứng dụng này được lưu trữ trên [Amazon CloudFront](https://aws.amazon.com/it/cloudfront/) và tương tác với các quy trình Step Function thông qua [Amazon API Gateway](https://aws.amazon.com/api-gateway/).

Quy trình này đảm bảo việc thực thi, giám sát và phân tích kiểm thử một cách toàn diện, đồng thời duy trì bảo mật nhờ cô lập trong VPC và kiểm soát quyền truy cập chặt chẽ.

---

## **Triển khai (Implementation)**

Bắt đầu hôm nay:

* Clone GitHub [repository](https://github.com/aws-samples/sample-sap-load-testing-a-serverless-approach-with-aws)

* Làm theo step-by-step deployment guide

* Chạy load test đầu tiên trên hệ thống SAP của bạn — từ setup tới kết quả — trong \< 60 phút\!

---

## **Chi phí (Costs)**

| Service | Cost | Description |
| ----- | ----- | ----- |
| Step Functions | Free Tier | Bao gồm 4.000 lần chuyển trạng thái miễn phí mỗi tháng |
| Lambda | Free Tier | 1 triệu yêu cầu miễn phí mỗi tháng và 400.000 GB-seconds thời gian xử lý mỗi tháng |
| Secrets Manager | $0.40 per secret per month | 1 secret được sử dụng để lưu trữ thông tin xác thực và tham số người dùng |
| Systems Manager | Free | Không tính phí thêm cho lệnh run command |
| Cloudwatch | $3.00 | Bao gồm Basic Monitoring Metrics \+ 10 Custom Metrics |
| Cloudfront | $0.12 per month |  |
| Cognito | $6.22 per month | 5 người dùng, 100 token request mỗi người dùng và 1 app client |
| ECR | $0.07 per month | 2 images, tổng dung lượng 750 MB mỗi tháng |
| Fargate | $6.32 per month | 16 vCPUs, 16 GB memory, 10 tác vụ mỗi pod, thời lượng 60 phút |
| API Gateway | $0.35 per month | 10.000 yêu cầu REST API mỗi tháng |
| S3 | Free Tier | 5 GB dung lượng lưu trữ S3 Standard, 20.000 GET, 2.000 PUT/COPY/POST/LIST requests, 100 GB Data Transfer Out mỗi tháng |
| Glue | Free Tier | Miễn phí cho 1 triệu đối tượng đầu tiên được lưu |
| Athena | $0.29 per month | 20 truy vấn mỗi ngày, 100 MB dữ liệu quét mỗi truy vấn |
| Managed Grafana | $9.00 per month (optional) | 1 editor, 5 users |
| Quicksight | $33 per month (optional) | 5 users |
| Tổng chi phí ước tính: | $16.77/tháng *không bao gồm* Grafana/Quicksight $25.77/tháng *với* Quicksight $58.77/tháng *với cả* Grafana và Quicksight | Nếu chỉ cần báo cáo, có thể sử dụng Quicksight. CloudWatch dashboards cũng có thể được dùng thay thế cho Grafana (với một số giới hạn). |

Thông tin chi tiết về chi phí có thể được tìm thấy tại [đây](https://calculator.aws/#/estimate?id=fab0d8351f395f84210206c6f94dac68f2b4b332).

---

## **Kết luận**

Bài viết trình bày cách tận dụng AWS Serverless services để thực hiện SAP load & performance testing. Bằng việc áp dụng cloud-native services, ta có một cách tiếp cận hiệu quả & tiết kiệm chi phí để thực hiện load tests quy mô lớn cho SAP, đồng thời thu thập metrics sâu sắc, hành động được. Cách tiếp cận này cho thấy khả năng mở rộng, chi phí tối ưu và ra quyết định dựa trên dữ liệu cho triển khai SAP.  
Xem thêm tại [AWS for SAP blogs](https://aws.amazon.com/blogs/awsforsap/); bắt đầu chiến lược Load Testing cho SAP landscape của bạn ngay hôm nay\!

---

## **Tham gia thảo luận SAP on AWS**

Ngoài customer account team và AWS Support, AWS cung cấp diễn đàn Q\&A công khai trên [re:Post](https://aws.amazon.com/blogs/aws/aws-repost-a-reimagined-qa-experience-for-the-aws-community/); [AWS for SAP](https://www.repost.aws/topics/TAujMzs1_ZR3SIaMwia7ln7A/sap-on-aws) Solution Architecture team thường xuyên theo dõi chủ đề AWS for SAP để hỗ trợ. Nếu câu hỏi không liên quan hỗ trợ, hãy tham gia re:Post để đóng góp cho cộng đồng.  