# Ecommerce Orders 2026 — Phân tích SQL (Notebook)

Một notebook phân tích dữ liệu đơn hàng thương mại điện tử (E-commerce) tập trung vào truy vấn SQL, kiểm tra chất lượng dữ liệu và trả lời các câu hỏi kinh doanh chính (doanh thu, sản phẩm bán chạy, phân tích khách hàng, vận chuyển). Notebook dùng Python (pandas, SQLAlchemy) làm cầu nối để chạy SQL trên SQL Server, lấy kết quả và trực quan hóa.

---

## Mục lục
- [Tổng quan ngắn](#tổng-quan-ngắn)  
- [Điểm nổi bật trong notebook](#điểm-nổi-bật-trong-notebook)  
- [Cấu trúc repo](#cấu-trúc-repo)  
- [Môi trường & phụ thuộc](#môi-trường--phụ-thuộc)  
- [Cách chạy (Quickstart)](#cách-chạy-quickstart)  
- [Cấu hình kết nối an toàn (Env vars)](#cấu-hình-kết-nối-an-toàn-env-vars)  
- [Lưu ý an toàn & hiện trạng destructive cells](#lưu-ý-an-toàn--hiện-trạng-destructive-cells)  
- [Schema tóm tắt (các cột chính)](#schema-tóm-tắt-các-cột-chính)  
- [Truy vấn SQL mẫu](#truy-vấn-sql-mẫu)  
- [Kết quả sơ bộ / Insights nhanh](#kết-quả-sơ-bộ--insights-nhanh)  
- [Next steps gợi ý](#next-steps-gợi-ý)  
- [Đóng góp & License](#đóng-góp--license)  
- [Liên hệ](#liên-hệ)

---

## Tổng quan ngắn
Notebook chính `Ecommerce_Orders_2026_SQL.ipynb` thực hiện: kết nối tới SQL Server, kiểm tra schema của bảng `[dbo].[ecommerce_orders_dataset]`, tiền xử lý (bao gồm mapping tên cột tiếng Việt để hiển thị), EDA (exploratory data analysis) và một loạt truy vấn SQL mẫu phục vụ báo cáo/dashboard.

---

## Điểm nổi bật trong notebook
- Kết nối tới SQL Server bằng SQLAlchemy + ODBC Driver.
- Đổi tên cột hiển thị sang tiếng Việt (snake_case) để người đọc dễ theo dõi.
- Có cell thực thi `sp_rename` để đổi tên cột trực tiếp trên DB (thao tác destructive — xem phần lưu ý).
- Notebook đọc schema, hiển thị sample (5 dòng), xác định grain (mỗi dòng = 1 đơn hàng) và thực hiện nhiều phép phân tích (số lượng khách hàng, số sản phẩm, phân bố ngày, return rate, LTV...).

---

## Cấu trúc repo
- Ecommerce_Orders_2026_SQL.ipynb — Notebook chính chứa toàn bộ phân tích và truy vấn.
- README.md — file hướng dẫn (bạn đang đọc).

---

## Môi trường & phụ thuộc
Yêu cầu tối thiểu:
- Python 3.8+
- JupyterLab / Jupyter Notebook

Thư viện gợi ý:
- pandas
- sqlalchemy
- pyodbc
- matplotlib, seaborn
- ipython-sql (tùy chọn)

Cài nhanh:
```bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install pandas sqlalchemy pyodbc matplotlib seaborn jupyterlab
jupyter lab
```

---

## Cách chạy (Quickstart)
1. Clone repo.
2. Tạo virtualenv và cài dependencies như trên.
3. Mở `Ecommerce_Orders_2026_SQL.ipynb` bằng Jupyter.
4. Trước khi chạy, cấu hình kết nối (xem phần "Cấu hình kết nối an toàn").
5. Chạy từng cell theo thứ tự; đọc chú thích for each section.

Lưu ý ngắn: không chạy cell đổi tên cột (sp_rename) trên database production nếu bạn không chắc chắn — cell này thay đổi schema thực tế.

---

## Cấu hình kết nối an toàn (Env vars)
Notebook hiện có các biến DB hardcoded (DB_USER, DB_PASSWORD, DB_HOST, DB_NAME). Thay bằng biến môi trường trước khi chạy:
```python
import os
DB_USER = os.getenv("ECO_DB_USER")
DB_PASSWORD = os.getenv("ECO_DB_PASSWORD")
DB_HOST = os.getenv("ECO_DB_HOST")
DB_PORT = os.getenv("ECO_DB_PORT", "1433")
DB_NAME = os.getenv("ECO_DB_NAME")
```
Hoặc dùng file `.env` (không commit) và load bằng python-dotenv.

Chuỗi kết nối mẫu (ví dụ cho SQL Server + ODBC):
```
DRIVER={ODBC Driver 17 for SQL Server};SERVER=<host>,<port>;DATABASE=<db>;UID=<user>;PWD=<password>;Encrypt=yes;TrustServerCertificate=yes;
```

---

## Lưu ý an toàn & hiện trạng destructive cells
- Credentials hardcoded: hiện có thông tin DB trong notebook. Cần xóa trước khi public hoặc chuyển sang biến môi trường.
- Cell dùng `sp_rename` — đổi tên cột trực tiếp trên database:
  - Đây là thao tác destructive (thay đổi schema). Chỉ chạy nếu bạn muốn đổi schema thật.
  - Nếu mục tiêu chỉ hiển thị tên cột tiếng Việt trong notebook, hãy tắt cell này và dùng mapping trong pandas (rename khi đọc).
- Nếu bạn muốn, mình có thể sửa notebook để:
  - Lấy credential từ env vars.
  - Vô hiệu hóa/đổi cell renaming thành non-destructive (chỉ rename trên DataFrame).

---

## Schema tóm tắt (các cột chính)
Notebook liệt kê 41 cột. Dưới đây là danh sách tóm tắt (tên cột tiếng Việt — kiểu dữ liệu như trong notebook):

- ma_don_hang (int)
- ma_khach_hang (nvarchar)
- ngay_dat_hang (date)
- nam (smallint), thang (tinyint), ngay (tinyint), quy (tinyint)
- thu_trong_tuan (nvarchar), mua (nvarchar), mua_le_hoi (bit)
- tuoi_khach_hang (tinyint), gioi_tinh_khach_hang (nvarchar)
- quoc_gia (nvarchar), thanh_pho (nvarchar)
- phan_khuc_khach_hang (nvarchar), trang_thai_thanh_vien (nvarchar)
- ma_san_pham (nvarchar), nhom_san_pham (nvarchar), phan_nhom_san_pham (nvarchar), thuong_hieu (nvarchar)
- don_gia (float), so_luong (tinyint)
- ty_le_giam_gia (tinyint), tien_giam_gia (float), da_dung_ma_giam_gia (bit)
- chi_phi_van_chuyen (float), phuong_thuc_van_chuyen (nvarchar), khu_vuc_kho (nvarchar), so_ngay_giao_hang (tinyint)
- tien_thue (float), tong_tien (float)
- phuong_thuc_thanh_toan (nvarchar)
- loai_thiet_bi (nvarchar), nguon_truy_cap (nvarchar)
- trang_thai_don_hang (nvarchar), da_tra_hang (bit)
- danh_gia (float)
- gia_tri_vong_doi_khach_hang (float), bien_loi_nhuan (float), loi_nhuan (float)
- don_hang_gia_tri_cao (bit)

(Chi tiết kiểu dữ liệu đầy đủ nằm trong notebook; bạn có thể xuất schema ra file CSV nếu cần.)

---

## Truy vấn SQL mẫu
Một vài truy vấn mẫu từ notebook — chỉnh tên bảng/cột nếu cần:

1) Doanh thu theo tháng
```sql
SELECT
  DATE_TRUNC('month', order_date) AS month,
  COUNT(DISTINCT order_id) AS orders_count,
  SUM(total_amount) AS revenue
FROM orders
WHERE order_date BETWEEN '2026-01-01' AND '2026-12-31'
GROUP BY month
ORDER BY month;
```

2) Top 10 sản phẩm theo số lượng bán
```sql
SELECT
  p.product_id,
  p.name,
  SUM(oi.quantity) AS total_quantity_sold,
  SUM(oi.quantity * oi.unit_price) AS revenue
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_date >= '2026-01-01'
GROUP BY p.product_id, p.name
ORDER BY total_quantity_sold DESC
LIMIT 10;
```

3) Tỷ lệ hoàn đơn theo category (ví dụ logic)
```sql
WITH sold AS (
  SELECT p.category, COUNT(DISTINCT o.order_id) AS sold_orders
  FROM orders o
  JOIN order_items oi ON o.order_id = oi.order_id
  JOIN products p ON oi.product_id = p.product_id
  WHERE o.order_date >= '2026-01-01'
  GROUP BY p.category
),
returned AS (
  SELECT p.category, COUNT(DISTINCT r.order_id) AS returned_orders
  FROM returns r
  JOIN order_items oi ON r.order_item_id = oi.order_item_id
  JOIN products p ON oi.product_id = p.product_id
  WHERE r.return_date >= '2026-01-01'
  GROUP BY p.category
)
SELECT
  s.category,
  s.sold_orders,
  COALESCE(r.returned_orders, 0) AS returned_orders,
  ROUND(100.0 * COALESCE(r.returned_orders, 0) / s.sold_orders, 2) AS return_rate_percent
FROM sold s
LEFT JOIN returned r ON s.category = r.category
ORDER BY return_rate_percent DESC;
```

---

## Kết quả sơ bộ / Insights nhanh (từ notebook)
- Tổng dòng: 30.000 (mỗi dòng tương ứng 1 đơn hàng).
- Số khách hàng: ~8.683.
- Số sản phẩm: ~2.500.
- Dữ liệu mẫu hiển thị bắt đầu từ 2023-01-01.
- Một số trường đáng lưu ý: `danh_gia` có giá trị lớn (không theo thang 1–5), cần kiểm tra đơn vị/scale; `gia_tri_vong_doi_khach_hang` đã tính sẵn cho từng khách.

Lưu ý: những insights chi tiết hơn cần chạy toàn bộ notebook trong môi trường của bạn và kiểm tra các phân tích (cohort, retention, LTV).

---

## Next steps gợi ý
1. Thay credential bằng env vars; remove hardcoded credentials.
2. Vô hiệu hóa/đổi cell `sp_rename` thành non-destructive (chỉ rename trong DataFrame).
3. Tách notebook lớn thành nhiều notebook nhỏ theo mục đích (01_overview, 02_cleaning, 03_sales, 04_customers, 05_products, 06_reporting).
4. Thêm file LICENSE và CONTRIBUTING.md nếu muốn mở hợp tác.
5. (Tuỳ chọn) Cung cấp snapshot dữ liệu anonymized (CSV) để người khác có thể chạy offline.

---

## Đóng góp & License
- Quy trình đóng góp: fork → tạo branch → PR. Ghi rõ mục tiêu PR & test (nếu có).
- License: chưa có file license; hãy thêm `LICENSE` (ví dụ MIT) nếu muốn công khai.

---

## Liên hệ
- Tác giả repo: sonbuwin-beep (GitHub).

---
