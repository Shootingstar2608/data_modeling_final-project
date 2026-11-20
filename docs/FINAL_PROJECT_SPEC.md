# FINAL PROJECT
## Database & Data Modeling
### Thiết kế cơ sở dữ liệu quản lý chương trình cố vấn học tập

**Hình thức:** Cá nhân · **Thời lượng:** 8–12 tuần · **Sản phẩm chính:** Mô hình dữ liệu, cơ sở dữ liệu quan hệ và bộ SQL có thể chạy lại.

## 1. Bối cảnh và mục tiêu

Một đơn vị tổ chức nhiều chương trình cố vấn học tập. Trong từng chương trình, người học (mentee) được ghép với người cố vấn (mentor), đặt mục tiêu, tham gia các buổi gặp và gửi phản hồi. Đơn vị cần quản lý được cả hoạt động hiện tại lẫn lịch sử thay đổi mentor, kết quả buổi gặp và tiến độ mục tiêu.

Bạn có nhiệm vụ phân tích nghiệp vụ, thiết kế mô hình dữ liệu và triển khai cơ sở dữ liệu quan hệ cho hệ thống này. Sau project, bạn cần chứng minh được khả năng:

- Chuyển yêu cầu nghiệp vụ thành thực thể, thuộc tính, quan hệ và ràng buộc.
- Thiết kế mô hình EER, ánh xạ sang lược đồ quan hệ và giải thích các lựa chọn.
- Phân tích khóa, phụ thuộc hàm và chuẩn hóa đến 3NF.
- Dùng SQL để bảo vệ tính toàn vẹn, thực hiện thao tác dữ liệu và trả lời câu hỏi vận hành.
- Kiểm thử thiết kế bằng cả dữ liệu hợp lệ và các thao tác phải bị từ chối.

**Phạm vi:** Chỉ xây dựng tầng dữ liệu. Không yêu cầu giao diện, API, đăng nhập, phân quyền tài khoản, gửi thông báo hoặc triển khai dịch vụ trực tuyến. Vai trò mentor/mentee là dữ liệu nghiệp vụ, không đồng nghĩa với tài khoản đăng nhập hay quyền của DBMS.

## 2. Phạm vi nghiệp vụ bắt buộc

Các quy tắc dưới đây là hợp đồng chung để thiết kế và chấm bài. Bạn được tự chọn tên bảng, khóa, kiểu dữ liệu và cách phân rã bảng; không được thay đổi ý nghĩa của quy tắc bắt buộc. Các khái niệm trong đề **không phải danh sách bảng cho sẵn**.

### 2.1. Người tham gia và chương trình

| Mã | Quy tắc |
| --- | --- |
| BR01 | Mỗi người có định danh riêng, họ tên và email liên hệ. Email bắt buộc, không trùng theo quy ước chuẩn hóa bạn công bố; không yêu cầu kiểm tra địa chỉ có thật. |
| BR02 | Mỗi chương trình có định danh, tên, ngày bắt đầu và ngày kết thúc. Ngày kết thúc không trước ngày bắt đầu. |
| BR03 | Một người có thể tham gia nhiều chương trình. Vai trò được xác định **trong từng chương trình**: mentor, mentee hoặc cả hai. Một người không được làm mentor của chính mình. |
| BR04 | Mỗi lần ghép cặp thuộc đúng một chương trình, có đúng một mentor và một mentee. Hai người phải tham gia chương trình đó với vai trò tương ứng. |

### 2.2. Ghép cặp và lịch sử

| Mã | Quy tắc |
| --- | --- |
| BR05 | Một mentor có thể phụ trách nhiều mentee. Trong cùng chương trình, một mentee có nhiều lần ghép cặp theo lịch sử nhưng không được có hai khoảng ghép cặp chồng lấn. Việc tham gia các chương trình khác nhau được xét độc lập. |
| BR06 | Mỗi lần ghép cặp có thời điểm bắt đầu và kết thúc dự kiến hoặc thực tế. Kết thúc có thể để trống khi chưa xác định; nếu có thì phải sau bắt đầu. Toàn bộ khoảng phải nằm trong thời gian chương trình. |
| BR07 | Đổi mentor phải tạo lần ghép cặp mới, giữ lại lần cũ và các buổi gặp đã gắn với lần cũ. Không ghi đè mentor trên một lần ghép cặp đã có buổi gặp. |

**Quy ước thời gian dùng chung:** Chọn và công bố một múi giờ. Khoảng ghép cặp dùng dạng nửa mở `[bắt đầu, kết thúc)`: một lần có thể kết thúc đúng lúc lần tiếp theo bắt đầu. Kết thúc để trống được hiểu là kéo dài đến hết chương trình. Với chương trình lưu theo ngày, khoảng hiệu lực bắt đầu lúc 00:00 ngày đầu và kết thúc lúc 00:00 ngày liền sau ngày cuối. “Đang được ghép cặp tại T” nghĩa là T nằm trong khoảng ghép cặp và trong thời gian chương trình; không suy ra chỉ từ việc ngày kết thúc để trống.

### 2.3. Mục tiêu, buổi gặp và phản hồi

| Mã | Quy tắc |
| --- | --- |
| BR08 | Mỗi mục tiêu thuộc một mentee trong một chương trình, có nội dung, ngày tạo và trạng thái: `not_started`, `in_progress`, `completed` hoặc `dropped`. Mục tiêu được giữ lại khi mentee đổi mentor trong cùng chương trình. |
| BR09 | Mỗi buổi gặp thuộc đúng một lần ghép cặp. Có thời điểm bắt đầu, kết thúc và trạng thái: `scheduled`, `completed`, `cancelled` hoặc `no_show`. Kết thúc phải sau bắt đầu; toàn bộ khoảng buổi gặp phải nằm trong khoảng ghép cặp và chương trình. Buổi bị hủy vẫn giữ thông tin lịch đã đặt. |
| BR10 | `scheduled` là đã lên lịch nhưng chưa chốt kết quả; `completed` là đã diễn ra; `cancelled` là đã hủy; `no_show` là buổi không diễn ra do ít nhất một bên vắng mặt. Buổi `completed` phải có ghi nhận nội dung/kết quả; `cancelled` và `no_show` phải có lý do. |
| BR11 | Một buổi gặp có thể thảo luận nhiều mục tiêu; một mục tiêu có thể được thảo luận trong nhiều buổi. Chỉ ghi nhận việc thảo luận cho buổi `completed`. Mục tiêu phải thuộc đúng mentee và chương trình của buổi gặp; thời điểm tạo mục tiêu không sau thời điểm bắt đầu buổi. Không gắn trùng cùng một mục tiêu vào cùng một buổi. |
| BR12 | Chỉ mentee của buổi gặp được gửi phản hồi, tối đa một phản hồi cho mỗi buổi `completed`. Điểm là số nguyên từ 1 đến 5, nội dung nhận xét có thể trống. Thời điểm gửi không trước khi buổi gặp kết thúc. Buổi hoàn thành chưa có phản hồi vẫn là dữ liệu hợp lệ. |
| BR13 | Xóa hoặc sửa dữ liệu không được tạo bản ghi mồ côi hay làm sai các quy tắc trên. Nếu buổi đã có mục tiêu được thảo luận hoặc phản hồi, không được chuyển khỏi `completed` hay xóa buổi đó. Không xóa dây chuyền lịch sử ghép cặp, buổi gặp hoặc phản hồi chỉ để thao tác xóa thành công. |

**Những lựa chọn bạn phải tự quyết định và ghi trong DESIGN.md:**

- Thông tin riêng nào cần quản lý cho mentor và mentee; thuộc tính nào bắt buộc hoặc được để trống.
- Cách quản lý kỹ năng/chuyên môn hoặc dữ liệu đa trị khác.
- Một quan hệ phân cấp/đệ quy có ý nghĩa nghiệp vụ; quy tắc chống tự tham chiếu và chu trình nếu quan hệ là cây.
- Trường hợp nào được sửa lịch, sửa kết quả, sửa phản hồi hoặc xóa dữ liệu chưa phát sinh lịch sử; cách phản hồi khi thao tác không hợp lệ.
- Quy tắc chuyển trạng thái ngoài các điều kiện đã chốt ở BR10–BR13. Không bắt buộc tự chuyển trạng thái theo đồng hồ hay lưu toàn bộ lịch sử trạng thái.

Viết các quyết định này thành quy tắc có mã riêng, có ví dụ hợp lệ và không hợp lệ. Quy tắc bổ sung không được mâu thuẫn với BR01–BR13. Không mặc định có yêu cầu giới hạn số mentee, chống trùng lịch mentor hay tự động ghép cặp nếu bạn chưa chọn bổ sung.

## 3. Yêu cầu thiết kế dữ liệu

### 3.1. Mô hình EER và lược đồ quan hệ

Nộp **hai mức biểu diễn phân biệt được**: mô hình khái niệm EER và lược đồ quan hệ sau ánh xạ. Có thể đặt trong cùng tài liệu nếu phân biệt rõ. Sơ đồ phải vẽ bằng công cụ, đọc được khi mở ở kích thước thông thường; không nộp bản vẽ tay hoặc ảnh chụp bản vẽ tay.

Mô hình khái niệm cần thể hiện thực thể, thuộc tính, định danh, bội số tối thiểu–tối đa và mức tham gia bắt buộc/tùy chọn. Lược đồ quan hệ cần thể hiện bảng, thuộc tính, PK, FK, khóa ứng viên/UNIQUE và khả năng nhận NULL; bổ sung kiểu dữ liệu trong từ điển dữ liệu nếu sơ đồ không hiển thị được.

Các nội dung EER bắt buộc thực hành:

| Nội dung | Bằng chứng cần có |
| --- | --- |
| Thực thể thông thường và ít nhất một thực thể yếu | Giải thích sự phụ thuộc định danh, chủ sở hữu và khóa bộ phận của thực thể yếu. Chỉ phụ thuộc tồn tại hoặc có FK chưa đủ để gọi là thực thể yếu. |
| Ít nhất một cấu trúc cha–con | Nêu ý nghĩa nghiệp vụ, điều kiện thuộc lớp con, rời nhau/chồng lấn và toàn phần/bộ phận. Cách biểu diễn phải phù hợp việc vai trò thay đổi theo chương trình. |
| Thuộc tính đơn, tổ hợp, đa trị và dẫn xuất | Chỉ rõ mỗi loại trên mô hình và cách ánh xạ. Không lưu danh sách giá trị bằng chuỗi phân cách chỉ để tránh tạo quan hệ phù hợp. |
| Quan hệ 1:N và M:N | Thể hiện bội số, mức tham gia, cách đặt FK hoặc tạo quan hệ liên kết. |
| Ít nhất một quan hệ đệ quy | Giải thích vai trò hai phía, bội số và ràng buộc cần có. |

Quan hệ 1:1, kiểu Union/category và thuộc tính phức hợp ngoài các loại trên là **tùy chọn**. Nếu không có 1:1 trong mô hình, giải thích nguyên tắc ánh xạ 1:1 bằng một ví dụ giả định ngắn, tách khỏi schema triển khai. Không phải tạo bảng mới cho mọi quan hệ 1:1 hoặc 1:N; lựa chọn ánh xạ phải dựa vào khóa, bội số, mức tham gia và ý nghĩa nghiệp vụ.

### 3.2. Từ điển dữ liệu và ma trận ràng buộc

Với mỗi bảng, mô tả mục đích, khóa, ý nghĩa thuộc tính, kiểu dữ liệu, NULL/default và các miền giá trị. Dùng tên nhất quán giữa sơ đồ, tài liệu và SQL.

Trong DESIGN.md, lập ma trận có các cột:

`Mã quy tắc | Nội dung | Bảng/thuộc tính liên quan | Cơ chế thực thi | Vị trí SQL | Ca kiểm thử`

Bao phủ BR01–BR13 và các quy tắc bổ sung. Phần không biểu diễn được trên sơ đồ phải được ghi ở đây. Mỗi quy tắc cần chỉ rõ được kiểm tra bằng constraint, trigger, thủ tục hay cơ chế khác; không ghi chung chung “xử lý ở ứng dụng”.

## 4. Chuẩn hóa và tính toàn vẹn

### 4.1. Phân tích chuẩn hóa

Trong DESIGN.md:

1. Đưa ra ít nhất một cấu trúc dữ liệu chưa chuẩn hóa hoặc còn dư thừa phù hợp bối cảnh, và minh họa bất thường khi thêm, sửa, xóa bằng dữ liệu cụ thể.
2. Với các quan hệ chính — người tham gia, tham gia chương trình/vai trò, ghép cặp, mục tiêu, buổi gặp, quan hệ buổi gặp–mục tiêu và phản hồi — xác định khóa ứng viên, khóa được chọn và các phụ thuộc hàm có ý nghĩa. Nếu ánh xạ khác cách gọi này, nêu bảng tương ứng.
3. Giải thích vì sao mỗi quan hệ đạt 1NF, 2NF, 3NF dựa trên phụ thuộc hàm và khóa ứng viên; không chỉ liệt kê tên dạng chuẩn. Khóa thay thế không tự loại bỏ phụ thuộc bộ phận hoặc bắc cầu giữa các thuộc tính còn lại.
4. Nếu thực hiện phân rã, giải thích khả năng nối không mất mát và việc bảo toàn phụ thuộc hàm; chỉ rõ hạn chế nếu có.
5. Chỉ rõ mọi dữ liệu dẫn xuất được lưu, hoặc lựa chọn phi chuẩn hóa: lý do, nguồn dữ liệu gốc, rủi ro sai lệch và cơ chế giữ nhất quán.

Mục tiêu mặc định là 3NF. Không cần ép mọi bảng phải được phân rã thêm nếu thiết kế đã phù hợp. Yêu cầu lưu một thuộc tính dẫn xuất ở mục 5 là ngoại lệ có chủ đích; bạn vẫn phải phân tích ảnh hưởng của ngoại lệ đó.

### 4.2. Thực thi ràng buộc

- Ưu tiên PK, FK, UNIQUE, NOT NULL, CHECK và cơ chế khai báo tương đương mà DBMS hỗ trợ. Không dùng trigger thay cho ràng buộc khai báo đã đủ để kiểm tra quy tắc.
- Kiểm tra cả INSERT, UPDATE và DELETE có thể làm vi phạm một quy tắc, kể cả thay đổi ở bảng cha hoặc bản ghi được tham chiếu.
- Với quy tắc liên bảng/liên hàng không thể diễn đạt đầy đủ bằng constraint của DBMS đã chọn, dùng trigger hoặc cơ chế tương đương có thể kiểm chứng trong cơ sở dữ liệu.
- Nếu chỉ cho phép thao tác qua thủ tục, phải có cơ chế quyền truy cập ngăn DML trực tiếp bỏ qua kiểm tra. Không cần làm hệ thống tài khoản ứng dụng.
- Các thao tác nhiều bước phải giữ tính nguyên tử: thất bại không được để lại trạng thái cập nhật dở dang. Nêu giới hạn về ghi đồng thời; kiểm thử đồng thời nâng cao là phần mở rộng.

## 5. Yêu cầu triển khai SQL

### 5.1. Nền tảng và khả năng chạy lại

Tự chọn **một DBMS có hỗ trợ trigger và stored procedure** để đáp ứng đầy đủ bài tập. Ghi tên, phiên bản thực tế và công cụ chạy SQL trong README.md. Nếu nền tảng chỉ có hàm có thể thực hiện DML với hành vi tương đương, được dùng thay stored procedure nhưng phải giải thích và demo được.

Từ một database/schema trống, bộ script phải tạo được toàn bộ đối tượng, nạp dữ liệu và chạy demo theo đúng thứ tự đã hướng dẫn. Không phụ thuộc thao tác tạo bảng hoặc sửa dữ liệu thủ công ngoài tài liệu. Không đưa mật khẩu hay thông tin kết nối thật vào bài nộp.

### 5.2. Dữ liệu mẫu

Tạo dữ liệu giả lập có ý nghĩa ở tất cả bảng. Quy mô tối thiểu:

| Khái niệm | Quy mô tối thiểu |
| --- | --- |
| Người tham gia | 15 người, trong đó có ít nhất 3 mentor và 10 mentee; cho phép vai trò chồng lấn |
| Chương trình | 2 chương trình |
| Lần ghép cặp | 12 lần, có ít nhất 2 mentee đổi mentor trong cùng chương trình |
| Mục tiêu | 20 mục tiêu, bao phủ đủ các trạng thái |
| Buổi gặp | 30 buổi, có đủ 4 trạng thái |
| Ghi nhận thảo luận mục tiêu | 15 liên kết hợp lệ; có một buổi thảo luận nhiều mục tiêu và một mục tiêu được thảo luận qua nhiều buổi |
| Phản hồi | 10 phản hồi; có buổi hoàn thành chưa được phản hồi |

Các con số là số đối tượng nghiệp vụ, không bắt buộc bằng số hàng của một bảng cụ thể. Bảng danh mục chỉ cần đủ giá trị hợp lệ; bảng khác cần đủ dữ liệu để minh họa quan hệ và truy vấn, không bắt buộc 5–10 hàng cho mọi bảng.

Dữ liệu phải có trường hợp biên: mentor chưa có mentee tại thời điểm báo cáo; mentee chưa có mục tiêu; chương trình chưa có buổi gặp; người tham gia nhiều chương trình. Bạn có thể cần thêm chương trình hoặc bản ghi ngoài mức tối thiểu để bao phủ các trường hợp này. Dùng thời điểm báo cáo cố định trong demo để kết quả có thể tái lập.

### 5.3. Thủ tục thêm, sửa, xóa

Chọn **một bảng nghiệp vụ** và viết ba stored procedure hoặc cơ chế tương đương cho INSERT, UPDATE, DELETE.

- Mô tả tham số, điều kiện được thực hiện, kết quả và lỗi trả về.
- INSERT phải tạo dữ liệu hợp lệ; UPDATE phải có thay đổi nghiệp vụ thực sự.
- DELETE phải có ít nhất một tình huống được phép và một tình huống bị từ chối. Giải thích vì sao cần xóa, dữ liệu nào phải giữ để bảo toàn lịch sử và chính sách FK liên quan.
- Mỗi thủ tục có ví dụ gọi thành công và thất bại. Khi thất bại, database phải giữ trạng thái nhất quán.

Đổi trạng thái hoặc soft delete có thể là chính sách bổ sung, nhưng không thay thế hoàn toàn bài thực hành thủ tục DELETE. Vì vậy, hãy chọn bảng có trường hợp xóa vật lý hợp lý, chẳng hạn dữ liệu tạo nhầm chưa phát sinh lịch sử.

### 5.4. Trigger

Thực hiện đủ hai mục đích:

1. **Bảo vệ nghiệp vụ:** Chọn ít nhất một quy tắc liên bảng/liên hàng cần trigger trong DBMS đã chọn. Giải thích vì sao constraint khai báo chưa đủ; xử lý các thao tác có thể phá vỡ quy tắc.
2. **Duy trì thuộc tính dẫn xuất được lưu:** Chọn một giá trị có ý nghĩa và lưu trong database, công bố công thức, nguồn dữ liệu và giá trị khởi tạo. Trigger phải cập nhật đúng khi thêm, sửa hoặc xóa dữ liệu nguồn, kể cả đổi trạng thái hay đổi đối tượng được tổng hợp nếu schema cho phép. Có truy vấn tính lại từ dữ liệu gốc để đối chiếu.

Không bắt buộc số lượng trigger cụ thể; số trigger phụ thuộc thiết kế và DBMS. Một view hoặc generated column đơn thuần không thay thế mục thực hành trigger thứ hai. Nếu có chuỗi phụ thuộc giữa các giá trị dẫn xuất, phải giải thích thứ tự tính hoặc thiết kế để tránh phụ thuộc vòng.

### 5.5. Truy vấn nghiệp vụ

Viết đủ **Q01–Q10** dưới đây trong queries.sql. Mỗi truy vấn có chú thích về mục đích, tham số, ý nghĩa từng hàng kết quả và quy ước tính toán. Chỉ cần một cách giải đúng; không phải cố sử dụng mọi kỹ thuật SQL.

| Mã | Câu hỏi và yêu cầu đầu ra |
| --- | --- |
| Q01 | Tại thời điểm T, liệt kê mentee đang được ghép với mentor nào trong chương trình được chọn; kèm định danh lần ghép cặp và khoảng thời gian. |
| Q02 | Với một mentee trong một chương trình, liệt kê lịch sử đổi mentor và các buổi gặp của từng lần ghép cặp. Lần ghép cặp chưa có buổi gặp vẫn phải xuất hiện. |
| Q03 | Liệt kê mục tiêu của một mentee trong một chương trình: trạng thái, số buổi hoàn thành đã thảo luận và thời điểm thảo luận gần nhất. Mục tiêu chưa được thảo luận có số buổi bằng 0. |
| Q04 | Liệt kê các mục tiêu được thảo luận trong một buổi hoàn thành, kèm nội dung ghi nhận nếu thiết kế có lưu. |
| Q05 | Đếm mentee đang được mỗi mentor phụ trách tại T theo từng chương trình. Hiển thị cả mentor đã tham gia chương trình nhưng có số mentee bằng 0. Không đếm trùng do JOIN với buổi gặp. |
| Q06 | Theo từng chương trình, thống kê số lượng và tỷ lệ phần trăm của 4 trạng thái buổi gặp trong khoảng báo cáo `[từ, đến)`, dựa trên thời điểm bắt đầu buổi. Mẫu số là tổng số buổi thuộc khoảng đó; hiển thị cả chương trình không có buổi, số lượng bằng 0 và tỷ lệ NULL khi mẫu số bằng 0. |
| Q07 | Theo mentor và chương trình, tính điểm phản hồi trung bình và số phản hồi của các buổi hoàn thành trong khoảng báo cáo, lọc theo thời điểm bắt đầu buổi. Buổi chưa có phản hồi không được coi là điểm 0. Mentor chưa có phản hồi có trung bình NULL và số phản hồi bằng 0. |
| Q08 | Tìm các buổi hoàn thành chưa có phản hồi được gửi tại hoặc trước T; trả về mentor, mentee và số ngày đã qua kể từ lúc kết thúc. Nêu cách làm tròn số ngày; không lấy buổi có thời điểm kết thúc sau T. |
| Q09 | Tìm mentee tham gia chương trình nhưng chưa có mục tiêu nào trong chương trình đó; trả về người và chương trình tương ứng. |
| Q10 | Với từng chương trình, xếp hạng mentee theo số buổi hoàn thành trong khoảng báo cáo, gộp các lần đổi mentor. Bao gồm mentee chưa có buổi với số lượng 0; những người cùng số buổi phải cùng hạng. |

Với Q07, tập mentor đầu ra là các mentor đã tham gia chương trình được chọn, kể cả chưa có buổi hoặc phản hồi trong khoảng báo cáo. Với các truy vấn khác có tham số, ghi rõ giá trị mẫu để người chấm chạy ngay.

Các báo cáo dùng trạng thái hiện đang lưu trong database; không yêu cầu tái dựng trạng thái lịch sử tại T. Riêng Q08 xét thời điểm gửi phản hồi để xác định phản hồi đã tồn tại tại T hay chưa.

Thêm **2 truy vấn do bạn đề xuất** (Q11–Q12), mỗi truy vấn trả lời một nhu cầu thực tế ngoài việc đổi bộ lọc của Q01–Q10. Giải thích ai sử dụng và quyết định nào được hỗ trợ.

## 6. Kiểm thử và demo

### 6.1. Bộ kiểm thử bắt buộc

Nộp ít nhất **12 ca kiểm thử có thể chạy lại**, trong đó có ít nhất **6 ca dữ liệu không hợp lệ bị database từ chối**. Mỗi ca có: mã, quy tắc liên quan, dữ liệu/điều kiện ban đầu, thao tác SQL, kết quả mong đợi, kết quả thực tế và cách khôi phục dữ liệu.

Bộ kiểm thử phải bao phủ các nhóm sau; một nhóm có thể cần nhiều ca:

- Luồng hợp lệ: tham gia chương trình → ghép cặp → buổi gặp → thảo luận mục tiêu → phản hồi.
- Khóa/miền giá trị: trùng dữ liệu phải duy nhất, FK không tồn tại, điểm hoặc trạng thái sai miền.
- Ghép cặp: sai vai trò/chương trình hoặc tự ghép với chính mình; hai khoảng của cùng mentee chồng lấn; hai khoảng tiếp giáp hợp lệ.
- Thời gian: buổi gặp nằm ngoài khoảng ghép cặp; sửa thời gian chương trình hoặc ghép cặp làm dữ liệu con không còn hợp lệ.
- Mục tiêu: gắn mục tiêu của mentee khác hoặc chương trình khác vào buổi gặp.
- Phản hồi: gửi cho buổi chưa hoàn thành hoặc gửi trùng cho một buổi.
- Lịch sử: DELETE được phép và DELETE bị từ chối; thử sửa/xóa dữ liệu đang có bản ghi phụ thuộc.
- Thuộc tính dẫn xuất: kiểm tra sau INSERT, UPDATE và DELETE dữ liệu nguồn; so sánh giá trị lưu với giá trị tính lại, bao gồm trường hợp kết quả bằng 0.

Các trường hợp sai phải xuất phát từ một trạng thái ban đầu hợp lệ. Không để một lỗi FK ngoài ý muốn che mất quy tắc mà ca kiểm thử định kiểm tra. Tách riêng lỗi dự kiến để bộ kiểm thử có thể chạy tiếp theo cách phù hợp với DBMS.

Với Q01–Q12, ghi đầu ra mong đợi hoặc kết quả kiểm chứng trên dữ liệu mẫu cố định. Kiểm tra đặc biệt các trường hợp số đếm bằng 0, NULL, trùng hàng sau JOIN và ranh giới thời gian.

### 6.2. Demo cuối khóa

Chuẩn bị phần trình bày **15–20 phút**, sau đó trao đổi thiết kế với mentor:

1. Bài toán, quy tắc và các giả định quan trọng.
2. EER, ánh xạ quan hệ, một quyết định chuẩn hóa đáng chú ý.
3. Chạy từ database/schema trống hoặc chứng minh quy trình dựng lại có thể lặp lại.
4. Demo các truy vấn tiêu biểu, ba thủ tục và hai mục đích trigger.
5. Demo một số ca bị từ chối, đối chiếu thuộc tính dẫn xuất và nêu giới hạn còn lại.

Không cần trình bày tất cả câu SQL trên màn hình; toàn bộ script và kết quả kiểm chứng vẫn phải có trong bài nộp. Bạn cần giải thích được mã và quyết định thiết kế của mình.

## 7. Nộp bài qua repository chung và Pull Request

### 7.1. Quy ước repository và nhánh

Mentor cung cấp một repository chung của lớp và quyền tạo nhánh cho học viên. Mỗi học viên dùng một mã định danh duy nhất do mentor xác nhận (ví dụ `hv001`), làm bài trên nhánh `submission/<ma-hoc-vien>` và lưu toàn bộ sản phẩm trong `submissions/<ma-hoc-vien>/`.

- Nhánh gốc: `main`. Tạo nhánh cá nhân từ `main`, không tạo từ nhánh của học viên khác.
- Chỉ commit bài của mình vào thư mục cá nhân. Không sửa đề, hướng dẫn chung hoặc bài của người khác trong PR nộp bài.
- Mỗi học viên duy trì **một PR xuyên suốt project**, từ nhánh cá nhân vào `main`; tiêu đề: `[Final Project] <ma-hoc-vien> - <Ho ten>`.
- Có thể mở Draft PR từ mốc góp ý đầu tiên. Sau mỗi mốc, push cập nhật lên cùng nhánh và ghi tiến độ trong PR để mentor review.
- Học viên không tự merge PR hoặc push trực tiếp lên `main`. Mentor quản lý việc merge; **PR chưa merge vẫn được xem là đã nộp** nếu đã đáp ứng điều kiện nộp cuối khóa.

### 7.2. Cấu trúc thư mục cá nhân

Trong nhánh của bạn, cấu trúc bài nộp là:

```text
submissions/<ma-hoc-vien>/
├── README.md
├── DESIGN.md
├── diagrams/
│   ├── eer.pdf                  # Hoặc SVG/PNG đủ rõ
│   ├── relational_schema.pdf    # Hoặc SVG/PNG đủ rõ
│   └── source/                 # File nguồn có thể chỉnh sửa của sơ đồ
├── sql/
│   ├── 01_schema.sql
│   ├── 02_routines.sql
│   ├── 03_triggers.sql
│   ├── 04_seed.sql
│   ├── queries.sql
│   └── demo.sql
└── tests/
    ├── integrity_tests.sql
    └── TEST_RESULTS.md
```

Đây là cấu trúc mặc định. Có thể tách/gộp file hoặc đổi thứ tự nếu DBMS và quan hệ phụ thuộc yêu cầu, nhưng README phải chỉ rõ thứ tự chạy và vị trí từng sản phẩm. Không bỏ hạng mục chỉ vì đổi cấu trúc. Trigger duy trì dữ liệu dẫn xuất cần hoạt động khi nạp seed, hoặc có bước khởi tạo và đối chiếu rõ ràng.

| Tệp | Nội dung |
| --- | --- |
| README.md | Giới thiệu ngắn, môi trường/phiên bản, cách tạo database/schema, thứ tự chạy, tham số demo, cách chạy kiểm thử, kết quả dự kiến và cách đưa môi trường về trạng thái ban đầu. Cảnh báo rõ nếu lệnh reset xóa dữ liệu. |
| DESIGN.md | Phạm vi, quy tắc bổ sung, giải thích EER và ánh xạ, từ điển dữ liệu, chuẩn hóa, ma trận ràng buộc, chính sách sửa/xóa, lựa chọn thuộc tính dẫn xuất và giới hạn. |
| diagrams/ | Bản xuất đọc được và file nguồn để chỉnh sửa. |
| 01_schema.sql | Bảng, khóa, constraint, chỉ mục và view nếu có. Giải thích chỉ mục/view ngoài khóa trong DESIGN.md. |
| 02_routines.sql | Các thủ tục thêm/sửa/xóa và thủ tục hỗ trợ nếu có. |
| 03_triggers.sql | Trigger, hàm hỗ trợ và chú thích quy tắc được bảo vệ. |
| 04_seed.sql | Dữ liệu mẫu hợp lệ, tái lập được. |
| queries.sql | Q01–Q12, tham số mẫu và chú thích. |
| demo.sql | Các lệnh gọi và kịch bản demo, nêu rõ phần nào làm thay đổi dữ liệu. |
| tests/ | Kiểm thử, kết quả thực tế, lỗi dự kiến và kiểm chứng kết quả truy vấn. |

Không nộp database dump thay cho schema và seed. Có thể đính kèm dump để tiện sử dụng, nhưng người chấm phải dựng lại được từ các file SQL.

### 7.3. Quy trình nộp và cập nhật bài

1. Clone repository mentor cung cấp. Từ `main` mới nhất, tạo nhánh `submission/<ma-hoc-vien>`.
2. Tạo thư mục `submissions/<ma-hoc-vien>/`, thêm sản phẩm và commit theo từng phần việc có ý nghĩa.
3. Push nhánh lên repository chung, mở PR vào `main` và điền mẫu PR có sẵn: thông tin học viên, đường dẫn bài, DBMS/phiên bản, cách chạy, tiến độ và vấn đề còn mở.
4. Sau review, sửa trên cùng nhánh, push commit mới và phản hồi từng góp ý. Không tạo PR mới chỉ để nộp bản sửa.
5. Khi nộp cuối khóa, chuyển PR khỏi Draft, hoàn thành checklist và thêm bình luận `Nộp cuối khóa: <commit SHA đầy đủ>`. Commit này phải chứa đủ sản phẩm, có thể chạy lại theo README và đã có trên nhánh PR trước hạn mentor công bố.
6. Khi mentor yêu cầu sửa sau hạn, tiếp tục dùng PR đó và ghi rõ commit sửa. Phiên bản chấm mặc định là commit đã chốt; mentor xác nhận nếu sử dụng phiên bản bổ sung.

Thông tin nộp bài gồm **URL PR và commit SHA được chốt**. Không cần nộp thêm ZIP trừ khi mentor yêu cầu. Không xóa nhánh, force-push hoặc viết lại lịch sử sau khi mở PR để giữ nguyên căn cứ review và phiên bản nộp. Nếu gặp sự cố với PR hoặc lịch sử Git, liên hệ mentor trước khi thay đổi cách nộp.

## 8. Tiêu chí đánh giá — 100 điểm

| Hạng mục | Điểm | Căn cứ đánh giá |
| --- | ---: | --- |
| Phân tích nghiệp vụ | 15 | Phạm vi và quy tắc rõ, không mâu thuẫn (8); giả định, trường hợp biên và ma trận quy tắc kiểm chứng được (7). |
| EER và mô hình quan hệ | 30 | Thực thể, khóa, bội số, mức tham gia đúng (12); các cấu trúc EER bắt buộc có ý nghĩa và ánh xạ hợp lý (10); sơ đồ, từ điển và SQL nhất quán (8). |
| Chuẩn hóa và toàn vẹn dữ liệu | 20 | Phụ thuộc hàm, khóa, bất thường dữ liệu và 1NF–3NF có lập luận (10); cơ chế bảo vệ đầy đủ quy tắc, bao gồm sửa/xóa dữ liệu liên quan (10). |
| SQL và kiểm thử | 25 | Dựng lại được và seed đủ trường hợp (5); Q01–Q12 đúng kết quả (8); thủ tục và trigger đúng hành vi, xử lý lỗi nhất quán (8); bộ kiểm thử có bằng chứng (4). |
| Tài liệu và bảo vệ thiết kế | 10 | Tài liệu và hướng dẫn đủ để người khác chạy lại (5); demo rõ, giải thích được lựa chọn và giới hạn (5). |
| **Tổng** | **100** | |

Mỗi phần được chấm theo bằng chứng thực tế, không theo số bảng, số trang hay độ dài SQL. Thiếu sản phẩm thì không có điểm cho tiêu chí tương ứng; không trừ lặp cùng một lỗi trong cùng tiêu chí. Một lỗi có thể ảnh hưởng các tiêu chí khác nhau nếu làm hỏng những kết quả riêng, chẳng hạn schema sai và truy vấn cho kết quả sai.

Thiết kế có nhiều phương án hợp lý, nhưng tính đúng của khóa, quan hệ, ràng buộc và kết quả SQL vẫn phải chứng minh được. Mở rộng chỉ có giá trị khi phần bắt buộc đã hoạt động; không cộng điểm chỉ vì thêm công nghệ hoặc bảng.

## 9. Lộ trình gợi ý cho 8–12 tuần

| Mốc | Bản 8 tuần | Bản 12 tuần | Sản phẩm để mentor góp ý |
| --- | --- | --- | --- |
| Chốt nghiệp vụ | Tuần 1 | Tuần 1–2 | Phạm vi, quy tắc, tình huống biên, câu hỏi còn mở |
| Thiết kế khái niệm | Tuần 2 | Tuần 3–4 | EER và giải thích các cấu trúc bắt buộc |
| Thiết kế logic | Tuần 3 | Tuần 5–6 | Lược đồ quan hệ, từ điển, phân tích chuẩn hóa |
| Triển khai | Tuần 4–5 | Tuần 7–8 | Schema, seed, constraint, thủ tục, trigger |
| Truy vấn và kiểm thử | Tuần 6–7 | Tuần 9–10 | Q01–Q12, kiểm thử và kết quả |
| Hoàn thiện và demo | Tuần 8 | Tuần 11–12 | Bài nộp dựng lại được, tài liệu nhất quán, demo |

Đây là lộ trình gợi ý; lịch nộp cụ thể theo thông báo của mentor. Khi thay đổi thiết kế sau góp ý, cập nhật đồng thời sơ đồ, tài liệu, SQL và kiểm thử.

## 10. Phần mở rộng tự chọn

Sau khi hoàn thành phần bắt buộc, có thể chọn một hướng: quản lý sức chứa mentor; phát hiện trùng lịch; audit lịch sử; kiểm thử ghi đồng thời; phân tích hiệu năng bằng execution plan; hoặc mở rộng nghiệp vụ có lý do rõ ràng. Nêu tác động đến mô hình, ràng buộc và truy vấn. Không yêu cầu xây dựng ứng dụng để chứng minh phần mở rộng.

## 11. Checklist trước khi nộp

- [ ] BR01–BR13 và quy tắc bổ sung có cơ chế thực thi và ca kiểm chứng.
- [ ] EER, lược đồ quan hệ, từ điển dữ liệu và SQL nhất quán.
- [ ] Có đủ cấu trúc EER bắt buộc và giải thích ánh xạ.
- [ ] Phân tích chuẩn hóa dựa trên khóa ứng viên và phụ thuộc hàm.
- [ ] Dựng lại thành công từ database/schema trống theo README.
- [ ] Seed đủ quy mô, có lịch sử và các trường hợp 0/NULL/ranh giới.
- [ ] Ba thủ tục và hai mục đích trigger hoạt động, không để dữ liệu dở dang khi lỗi.
- [ ] Q01–Q12 có tham số mẫu và kết quả kiểm chứng.
- [ ] Có ít nhất 12 ca kiểm thử, ít nhất 6 ca bị từ chối như dự kiến.
- [ ] File nguồn sơ đồ, hướng dẫn, script và kết quả demo đầy đủ.
- [ ] Bài nằm trong đúng thư mục cá nhân; PR vào `main` đã sẵn sàng review và có commit SHA chốt bài.
