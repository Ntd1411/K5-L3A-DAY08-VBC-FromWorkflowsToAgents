# Báo cáo cá nhân — 2A202603599_NguyenXuanHung

## 1. Thông tin thành viên

- Họ tên: Nguyễn Xuân Hùng
- MSSV: 2A202603599
- Email: Xuanhung17032003@gmail.com
- GitHub: https://github.com/XuanHung1703

## 2. Phần việc phụ trách

Tôi phụ trách Shape C — Orchestrator và handoff: nhận yêu cầu hoàn tiền, giao investigator tra cứu đơn hàng và chính sách với quyền chỉ đọc, tính mức hoàn bằng code và gọi host kiểm duyệt trước khi ghi tiền. Tôi bổ sung test cho giới hạn quyền và bảo mật dữ liệu khi giao việc.

## 3. Đóng góp cụ thể

- Commit triển khai: `34897f7` — `Implement orchestrator and add handoff security tests`.
- Hoàn thiện `src/lab8/shapes/orchestrator.py`, giữ nguyên chữ ký `run_claim`; mở và đóng kết nối database bằng `try/finally`.
- Đọc claim bằng `get_claim_view`. Claim không tồn tại trả `failed`; thiếu mã đơn trả `needs_info`; đơn không hiển thị trong phạm vi người yêu cầu trả `escalated`.
- Tạo handoff bằng `make_handoff`, chỉ cấp `lookup_order` và `lookup_policy`. Investigator không được cấp quyền ghi refund.
- Dùng `run_readonly_investigators` để tra cứu; ghi handoff đã lọc vào trace, không ghi nguyên claim, ticket hoặc parent context.
- Gọi `compute_compensation` từ dữ liệu đơn hàng. Không dùng lời khách trong ticket để thay đổi số ngày trễ hay mức hoàn.
- Truyền quote đầy đủ và `crash_at` vào `issue_refund`; host kiểm approval, trần theo đơn và chống ghi trùng. Không bắt `InjectedCrash`.
- Dùng `finish()` ở các nhánh kết thúc bình thường để ghi trace và tạo `RunResult`; giữ `proposal_amount` là quote đầy đủ.
- Thêm bốn unit test trong `src/tests/test_orchestrator_handoff.py`: từ chối tool hoàn tiền, chấp nhận tool đọc, lọc dữ kiện nhạy cảm và chặn secret trong nội dung giao việc. Các test này kiểm cơ chế handoff, không thay thế test tích hợp orchestrator.

## 4. Bằng chứng

- [Commit triển khai](https://github.com/Ntd1411/K5-L3A-DAY08-VBC-FromWorkflowsToAgents/commit/34897f7).
- Kiểm tra riêng orchestrator: `62 passed` khi chạy `python -m pytest -q -m student -o addopts= -k orchestrator`.
- Cổng host tại thời điểm kiểm tra trước khi thêm bốn test: `54 passed`.
- Bốn test bổ sung đã chạy và đều pass:
  - `test_handoff_rejects_refund_tool`
  - `test_handoff_accepts_readonly_tools`
  - `test_handoff_filters_sensitive_facts`
  - `test_handoff_rejects_secret_in_task`
- Theo kết quả benchmark chung của nhóm trong `src/reports/benchmark.json` trên nhánh `main`: orchestrator đạt 70/70 trial trên 14 claim và năm seed `11,22,33,44,55`; pass rate và pass^5 đều là `1.000`; seed variance `0`; seed min–max `1.000–1.000`.
- Chi phí mô phỏng của orchestrator: khoảng `46.43 VND/success`; mean/p95 steps khoảng `4.36/5`. Bản triển khai hiện tại không gọi MockLLM, nên số lần gọi model bằng 0; đây là điều phối bằng code qua bộ chạy investigator có sẵn.
- Run id đối chiếu trong benchmark: `R-orchestrator-C-001-s11`.
- Báo cáo crash chung của nhóm `src/reports/crash.json` ghi `20/20`. Đây là kết quả suite chung, không phải 20 tình huống riêng của orchestrator.
- Các số liệu benchmark và crash ở trên được đối chiếu từ file kết quả của nhóm, không phải số liệu tự tạo hoặc khẳng định do cá nhân tôi chạy toàn bộ.

## 5. Công cụ AI đã dùng

Tôi sử dụng ChatGPT/OpenAI Codex để giải thích yêu cầu lab, hướng dẫn từng bước triển khai orchestrator, sửa thụt lề Python, đề xuất test và hỗ trợ thao tác Git. Tôi kiểm tra code bằng cách chạy test orchestrator, test handoff và cổng host; đối chiếu kết quả thực tế trước khi commit. Báo cáo được hỗ trợ soạn bằng AI và đối chiếu với code, lịch sử commit cùng báo cáo kết quả của nhóm.

## 6. Tự đánh giá

Tôi đã hoàn thành phần orchestrator và kiểm tra các nhánh xử lý trên bộ public. Tôi hiểu ranh giới trách nhiệm: investigator chỉ tra cứu, code tính mức hoàn, host kiểm duyệt và ghi tiền. Handoff phải lọc theo danh sách field được phép; việc tóm tắt bằng model không bảo đảm loại bỏ bí mật.

Kết quả public chưa chứng minh đạt hidden test hoặc hiệu quả trong môi trường thực tế. Chi phí là mô phỏng, latency là `unavailable`. Benchmark cho thấy orchestrator có cùng tỷ lệ đúng public với workflow nhưng chi phí cao hơn, vì vậy thêm bước điều phối chưa thể hiện lợi ích về độ đúng trong bộ dữ liệu này.
