# Sơ đồ chức năng hệ thống LMS AI

## 1. Sơ đồ kiến trúc tổng thể

```mermaid
flowchart TB
  U["Người dùng<br/>Học viên / Giảng viên / Quản trị viên"] --> FE["Web/App<br/>Đăng nhập, học tập, kiểm tra,<br/>AI Assistant, quản lý nội dung"]
  FE --> GW["API Gateway<br/>Điều hướng request, xác thực,<br/>phân quyền, ghi log, chuẩn hóa dữ liệu"]

  GW --> AUTH["Auth Service<br/>Đăng nhập, phân quyền,<br/>quản lý phiên"]
  GW --> CORE["LMS Core<br/>Modular Monolith<br/>(Lớp học, Nghiệp vụ, Quản lý dự án)"]
  GW --> AI["AI Services<br/>Microservices<br/>(RAG, Risk Score, AI PM Agent)"]

  CORE --> RDB["Relational Database<br/>Dữ liệu lõi & Quản lý dự án (Tasks)"]
  CORE --> NOSQL["NoSQL Database<br/>Log, trạng thái, dữ liệu linh hoạt"]

  AI --> VDB["Vector Database<br/>Tri thức, tài liệu, embedding"]
  AI --> LLM["LLM API / Local LLM<br/>RAG, phân tích, sinh nội dung, sinh task"]

  CORE <--> AI_SYNC["Đồng bộ chỉ số học tập,<br/>risk score, nhóm sinh viên,<br/>task dự án & trễ hạn"]
  AI <--> AI_SYNC

  CORE --> NOTI["Hệ thống thông báo tự động"]
  CORE --> ONE["Cổng dịch vụ 1 cửa"]
  CORE --> EXAM["Hệ thống Khảo thí chuyên biệt"]
  CORE --> LARK["Lark/Base<br/>Group hỗ trợ, bàn giao danh sách, báo trễ task"]
```

### 1.2 Các vấn đề tồn tại ở hệ thống cũ & Giải pháp cải tiến

Hệ thống quản lý đào tạo hiện tại đang gặp một số bất cập lớn về nghiệp vụ cốt lõi. LMS AI phiên bản mới giải quyết triệt để các vấn đề này thông qua định hướng kiến trúc như sau:

1. **Sinh viên thuộc các hệ học khác nhau học cùng một lớp vật lý:**
   - *Vấn đề:* Ràng buộc chương trình học và trọng số điểm theo Lớp (Class), dẫn đến việc sinh viên chất lượng cao, hệ chuẩn, hoặc học lại cùng lớp không thể áp dụng thang điểm khác nhau.
   - *Giải pháp:* Tách biệt thực thể **Lớp học (ClassSession)** khỏi **Chương trình học của Sinh viên (StudentCurriculum)**. Hệ thống tự động ánh xạ bộ trọng số điểm theo hệ đào tạo riêng của từng sinh viên.

2. **Lưu giữ lịch sử dữ liệu lớp học sau từng môn học:**
   - *Vấn đề:* Khi lớp chuyển sang môn học mới, dữ liệu môn cũ (bao gồm giảng viên phụ trách và các chỉ số chuyên cần, điểm số) bị ghi đè hoặc mất đi.
   - *Giải pháp:* Thiết kế bảng lịch sử `class_course_history` để lưu snapshot sau khi kết thúc môn học (Mã lớp, Mã môn, Giảng viên, Chuyên cần, Điểm trung bình môn).

3. **Sinh viên học lại và chuyển hệ học:**
   - *Vấn đề:* Khó kiểm soát lộ trình môn học bắt buộc khi sinh viên chuyển hệ hoặc đăng ký học lại do chương trình học của từng hệ bị cố định cứng.
   - *Giải pháp:* Thiết kế bảng quy đổi tương đương `course_equivalents` và quản lý lộ trình học thông qua `IndividualStudyPlan` (Chương trình học cá nhân hóa).

4. **Đồng nhất môn học khác ID giữa các khóa (Ví dụ: môn C của K24 và K25):**
   - *Vấn đề:* Môn C của K24 (ID: 101) và K25 (ID: 202) là hai dòng độc lập trong cơ sở dữ liệu. Khi sinh viên K24 học lại cùng lớp K25, hệ thống cũ không tự động tính điểm thay thế.
   - *Giải pháp:* Ánh xạ tất cả các môn học tương đương về một mã môn học gốc chung (`base_subject_id`). GPA engine sẽ gom nhóm theo mã gốc để thực hiện thay thế điểm học lại.

5. **Chưa áp dụng được các dịch vụ Trí tuệ nhân tạo (AI Services):**
   - *Vấn đề:* Quy trình giám sát học tập, phân loại rủi ro học tập của sinh viên và phân rã công việc cho dự án hoàn toàn làm thủ công, tốn nhiều thời gian của giảng viên và phòng CTSV.
   - *Giải pháp:* Tích hợp hệ thống AI Services (Microservices độc lập dùng GPU) chạy song song với LMS Core. Hệ thống tự động phân loại rủi ro học tập (Risk Score), tự động phân rã và theo dõi task của sinh viên (AI Project Agent) và cung cấp trợ lý học tập AI Assistant.

6. **Thủ tục hành chính rời rạc & Thiếu tự động hóa khảo thí (Thiếu mô hình một cửa):**
   - *Vấn đề:* Sinh viên phải liên hệ nhiều phòng ban khác nhau cho các nhu cầu hành chính, gây đùn đẩy trách nhiệm và chậm trễ. Quy trình thi cử, khảo thí và phúc khảo bài thi còn thủ công, tốn nhiều nhân lực của ban khảo thí và giảng viên.
   - *Giải pháp:* Ban hành **Cổng dịch vụ 1 cửa của RE (Rika)** trên LMS để tự động hóa/luật hóa dịch vụ. Tất cả sinh viên chỉ liên hệ làm việc với 01 nhân sự hỗ trợ duy nhất đặt tên chung là Rika (hỗ trợ bởi hệ thống tự động hóa). Đồng thời định nghĩa các vai trò mới (Giáo viên chủ nhiệm khuyến khích là Trợ giảng, Cố vấn phụ trách lớp dưới tên chung Rika) và cải tiến thi cử (thi trắc nghiệm & coding tự động, AI phúc khảo tự động bước 1, đồng bộ điểm tức thời).

## 2. Sơ đồ cây chức năng chi tiết

```mermaid
mindmap
  root((LMS AI))
    Tầng giao diện & điều hướng
      Web/App
        Đăng nhập
        Học khóa học
        Làm bài kiểm tra
        Tương tác AI Assistant
        Quản lý nội dung
      API Gateway
        Điều hướng request
        Xác thực với Auth Service
        Phân quyền
        Ghi log
        Chuẩn hóa dữ liệu
    LMS Core - Modular Monolith
      Hệ đào tạo
        Chuyên ngành đào tạo
        Kỳ học
      Môn học
      Lớp học
        Điểm danh
        Thời khóa biểu
        Quản lý nhóm
        CSSV
        Kiểm tra đầu giờ
        Tài nguyên sau buổi học
        Đơn xin nghỉ phép
        Kiểm tra BTVN
      Khảo thí
        Kết nối hệ thống khảo thí
        Thống kê điểm thi
        Chấm thi coding & trắc nghiệm tự động
        AI hỗ trợ phúc khảo tự động bước 1
      E-learning (Tùy biến học liệu)
        4 loại học liệu cốt lõi
          Video bài giảng
          Bài đọc tài liệu
          Slide trình chiếu
          Trắc nghiệm củng cố
        Tùy biến học liệu theo môn học
        Tối ưu băng thông & HLS Streaming
      Thông báo tự động
      Quản lý dự án
        Bảng Jira-style cho Sinh viên
        Bảng Jira-style cho Giảng viên
        Tích hợp chỉ số trễ task của AI
      Cổng dịch vụ 1 cửa (Rika)
        Tự động hóa nghiệp vụ hành chính
        Đầu mối liên hệ duy nhất Rika
        Vai trò Giáo viên chủ nhiệm (Trợ giảng)
        Vai trò Cố vấn phụ trách lớp
    AI Services - Microservices
      AI Project Agent
        Phân rã task tự động
        Đánh giá tiến độ dự án
        Gợi ý lịch trình giảng dạy
      AI chấm BTVN / Project
      AI phân tích chỉ số học tập
        RAG
        Local LLM thử nghiệm
        Phân tích theo từng buổi
      AI Risk Score realtime
        Giảm tương tác LMS
        Nghỉ học tăng
        Tụt điểm nhanh
        Trễ task dự án kéo dài
        Dynamic Re-Grouping
      AI Assistant
        Tra cứu hồ sơ
        Tóm tắt lịch sử học tập
        Tóm tắt hành vi
        Sinh kịch bản tư vấn
        Sinh nội dung nhắc nhở
        Gợi ý bài tập phụ đạo
      AI báo cáo & dự đoán
        Dashboard quản trị
        Báo cáo định kỳ
        Dự đoán fail môn
        Dự đoán bảo lưu
        Dự đoán PLO/CLO
      AI đánh giá giảng dạy
```

## 3. Sơ đồ triển khai theo giai đoạn

```mermaid
flowchart LR
  P1["Giai đoạn 1<br/>Nền tảng & nghiệp vụ lõi"] --> P2["Giai đoạn 2<br/>Tự động hóa CSSV"]
  P2 --> P3["Giai đoạn 3<br/>Tích hợp AI Services"]

  P1 --> P1A["API Gateway"]
  P1 --> P1B["Auth Service"]
  P1 --> P1C["Relational DB / NoSQL"]
  P1 --> P1D["Web/App cơ bản"]
  P1 --> P1E["Hệ đào tạo, môn học, lớp học"]
  P1 --> P1F["Điểm danh, TKB, quản lý nhóm"]
  P1 --> P1G["Cổng dịch vụ 1 cửa"]
  P1 --> P1H["Thông báo tự động"]
  P1 --> P1I["Thiết lập DB Schema Quản lý dự án"]

  P2 --> P2A["CSSV theo ngày"]
  P2A --> P2A1["Quét điểm danh hàng ngày"]
  P2A --> P2A2["Lọc sinh viên nghỉ không phép"]
  P2A --> P2A3["Cập nhật trạng thái nhanh"]
  P2A --> P2A4["Đồng bộ CTSV - Giảng viên"]

  P2 --> P2B["CSSV theo phân loại nhóm"]
  P2B --> P2B1["Tính R-point"]
  P2B --> P2B2["Kết hợp điểm môn trước"]
  P2B --> P2B3["Phân nhóm A/B/C/D"]
  P2B --> P2B4["Bàn giao danh sách"]
  P2B --> P2B5["Tạo group Lark/Base"]
  P2 --> P2C["Triển khai Bảng Kanban/Scrum độc lập"]

  P3 --> P3A["AI Microservice"]
  P3A --> P3A1["Kết nối Vector DB"]
  P3A --> P3A2["Kết nối LLM API"]
  P3 --> P3B["RAG / Local LLM"]
  P3 --> P3C["Risk Score realtime"]
  P3 --> P3D["Dynamic Re-Grouping"]
  P3 --> P3E["AI Assistant"]
  P3 --> P3F["AI báo cáo & dự đoán"]
  P3 --> P3G["AI chấm BTVN / Project"]
  P3 --> P3H["AI đánh giá giảng dạy"]
  P3 --> P3I["Tích hợp AI Project Agent (Sinh việc/Cảnh báo)"]
```

## 4. Sơ đồ luồng CSSV và phân loại nhóm

```mermaid
flowchart TD
  START["Kết thúc buổi học / kết thúc môn"] --> DAILY{"Xử lý theo ngày?"}

  DAILY -->|Có| ATT["Quét dữ liệu điểm danh"]
  ATT --> ABS["Lọc sinh viên nghỉ học không phép"]
  ABS --> STATUS["Cập nhật trạng thái nhanh"]
  STATUS --> SYNC["Đồng bộ CTSV - Giảng viên"]
  SYNC --> NOTIFY["Gửi thông báo / nhắc nhở"]

  DAILY -->|Khi có điểm cuối môn| SCORE["Thu thập R-point và điểm môn trước"]
  SCORE --> CLASSIFY{"Phân loại sinh viên"}

  CLASSIFY --> A["Nhóm A<br/>Ý thức tốt<br/>Học lực khá/giỏi"]
  CLASSIFY --> B["Nhóm B<br/>Ý thức kém<br/>Học lực khá/giỏi"]
  CLASSIFY --> C["Nhóm C<br/>Ý thức tốt<br/>Học lực TB/yếu"]
  CLASSIFY --> D["Nhóm D<br/>Ý thức kém<br/>Học lực TB/yếu"]

  A --> GV1["Giảng viên phụ trách"]
  GV1 --> A_ACT["Tạo group Lark/Base<br/>Khen thưởng, thúc đẩy phát triển"]

  B --> CTSV1["Phòng CTSV phụ trách"]
  CTSV1 --> B_ACT["Nhắc nhở quy chế<br/>Cải thiện ý thức kỷ luật"]

  C --> GV2["Giảng viên phụ trách"]
  GV2 --> C_ACT["Cố vấn 1-1<br/>Gợi ý bài tập bổ sung<br/>Phụ đạo riêng"]

  D --> CTSV2["Phòng CTSV phụ trách"]
  CTSV2 --> D_ACT["Can thiệp sâu<br/>Phối hợp phụ huynh<br/>Xử lý rủi ro sớm"]

  A_ACT --> GROUP["Tự động khởi tạo group hỗ trợ"]
  B_ACT --> GROUP
  C_ACT --> GROUP
  D_ACT --> GROUP

  GROUP --> TRACK["Theo dõi kết quả can thiệp"]
  TRACK --> AI_RISK["AI Risk Score realtime<br/>Cập nhật nguy cơ và đề xuất chuyển nhóm"]
  AI_RISK --> REGROUP{"Dynamic Re-Grouping?"}
  REGROUP -->|Có| CLASSIFY
  REGROUP -->|Không| REPORT["Báo cáo CSSV / Dashboard quản trị"]
```

## 5. Sơ đồ chức năng AI Services

```mermaid
flowchart TB
  DATA["Dữ liệu đầu vào<br/>Điểm danh, điểm thi, tương tác LMS,<br/>BTVN, project, tài liệu học tập, hồ sơ sinh viên"] --> PIPE["AI Data Pipeline"]

  PIPE --> EMB["Embedding / Indexing"]
  EMB --> VDB["Vector Database"]

  PIPE --> METRIC["Tính chỉ số học tập"]
  METRIC --> RISK["Risk Score realtime"]
  RISK --> REGROUP["Dynamic Re-Grouping"]
  REGROUP --> CORE["Cập nhật LMS Core<br/>Nhóm sinh viên, trạng thái CSSV,<br/>thông báo, dashboard"]

  VDB --> RAG["RAG Service"]
  RAG --> LLM["LLM API / Local LLM"]

  LLM --> ASSIST["AI Assistant"]
  ASSIST --> ASSIST_OUT["Tra cứu hồ sơ<br/>Tóm tắt lịch sử<br/>Sinh kịch bản tư vấn<br/>Gợi ý phụ đạo"]

  LLM --> REPORT["AI báo cáo & dự đoán"]
  REPORT --> REPORT_OUT["Báo cáo định kỳ<br/>Dự đoán fail môn<br/>Dự đoán bảo lưu<br/>Dự đoán PLO/CLO"]

  LLM --> GRADING["AI chấm BTVN / Project"]
  GRADING --> GRADING_OUT["Nhận xét bài làm<br/>Chấm điểm gợi ý<br/>Phát hiện vấn đề học tập"]

  LLM --> TEACHING["AI đánh giá giảng dạy"]
  TEACHING --> TEACHING_OUT["Phân tích chất lượng giảng dạy<br/>Gợi ý cải thiện"]

  LLM --> PM_AGENT["AI Project Agent"]
  PM_AGENT --> PM_AGENT_OUT["Sinh backlog & subtasks gợi ý<br/>Phát hiện thẻ trễ hạn<br/>Gợi ý lịch trình làm việc thầy cô"]
```

## 6. Luồng nghiệp vụ Quản lý Dự án (Jira Style)

### Luồng 1: AI tự động phân rã công việc (AI Task Generator)
```mermaid
sequenceDiagram
    actor SV as Sinh viên / Giảng viên
    participant FE as Web/App Client
    participant CORE as LMS Core Monolith
    participant AI as AI Project Agent
    participant LLM as LLM API

    SV->>FE: Nhập tên đề tài / mục tiêu công việc
    FE->>CORE: Lưu đề tài & yêu cầu phân rã
    CORE->>AI: Gửi yêu cầu sinh backlog
    AI->>LLM: Gửi prompt phân tích đề tài thành subtasks
    LLM-->>AI: Trả về danh sách tasks chi tiết (To Do, In Progress, Done)
    AI-->>CORE: Lưu danh sách tasks vào database lõi
    CORE-->>FE: Hiển thị danh sách task lên bảng Kanban
    FE-->>SV: Xem & phân vai (Assign) các công việc gợi ý
```

### Luồng 2: AI theo dõi tiến độ & cảnh báo rủi ro (AI Progress Tracker)
```mermaid
sequenceDiagram
    participant DB as Database lõi
    participant AI as AI Project Agent
    participant RISK as AI Risk Score Service
    participant NOTI as Hệ thống thông báo
    participant GV as Giảng viên

    loop Quét tự động định kỳ
        AI->>DB: Quét trạng thái tasks đang thực hiện
        DB-->>AI: Trả về danh sách tasks trễ hạn (overdue)
        alt Phát hiện task trễ hạn lâu ngày
            AI->>RISK: Gửi dữ liệu trễ task (MSSV, Số ngày trễ)
            RISK->>RISK: Tự động tính toán & tăng Risk Score (+15%)
            RISK->>DB: Lưu chỉ số rủi ro mới vào hồ sơ sinh viên
            AI->>NOTI: Kích hoạt thông báo cảnh báo trễ hạn
            NOTI->>GV: Gửi kịch bản/thư cảnh báo qua Lark/Base cho Giảng viên
        end
    end
```
