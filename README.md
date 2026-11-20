# Final Project — Database & Data Modeling

Repository chung để các bạn nộp bài và nhận review từ mentor. Mỗi em làm cá nhân trong 8–10 tuần.

Đọc [đề bài đầy đủ](docs/FINAL_PROJECT_SPEC.md) hoặc [bản PDF](docs/FINAL_PROJECT_SPEC.pdf). Yêu cầu nghiệp vụ, SQL, kiểm thử và tiêu chí đánh giá nằm trong đề bài.

## Quy ước nộp bài

| Nội dung | Quy ước |
| --- | --- |
| Mã học viên | Mã duy nhất được mentor xác nhận, ví dụ `hv001` |
| Nhánh | `submission/<hovaten>` |
| Thư mục bài | `submissions/<hovaten>/` |
| PR | Từ nhánh cá nhân vào `main` |
| Tiêu đề PR | `[Final Project] <Ho ten>` |
| Số PR | Một PR xuyên suốt project; cập nhật cùng PR sau review |
| Phiên bản nộp | URL PR và commit SHA đầy đủ ghi trong bình luận chốt bài |

Chỉ thay đổi thư mục bài của mình. Mentor quản lý đề và hướng dẫn chung trên `main`. Học viên không push trực tiếp lên `main` hoặc tự merge PR.

## Bắt đầu

Thay `REPO_URL` bằng URL mentor cung cấp và `hv001` bằng tên của các bạn. Các lệnh sau dành cho lần tạo nhánh đầu tiên, thực hiện trong terminal Bash/Git Bash:

```bash
git clone REPO_URL database-final-project
cd database-final-project
git switch main
git pull --ff-only origin main
git switch -c submission/hv001
mkdir -p submissions/hv001/diagrams/source
mkdir -p submissions/hv001/sql submissions/hv001/tests
```

Tạo các tệp theo cấu trúc ở mục 7 của đề. Git chỉ lưu thư mục khi có tệp; không cần commit thư mục trống.

Sau khi có sản phẩm đầu tiên:

```bash
git add submissions/hv001/
git commit -m "docs: add initial business rules and project scope"
git push -u origin submission/hv001
```

Trên GitHub, mở PR với **base `main`**, **compare `submission/hv001`** và điền mẫu có sẵn. Có thể để Draft trong thời gian làm bài. Đặt PR là sẵn sàng review khi cần mentor góp ý và thông báo theo cách lớp thống nhất.

## Cập nhật sau review

Tiếp tục trên nhánh cá nhân đã tạo:

```bash
git switch submission/hv001
git add submissions/hv001/
git commit -m "fix: address mentor review on pairing constraints"
git push
```

PR tự cập nhật theo commit mới. Phản hồi góp ý của mentor, mô tả phần đã sửa và phần cần trao đổi. Không mở PR mới cho mỗi lần sửa. Không force-push, viết lại lịch sử hoặc xóa nhánh đã mở PR.

Nếu mentor cập nhật đề trên `main`, trước tiên commit công việc đang làm, rồi đồng bộ vào nhánh cá nhân bằng merge để giữ lịch sử:

```bash
git fetch origin
git merge origin/main
git push
```

Nếu có conflict, xử lý và kiểm tra lại trước khi push. Không xóa thay đổi của người khác để giải quyết conflict.

## Nộp cuối khóa

1. Chạy lại toàn bộ bài từ database/schema trống theo README cá nhân.
2. Kiểm tra đủ sản phẩm và checklist của đề; cập nhật mô tả PR.
3. Push toàn bộ commit cần nộp; chuyển PR khỏi Draft.
4. Chạy `git rev-parse HEAD` trên nhánh cá nhân để lấy commit SHA đầy đủ.
5. Bình luận trong PR: `Nộp cuối khóa: <commit SHA đầy đủ>` và gửi URL PR theo thông báo của mentor trước hạn nộp.

PR chưa merge vẫn được tính đã nộp nếu đã hoàn tất quy trình trên. Mentor chấm theo commit đã chốt; thay đổi sau hạn chỉ được tính khi mentor xác nhận. Khi được yêu cầu sửa, dùng tiếp PR hiện tại và ghi rõ commit bổ sung.

Không đưa mật khẩu thật, dữ liệu cá nhân thật hoặc file dữ liệu DBMS đang chạy vào repository.
