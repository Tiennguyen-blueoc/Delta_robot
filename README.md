# Mô phỏng Delta Robot bằng Python

Mô phỏng động học vị trí của Delta robot 3 bậc tự do bằng `math`, NumPy và Matplotlib 3D. Có hoạt ảnh chuyển động, đường đi của tâm bàn động, ba góc khớp và sai số kiểm tra `FK(IK(P))`.

## Chạy trên máy hiện tại

Môi trường `.venv` đã được chuẩn bị với Python Windows, NumPy, Matplotlib và Pillow. Nhấp đúp **`run.bat`**, hoặc chạy trong PowerShell tại thư mục dự án:

```powershell
.\.venv\Scripts\python.exe start.py
```

Môi trường `delta_robot/` có sẵn là Python MSYS2. Dự án dùng `.venv/` riêng với Python Windows để tương thích các thư viện đồ họa; giữ nguyên môi trường MSYS2.

## Cài mới trên máy khác

Dùng Python 3.10 trở lên (bản Windows chính thức có Tcl/Tk):

```powershell
py -3 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe start.py
```

Trên Linux/macOS: `python3 -m venv .venv`, dùng `.venv/bin/python` cho các lệnh tương tự. Linux cần backend GUI như Tk (`python3-tk` trên Ubuntu) để mở cửa sổ. Chế độ `--check` chỉ cần thư viện chuẩn Python.

## Sử dụng giao diện

- **Quỹ đạo**: chọn Tròn, Số 8, Vuông hoặc Sóng 3D. Thanh trượt chỉnh bán kính, cao độ Z và tốc độ phát lại. Quỹ đạo được kiểm tra toàn bộ trước khi chạy.
- **IK / XYZ**: kéo X, Y, Z để tính ba góc động cơ bằng động học nghịch, sau đó tính lại XYZ bằng động học thuận.
- **FK / Góc**: kéo θ1, θ2, θ3 để tính và hiển thị vị trí bàn động bằng động học thuận.
- **Tạm dừng / Tiếp tục**, **Chạy lại** hoặc phím **Space**, **R** điều khiển hoạt ảnh. Kéo chuột trong hình 3D để xoay góc nhìn.
- **Lưu CSV** ghi quỹ đạo hiện tại vào `outputs/motion.csv` (ghi đè tệp cùng tên). Trong chế độ thủ công, nút lưu một tư thế.
- Điểm không tới được hoặc góc vượt giới hạn sẽ hiển thị thông báo đỏ và giữ tư thế hợp lệ gần nhất.

Các thanh màu xanh ngọc, cam và tím tương ứng ba nhánh. Mỗi tay dưới được vẽ bằng hai thanh song song. Nét đứt là quỹ đạo đặt; đường xanh ngọc là đường đã đi, tính bằng FK. Biểu đồ bên phải thể hiện góc khớp của toàn chu kỳ, với vạch đứng chỉ thời điểm hiện tại; chế độ thủ công hiển thị ba góc hiện tại.

## Lệnh mẫu

```powershell
# Chạy quỹ đạo số 8, chu kỳ 6 giây
.\.venv\Scripts\python.exe start.py --path figure8 --duration 6

# Kiểm tra và xuất dữ liệu; không mở cửa sổ
.\.venv\Scripts\python.exe start.py --path wave --check --export-csv outputs/wave.csv

# Phát lại các điểm từ CSV, dàn đều theo thời gian 10 giây
.\.venv\Scripts\python.exe start.py --replay outputs/wave.csv --duration 10

# Lưu ảnh hoặc hoạt ảnh, không cần cửa sổ GUI
.\.venv\Scripts\python.exe start.py --path wave --preview assets/preview.png
.\.venv\Scripts\python.exe start.py --path figure8 --duration 4 --fps 15 --gif outputs/delta.gif

# Xem các tham số thay đổi kích thước robot
.\.venv\Scripts\python.exe start.py --help

# Chạy toàn bộ kiểm thử
.\.venv\Scripts\python.exe -m unittest discover -s tests -v
```

CSV đầu vào cần tiêu đề `x_mm,y_mm,z_mm`. Các hàng là các điểm liên tiếp; toàn bộ điểm phải tới được. Tệp xuất còn chứa thời gian `t_s`, ba góc khớp (độ), tọa độ từ FK và sai số (mm). Khi nhập lại, chương trình dùng các cột XYZ và dàn đều các mẫu trong `--duration`; cột thời gian và góc cũ không được dùng làm lệnh điều khiển. Quỹ đạo kín lặp lại; đường hở dừng ở điểm cuối. Đổi quỹ đạo mẫu trên giao diện sẽ thay thế dữ liệu CSV đang phát.

## Mô hình và quy ước

| Tham số | Mặc định | Ý nghĩa |
|---|---:|---|
| `base_radius` | 150 mm | Khoảng cách từ tâm đế tới tâm khớp động cơ |
| `platform_radius` | 45 mm | Khoảng cách từ tâm bàn động tới khớp tay dưới |
| `upper_arm` | 150 mm | Chiều dài tay trên |
| `lower_arm` | 330 mm | Chiều dài mỗi thanh tay dưới |
| Góc khớp | −60° đến 90° | Giới hạn áp dụng cho cả IK và FK |

Gốc tọa độ ở tâm đế, mặt đế là `Z=0`, bàn động nằm ở `Z<0`. Góc 0° ứng với tay trên nằm ngang hướng ra ngoài; góc dương làm tay trên hạ xuống. Ba nhánh cách nhau 120°, lần lượt có phương vị −90°, 30°, 150°. Các bán kính ở đây là bán kính tâm khớp, không phải chiều dài cạnh tam giác.

Gọi `uᵢ` là vectơ đơn vị hướng tâm nhánh i, `k=(0,0,1)`, `P=(x,y,z)`, `Rb` và `Rp` là hai bán kính, `L` và `l` là chiều dài tay trên/dưới:

```text
Khớp đế:       Bᵢ = Rb uᵢ
Khớp khuỷu:    Eᵢ = Bᵢ + L cos(θᵢ) uᵢ − L sin(θᵢ) k
Khớp bàn động: Qᵢ = P + Rp uᵢ
Ràng buộc:     ||Qᵢ − Eᵢ||² = l²
```

**IK**: đặt `dᵢ=P+(Rp−Rb)uᵢ`, khai triển ràng buộc thành:

```text
A cos(θᵢ) + B sin(θᵢ) = C
A = dᵢ · uᵢ
B = −z
C = (dᵢ · dᵢ + L² − l²) / (2L)
θᵢ = atan2(B,A) − acos(C / sqrt(A²+B²))
```

Chọn nhánh khuỷu hướng ra ngoài, kiểm tra giới hạn góc và xác nhận lại bằng FK. Các hàm `math` tính bằng radian nội bộ; API trả góc theo độ.

**FK**: ba tâm cầu tương đương là `Cᵢ=Eᵢ−Rp uᵢ`, cùng bán kính `l`. Tính giao ba mặt cầu bằng cơ sở trực chuẩn, chọn nghiệm có Z thấp hơn. Phát hiện các tâm cầu trùng/thẳng hàng, hai nghiệm tiếp xúc và trường hợp không có nghiệm thực.

Sai số hiển thị là `||P_đặt − FK(IK(P_đặt))||`, dùng kiểm chứng số học, không đại diện sai số điều khiển hoặc sai số cơ khí. Ở chế độ FK, vị trí đặt được lấy bằng vị trí tính được, nên sai số này bằng 0 theo định nghĩa.

Đây là mô hình động học lý tưởng: bàn động luôn song song với đế, không mô phỏng lực, mô-men, khối lượng, độ rơ, va chạm hoặc giới hạn góc của khớp thụ động. Tốc độ phát lại là tốc độ thời gian mô phỏng, không phải bộ lập kế hoạch vận tốc/gia tốc động cơ; quỹ đạo vuông có góc gấp tại đỉnh. Hoạt ảnh lấy mẫu tư thế theo thời gian, tốc độ hiển thị thực tế tùy máy.

## Cấu trúc mã nguồn

- `start.py`: lệnh chạy, tham số hình học và xuất dữ liệu.
- `delta_sim/kinematics.py`: động học thuận/nghịch và tọa độ các khớp, chỉ dùng thư viện chuẩn.
- `delta_sim/trajectory.py`: tạo quỹ đạo, kiểm tra khả đạt, đọc/ghi CSV.
- `delta_sim/app.py`: dựng robot 3D, hoạt ảnh, thanh trượt và biểu đồ góc khớp.
- `tests/`: kiểm thử nghiệm hình học độc lập, IK/FK, quỹ đạo, CSV và tương tác giao diện.

Tham khảo mô hình ràng buộc chiều dài và giao ba mặt cầu: [R. L. Williams II, Ohio University — The Delta Parallel Robot: Kinematics Solutions](https://people.ohio.edu/williams/html/PDF/DeltaKin.pdf). Tài liệu hoạt ảnh và điều khiển: [Matplotlib FuncAnimation](https://matplotlib.org/stable/api/_as_gen/matplotlib.animation.FuncAnimation.html), [Matplotlib widgets](https://matplotlib.org/stable/api/widgets_api.html).

## Demo

![Mô phỏng robot Delta](outputs/delta_demo.gif)
