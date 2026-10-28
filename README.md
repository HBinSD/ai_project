# BÀI TẬP LỚN: TÌM TUYẾN ĐƯỜNG GIAO HÀNG TRÊN MẠNG ĐƯỜNG BỘ CÓ HƯỚNG
> **Môn học:** Nhập môn Trí tuệ Nhân tạo (Artificial Intelligence)  
> **Trường:** Đại học Thủ đô Hà Nội (HNMU) — Năm học 2025–2026  
> **Chủ đề:** CHỦ ĐỀ 01 • AI Agents & Graph Search

---

## PHẦN 1: BÁO CÁO MÔ TẢ TỔNG QUAN (1-PAGE EXECUTIVE SUMMARY)

### 1. Bối cảnh & Phát biểu bài toán
* **Tình huống:** Một cửa hàng dịch vụ cần giao đơn hàng từ kho xuất phát $S$ tới vị trí khách hàng $T$ trong khu vực nội thành. Lựa chọn sai tuyến đường sẽ làm tăng thời gian và chi phí di chuyển.
* **Ý nghĩa:** Mạng giao thông đô thị có tính chất có hướng (đường 1 chiều, dải phân cách, cấm rẽ) và nhiều tuyến thay thế. Do đó, hệ thống không thể sử dụng đường chim bay mà phải tìm đường đi hợp lệ trên đồ thị giao thông.
* **Bài toán:** Tìm tuyến đường hợp lệ có tổng chiều dài nhỏ nhất từ nút $S$ đến nút $T$ trên đồ thị đường bộ có hướng $G = (V, E)$.

### 2. Đầu vào, Đầu ra và Mục tiêu
* **Đầu vào (Inputs):**
  * Phạm vi khu vực địa lý (ví dụ: *Phường Cầu Giấy, Hà Nội*, bán kính $1 - 3\text{ km}$).
  * Tọa độ hoặc ID nút xuất phát $S$ và nút đích $T$.
  * Đồ thị đường bộ $G = (V, E)$ thu thập từ OpenStreetMap qua `OSMnx` với `network_type="drive"`.
* **Đầu ra (Outputs):**
  * Chuỗi nút/cạnh tạo thành lộ trình $P = (v_S, v_1, v_2, \dots, v_T)$.
  * Tổng chiều dài tuyến đường $L = \sum \text{length}(e_i)$ (mét).
  * Trực quan hóa tuyến đường trên bản đồ thực tế.
* **Mục tiêu:**
  * **Tính đúng đắn:** Tuân thủ quy tắc giao thông (đi đúng chiều, không đi qua cạnh cấm).
  * **Tính tối ưu:** Tìm ra quãng đường ngắn nhất.
  * **So sánh:** Đánh giá hiệu năng tìm kiếm giữa **Dijkstra (Uniform Cost Search)** và **A* Search**.

### 3. Nguồn dữ liệu, Giấy phép và Giới hạn
* **Nguồn dữ liệu:** OpenStreetMap (OSM) thông qua thư viện Python `OSMnx`.
* **Giấy phép (License):** Cấp phép bởi OpenStreetMap Foundation (OSMF) theo giấy phép Open Database License (ODbL). Nghĩa vụ ghi công: `"© OpenStreetMap contributors"`.
* **Giới hạn bài toán:**
  * **Mô hình tĩnh:** Trọng số là độ dài vật lý $\text{length}$ ($m$). Bỏ qua các yếu tố động như mật độ giao thông, kẹt xe hay thời tiết.
  * **Phạm vi đơn:** Định tuyến cho $1$ đơn giao hàng ($S \to T$).

### 4. Mô hình hóa Trạng thái, Feature và Chi phí
* **Trạng thái (State Space):** Mỗi trạng thái $s \in V$ đại định cho một giao lộ/nút trên bản đồ:
  $$\text{State}(s) = (\text{node\_id}, \text{latitude}, \text{longitude})$$
* **Hành động (Actions):** Di chuyển từ nút $u$ sang nút $v$ nếu tồn tại cạnh có hướng $(u, v) \in E$.
* **Chi phí thực tế $g(n)$:** Tổng chiều dài tích lũy từ nút $S$ tới nút $n$:
  $$g(n) = \sum_{e \in \text{path}(S \to n)} \text{length}(e)$$
* **Hàm Heuristic $h(n)$ cho A*:** Khoảng cách Haversine (đường chim bay trên mặt cầu) từ nút $n$ đến đích $T$:
  $$h(n) = \text{Haversine}\left((\text{lat}_n, \text{lon}_n), (\text{lat}_T, \text{lon}_T)\right)$$
* **Chứng minh tính chấp nhận được (Admissibility $h(n) \le h^*(n)$):**
  Khoảng cách ngắn nhất giữa hai điểm trong không gian không bao giờ nhỏ hơn đường nối thẳng giữa chúng. Quãng đường thực tế đi qua các nút giao $h^*(n)$ luôn lớn hơn hoặc bằng khoảng cách chim bay trực tiếp $h(n)$. Do đó $h(n) \le h^*(n), \forall n \in V$, đảm bảo A* Search luôn tìm ra nghiệm tối ưu toàn cục.

## PHẦN 2: KẾ HOẠCH THỰC NGHIỆM CHI TIẾT

### 1. Các bước tiến hành thực nghiệm
1. **Bước 1: Tải đồ thị & Lưu Cache:** Chọn khu vực Phường Cầu Giấy (bán kính $2\text{ km}$), tải đồ thị có hướng `network_type="drive"`, lưu thành file `data/caugiai_drive.graphml`.
2. **Bước 2: Cài đặt thuật toán Baseline:** Lập trình thuật toán Dijkstra (Sử dụng Priority Queue, chi phí $g(n)$).
3. **Bước 3: Cài đặt thuật toán A* Search:** Lập trình A* với hàm Heuristic Haversine $h(n)$. Ghi nhận số lượng trạng thái được mở rộng ($\text{expanded nodes}$) và thời gian thực thi ($\text{ms}$).
4. **Bước 4: Chạy thử nghiệm trên tập 10+ Test Cases:** Chọn 10 cặp $S/T$ hợp lệ từ ngắn đến xa và chạy song song 2 thuật toán.
5. **Bước 5: Trực quan hóa & Báo cáo trường hợp biên:** Xuất bản đồ tuyến đường và phân tích các trường hợp khó.

### 2. Danh sách 10 ca kiểm thử (Test Cases)

| Mã ca | Khoảng cách chim bay | Đặc điểm lộ trình / Mục đích kiểm thử |
| :---: | :---: | :--- |
| `CASE_01` | $\sim 300\text{ m}$ | Lộ trình rất ngắn, kiểm tra độ trễ khởi tạo |
| `CASE_02` | $\sim 600\text{ m}$ | Lộ trình ngắn qua 2 ngã tư |
| `CASE_03` | $\sim 1.0\text{ km}$ | Tuyến đường nội đô thông thường |
| `CASE_04` | $\sim 1.4\text{ km}$ | Tuyến đi qua trục đường một chiều |
| `CASE_05` | $\sim 1.8\text{ km}$ | Tuyến có nhiều đường ngõ rẽ thay thế |
| `CASE_06` | $\sim 2.3\text{ km}$ | Lộ trình trung bình qua nút giao lớn |
| `CASE_07` | $\sim 3.0\text{ km}$ | Lộ trình xa kết nối 2 phường |
| `CASE_08` | $\sim 4.2\text{ km}$ | Lộ trình rất xa xuyên khu vực |
| `CASE_09` | Khó | Điểm $S$ nằm trong ngõ cụt, bắt buộc quay đầu |
| `CASE_10` | Khó | Điểm $T$ nằm trên tuyến một chiều ngược hướng di chuyển |

### 3. Mẫu bảng tổng hợp kết quả thực nghiệm

| Mã ca | Chiều dài Dijkstra ($m$) | Chiều dài A* ($m$) | Số nút duyệt (Dijkstra) | Số nút duyệt (A*) | Thời gian Dijkstra ($\text{ms}$) | Thời gian A* ($\text{ms}$) | Tỷ lệ giảm nút duyệt (%) |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| `CASE_01` | *100% Khớp* | *100% Khớp* | ... | ... | ... | ... | ... |
| `CASE_02` | ... | ... | ... | ... | ... | ... | ... |
| ... | ... | ... | ... | ... | ... | ... | ... |
| `CASE_10` | ... | ... | ... | ... | ... | ... | ... |

### 4. Phân tích 3 trường hợp khó (Edge Cases)
1. **Trường hợp 1: Đường một chiều ngược hướng:**
   * *Hiện tượng:* Điểm $T$ ở ngay phía sau điểm $S$ nhưng đường là một chiều.
   * *Giải thích:* Thuật toán bắt buộc phải cho xe đi vòng qua các giao lộ phía trước để quay lại, quãng đường thực tế $h^*(n)$ lớn hơn nhiều so với $h(n)$.
2. **Trường hợp 2: Ngõ cụt / Hai thành phần không liên thông:**
   * *Hiện tượng:* Không tìm thấy đường đi từ $S$ đến $T$.
   * *Xử lý:* Đồ thị hướng có thể có các nút bị cô lập. Hệ thống sẽ phát hiện tập mở rỗng ($\text{OpenSet} = \emptyset$) và trả về thông báo lỗi hợp lệ thay vì treo chương trình.
3. **Trường hợp 3: Điểm tọa độ $S, T$ nằm ngoài mạng đường bộ:**
   * *Xử lý:* Sử dụng hàm `osmnx.distance.nearest_nodes` để ánh xạ tọa độ GPS người dùng nhập vào nút giao gần nhất trên đồ thị trước khi chạy thuật toán tìm kiếm.
