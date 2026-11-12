# Báo cáo thực hành Day 3 — Image Segmentation

- Mã học viên: **2A202603176**.
- Họ và tên: **Bùi Kim Trai**.
- Ngày tổng hợp bài: **12/11/2026**; CVAT local trên Windows/WSL2.
- Công cụ đã dùng: Polygon trong CVAT; SAM 2.1 Hiera Small; Faster R-CNN của torchvision để đề xuất vị trí/lớp; SegFormer B0 Cityscapes để gợi ý vùng semantic; trợ lý AI hỗ trợ thao tác, kiểm tra và xuất dữ liệu.
- Phạm vi tự làm và hỗ trợ: Easy do người học tự vẽ. Ở Medium, người học tự vẽ các polygon ban đầu cho người và xe; sau đó dùng AI bổ sung các vật còn thiếu. Hard và sáu checkpoint dùng công cụ và trợ lý AI để hỗ trợ tạo, kiểm tra và sửa mask.

## 1. Bài đã nộp

Bảng dưới ghi các ZIP đã xuất từ CVAT và chuẩn bị trong `submissions/`. Trạng thái này không tự xác nhận đã nộp link trên LMS. Số ảnh có chú thích và đã Save không đồng nghĩa mọi pixel đã được chấm đúng.

| Task | File ZIP đúng tên | Ảnh có chú thích đã lưu | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | **9 ZIP; 14 ảnh** | | **100** |

Semantic xuất Segmentation mask 1.1; instance và panoptic xuất COCO 1.0. Cả 9 ZIP được tạo bằng bộ xuất của CVAT, không sửa JSON/PNG trong ZIP sau khi xuất. Bộ kiểm tra cấu trúc của repo báo không thiếu task và không có lỗi cấu trúc; Hard có nhắc kiểm trực quan độ phủ/chồng lấn. Chưa dùng ground truth hoặc có điểm so reference; không tự ghi PASS, bonus hay điểm chính thức.

## 2. Một quyết định trước khi dùng gợi ý

- Object đầu tiên tự vẽ ở Medium: **người phụ nữ mặc áo dài đứng giữa ảnh `000000181542.jpg`**, class **person**. Người học xác nhận đây là object đã tự vẽ trước khi dùng AI.
- Polygon đi theo phần cơ thể và áo dài nhìn thấy, phân biệt đường biên với xe máy và nền xung quanh; chỉ gán vùng nhìn thấy, không nối xuyên vật che. Kết quả trước khi bổ sung AI đã được sao lưu.
- Easy được người học tự vẽ toàn bộ; Medium có phần người và xe tự vẽ. Trong lần hỗ trợ đầu, 16 polygon đang có được giữ nguyên và bổ sung 65 mask AI. Sau đó người học bổ sung thêm 2 polygon thủ công; bản Medium mới nhất có **83 đối tượng**, gồm **18 polygon thủ công và 65 mask AI**.
- Khi dùng gợi ý, không giữ tất cả đề xuất: đã loại các đề xuất trùng một phần cùng ô tô và đề xuất xe đẩy trẻ em bị nhận thành motorcycle. Các thao tác này được thực hiện trong lượt kiểm có công cụ và trợ lý AI hỗ trợ.

## 3. Một lỗi tôi tìm thấy và sửa

Ca sửa lỗi trong lượt kiểm có **trợ lý AI hỗ trợ**:

- Task/ảnh/vùng: **cp6_coverage**, ảnh **`7daa6479-67988f3f.jpg`**, chiếc bus lớn ở giữa ảnh, mang số 2555.
- Loại lỗi: **sai lớp**; mô hình semantic ban đầu tô vùng thân bus vào **car**.
- Bằng chứng quan sát: phương tiện có thân xe buýt cao, rộng, cửa kính phía sau và chữ “New York City Bus”; không phải ô tô con.
- Quy tắc: chỉ dùng các lớp trong `classes.json` của cp6. Task có car nhưng không có bus, nên không ép bus thành car để tăng coverage.
- Hành động: trợ lý loại vùng bus nhận nhầm khỏi mask car, kiểm ảnh phủ màu, lưu lại trong CVAT và xuất lại **cp6_coverage.zip**. Phần nằm ngoài danh sách lớp được để ngoài các mask mục tiêu.
- Kiểm tra sau sửa: ZIP đã xuất lại và đọc được bằng bộ kiểm cấu trúc của repo. Chưa có metric/điểm với ground truth; điểm tin cậy của mô hình không được coi là IoU hoặc điểm chấm.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

Ba ca dưới đây được tổng hợp từ những vùng thực tế đã xem trong lượt kiểm có AI hỗ trợ.

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| cp1_holes, `000000144300.jpg`, motorcycle và các chi tiết bên trong | Khoét các vùng trông giống lỗ; hoặc giữ silhouette theo yêu cầu checkpoint | Đề cp1 yêu cầu kính/khe nằm trong mask, không khoét tùy tiện | Dùng mask theo quy tắc cp1; cần phân biệt chi tiết của vật với nền thật khi xem ở mức phóng to |
| Hard, `000000460147.jpg`, xe chuyên chở và các ô tô trên xe | Gộp toàn bộ thành một truck; hoặc tách xe chuyên chở và từng car nhìn thấy | Instance phân biệt từng vật, không chỉ từng lớp; có nhiều ô tô riêng nằm trên xe chuyên chở | Giữ từng car là một instance; phần truck còn nhìn thấy là instance riêng, không vẽ xuyên vùng che và kiểm chồng lấn |
| cp6_coverage, `7daa6479-67988f3f.jpg`, bus giữa ảnh | Gán bus thành car để kín ảnh; hoặc để ngoài các lớp mục tiêu | Danh sách lớp cp6 không có bus; hình dáng và chữ trên phương tiện cho thấy đó là bus | Không dùng car thay bus. Nếu cần phủ cả phương tiện ngoài schema, hỏi coach thay vì tự thêm lớp hoặc ép nhãn |

Hạn chế còn lại: các vật rất nhỏ ở xa, chuyển động nhòe, đường biên cột/biển mảnh và ranh road–sidewalk có thể chưa chính xác tuyệt đối. Kiểm cấu trúc chỉ xác nhận ảnh/lớp/định dạng ZIP, không chứng minh chất lượng hình học. Ground truth chưa được tải hoặc dùng trong lượt hỗ trợ này.

Nguồn: hướng dẫn, RULES, RUBRIC và các `classes.json` trong repo bài lab; mã nguồn chính thức [SAM 2](https://github.com/facebookresearch/sam2), mô hình torchvision [Faster R-CNN ResNet50 FPN v2](https://docs.pytorch.org/vision/main/models/generated/torchvision.models.detection.fasterrcnn_resnet50_fpn_v2.html), mô hình NVIDIA [SegFormer B0 Cityscapes](https://huggingface.co/nvidia/segformer-b0-finetuned-cityscapes-1024-1024).
