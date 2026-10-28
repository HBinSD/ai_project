# Tìm tuyến đường giao hàng trên mạng đường bộ

Đồ án mô hình hóa mạng đường bộ quận Long Biên, Hà Nội thành đồ thị và tìm tuyến đường ngắn nhất giữa một điểm xuất phát và một điểm giao. Notebook hiện thực, so sánh Dijkstra với A*, và hiển thị tuyến tìm được trên bản đồ.

> **Phạm vi hiện tại:** tìm đường cho một cặp điểm nguồn–đích. Dự án chưa tối ưu thứ tự giao nhiều đơn, tải trọng xe, khung giờ giao hàng hoặc tình trạng giao thông theo thời gian thực.

## 1. Bài toán, mục tiêu và đầu vào/đầu ra

### Bài toán
Cho trước mạng đường bộ trong khu vực Long Biên và hai nút nguồn, đích trên mạng, tìm đường đi có tổng chiều dài nhỏ nhất từ nguồn đến đích.

### Mục tiêu
- Xây dựng mô hình đồ thị từ dữ liệu đường bộ.
- Cài đặt và so sánh Dijkstra và A* trên cùng các cặp điểm.
- Đánh giá chiều dài tuyến, số nút được mở rộng và thời gian chạy.
- Trực quan hóa tuyến và cung cấp ô nhập ID nút nguồn, đích để thử nghiệm.

### Đầu vào
- Đồ thị đường bộ Long Biên lưu trong `longbien_map.graphml`.
- ID nút nguồn `S` và ID nút đích `T` có trong đồ thị.
- Trọng số cạnh là chiều dài đoạn đường, tính bằng mét (`length`).

### Đầu ra
- Danh sách nút cấu thành tuyến đường.
- Tổng chiều dài tuyến (mét).
- Số nút được mở rộng và thời gian chạy (ms) của mỗi thuật toán.
- Hình bản đồ thể hiện tuyến tìm được.

## 2. Dữ liệu, nguồn, giấy phép và giới hạn

- **Khu vực:** Long Biên, Hà Nội, Việt Nam.
- **Nguồn theo notebook:** OpenStreetMap, được lấy bằng OSMnx với `network_type='drive'` và lưu ra GraphML. File GraphML đi kèm là bản dữ liệu đã lưu để chương trình có thể chạy mà không cần tải lại bản đồ.
- **Kích thước theo kết quả notebook:** 4.225 nút và 10.455 cạnh.
- **Thuộc tính được dùng:** tọa độ nút `x`, `y`; chiều dài cạnh `length`. GraphML còn có một số thuộc tính đường khác như tên đường, loại đường và một chiều.
- **Giấy phép:** dữ liệu OpenStreetMap được cung cấp theo Open Database License (ODbL). Khi phân phối hoặc công bố dữ liệu/bản dẫn xuất, cần tuân thủ điều kiện ghi công và các điều khoản áp dụng của ODbL. Hãy giữ thông tin nguồn và ngày tải dữ liệu trong bản nộp.
- **Giới hạn dữ liệu:** dữ liệu biểu diễn mạng đường được trích xuất tại một thời điểm, không cung cấp tình trạng ùn tắc trực tiếp. Kết quả tối ưu theo chiều dài, không phải thời gian giao hàng thực tế. Độ đầy đủ và chính xác phụ thuộc vào dữ liệu bản đồ tại khu vực.

Nguồn tham khảo: [OpenStreetMap Copyright and License](https://www.openstreetmap.org/copyright) và [OSMnx documentation](https://osmnx.readthedocs.io/).

## 3. Mô hình hóa trạng thái, đặc trưng và chi phí

Biểu diễn mạng đường dưới dạng đồ thị có hướng:
- **Nút (node):** vị trí giao nhau hoặc điểm hình học trên mạng đường; lưu tọa độ kinh độ `x` và vĩ độ `y`.
- **Cạnh (edge):** đoạn đường nối hai nút; đồ thị có hướng giúp biểu diễn các đường một chiều.
- **Trạng thái tìm kiếm:** nút hiện tại trong hành trình từ `S` đến `T`.
- **Chi phí cạnh:** `length` (mét). Chi phí của một tuyến là tổng chiều dài các cạnh trên tuyến.
- **Heuristic của A\*:** khoảng cách đường chim bay Haversine từ nút đang xét đến nút đích, tính bằng mét.

Với tiêu chí chiều dài, mục tiêu là tối thiểu hóa tổng trọng số cạnh trên đường đi. Heuristic khoảng cách đường chim bay được dùng để ưu tiên mở rộng các nút gần đích.

## 4. Thuật toán

### Dijkstra
Mở rộng nút có tổng chi phí từ nguồn nhỏ nhất. Thuật toán tìm đường tối ưu khi trọng số cạnh không âm; notebook dùng chiều dài đường làm trọng số.

### A*
Ưu tiên nút theo `f(n) = g(n) + h(n)`, trong đó `g(n)` là chiều dài đã đi và `h(n)` là khoảng cách Haversine đến đích. Với dữ liệu chiều dài đường và heuristic đường chim bay, A* hướng tìm kiếm về phía đích trong khi vẫn tìm tuyến ngắn.

Hai thuật toán được chạy trên cùng đồ thị và cặp điểm. Notebook dùng `random.seed(2026)` để tạo 10 cặp có đường đi, nhờ đó lần chạy có thể tái lập với cùng dữ liệu và môi trường.

## 5. Cấu trúc và cách chạy

Đặt các tệp cùng thư mục:

```text
project/
├── main_final.ipynb
├── longbien_map.graphml
└── README.md
```

Notebook hiện tìm file tên chính xác là `longbien_map.graphml`. Nếu file tải về có tên `longbien_map (1).graphml`, hãy đổi tên thành `longbien_map.graphml` hoặc sửa biến `filename` ở ô đầu tiên.

### Cài đặt
Khuyến nghị chạy bằng Jupyter Notebook hoặc JupyterLab. Cài các thư viện được notebook sử dụng:

```bash
pip install osmnx networkx matplotlib ipywidgets jupyter
```

### Chạy
1. Đặt notebook và GraphML cùng thư mục, đảm bảo tên file khớp như trên.
2. Mở `main_final.ipynb` bằng Jupyter.
3. Chạy lần lượt các ô từ trên xuống.
4. Dùng widget ở ô cuối để nhập ID nút nguồn và đích rồi chọn nút chạy.

Notebook kiểm tra ID có trong đồ thị và có đường đi trước khi chạy. Nếu tải bản đồ mới từ OpenStreetMap, cần có kết nối mạng; nếu dùng GraphML đã tải kèm thì không cần tải dữ liệu lại.

## 6. Thiết kế và chức năng hiện có

- Tải mạng đường từ GraphML; nếu không tìm thấy file thì notebook thử tải lại khu vực bằng OSMnx.
- Tự cài đặt Dijkstra và A* bằng hàng đợi ưu tiên.
- Tạo 10 cặp kiểm thử ngẫu nhiên có đường đi.
- In chiều dài, số nút mở rộng và thời gian chạy cho mỗi thuật toán.
- Vẽ tuyến của ca thử đầu tiên.
- Cho phép nhập ID nguồn/đích qua giao diện `ipywidgets` và hiển thị so sánh cùng bản đồ tuyến A*.

## 7. Kế hoạch thực nghiệm và đánh giá

### Thiết lập đang có
- Đồ thị Long Biên được lưu trong GraphML.
- 10 cặp nguồn–đích được tạo với seed cố định `2026`.
- Mỗi cặp được chạy bằng cả Dijkstra và A*.

### Chỉ số
| Chỉ số | Ý nghĩa |
|---|---|
| Chiều dài tuyến (m) | Chi phí mục tiêu; dùng để đối chiếu tính tối ưu giữa hai thuật toán |
| Số nút mở rộng | Lượng trạng thái thuật toán đã xét |
| Thời gian chạy (ms) | Thời gian thực thi trong môi trường chạy notebook |

Lập bảng cho 10 cặp điểm, tính thời gian trung bình và số nút mở rộng trung bình của mỗi thuật toán. Kiểm tra chiều dài tuyến do hai thuật toán trả về có bằng nhau hay không. Thời gian chạy có thể biến động theo máy, phiên chạy và tải hệ thống, vì vậy nên báo cáo môi trường thực nghiệm và tránh kết luận chỉ từ một lần đo. Notebook hiện có in kết quả trực tiếp, nhưng README không cố định số đo thời gian mẫu vì chúng phụ thuộc môi trường thực thi.

## 8. Hạn chế và hướng phát triển

- Đầu vào widget là ID nút, chưa phải địa chỉ hoặc tọa độ người dùng dễ chọn trên bản đồ.
- Bài toán hiện chỉ định tuyến một điểm giao mỗi lần, chưa giải bài toán nhiều điểm giao/VRP.
- Chi phí chỉ là chiều dài; chưa tính tốc độ, ùn tắc theo thời gian, chi phí nhiên liệu hay ưu tiên đường.
- Nên xử lý rõ trường hợp không tìm được đường trong hàm thuật toán trước khi dựng lại đường đi.
- Cần kiểm tra các cạnh song song trong đồ thị nhiều cạnh để luôn chọn trọng số phù hợp nhỏ nhất khi duyệt láng giềng.
- Nên bổ sung kiểm thử tự động cho tính hợp lệ của tuyến, tổng chiều dài và so sánh kết quả Dijkstra/A*.
- Hướng mở rộng: chọn điểm theo tọa độ gần nhất, hỗ trợ nhiều điểm giao, giới hạn tải trọng/khung giờ, thêm tiêu chí thời gian ước tính và giao diện bản đồ tương tác.

## 9. Kế hoạch bài tập lớn và phân công gợi ý

| Giai đoạn | Công việc | Sản phẩm |
|---|---|---|
| 1. Chốt đề bài | Xác định khu vực, một hay nhiều điểm giao, tiêu chí tối ưu, đầu vào/đầu ra | Mô tả đề tài một trang |
| 2. Dữ liệu | Ghi nguồn, ngày lấy, giấy phép, phạm vi và giới hạn dữ liệu | GraphML và mô tả dữ liệu |
| 3. Mô hình | Định nghĩa nút, cạnh, trọng số, heuristic và tiêu chí đánh giá | Phần mô hình hóa trong báo cáo |
| 4. Cài đặt | Hoàn thiện nạp dữ liệu, Dijkstra, A*, giao diện và xử lý lỗi | Notebook chạy được |
| 5. Thực nghiệm | Chạy các cặp kiểm thử, lập bảng và phân tích kết quả | Bảng/biểu đồ và nhận xét |
| 6. Hoàn thiện | Viết báo cáo, README, hướng dẫn chạy và chuẩn bị trình bày | Repo GitHub và báo cáo cuối kỳ |

Phân công có thể chia theo các phần: (1) dữ liệu và giấy phép, (2) thuật toán và mô hình, (3) giao diện và trực quan hóa, (4) thực nghiệm, báo cáo và tích hợp. Nếu nhóm ít thành viên, có thể gộp các phần và ghi rõ người phụ trách trong báo cáo.

## 10. Gợi ý nội dung repo GitHub

```text
road-delivery-routing/
├── README.md
├── main_final.ipynb
├── data/
│   └── README.md             # nguồn dữ liệu, giấy phép, cách lấy GraphML
├── reports/
│   ├── proposal.md           # mô tả đề tài một trang
│   └── experiments.md        # thiết lập và kết quả thử nghiệm
├── requirements.txt
└── .gitignore
```

Nếu không muốn đưa GraphML vài MB lên GitHub, hướng dẫn trong `data/README.md` cách tải/tạo lại tệp và lưu dữ liệu ngoài repo. Khi dùng lại dữ liệu OpenStreetMap, giữ ghi công và tuân thủ ODbL. Không đưa token, thông tin cá nhân hoặc tệp môi trường vào repo.

## Ghi công dữ liệu

© OpenStreetMap contributors. Dữ liệu bản đồ được cung cấp theo [ODbL](https://www.openstreetmap.org/copyright). Dữ liệu dùng trong đồ án là bản trích xuất mạng đường khu vực Long Biên, Hà Nội; ngày trích xuất cụ thể nên được nhóm bổ sung nếu biết.
