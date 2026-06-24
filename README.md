# Chuyển Giao Tri Thức Tiên Nghiệm Xuất Huyết Não Xuyên Bộ Dữ Liệu Trong Phân Đoạn Phù Não Quanh Ổ Xuất Huyết Trên Ảnh CT

Đánh giá hiệu quả của việc chuyển giao tri thức không gian từ vùng xuất huyết nội sọ (**ICH Prior**) sang phân đoạn phù não quanh ổ xuất huyết (**PHE**) trên ảnh CT sọ não không tiêm thuốc.

---

## 🔬 Câu hỏi nghiên cứu cốt lõi
> **Liệu thông tin tiên nghiệm không gian học từ các bộ dữ liệu xuất huyết nội sọ (ICH) ngoài có giúp cải thiện hiệu năng phân đoạn phù não quanh ổ xuất huyết (PHE) trên ảnh CT không tiêm thuốc hay không?**

---

## 🏆 Đóng góp chính của đề tài

1. **Khung đánh giá đối chứng xuyên miền dữ liệu (Cross-dataset evaluation):** Thiết lập một pipeline hoàn chỉnh đánh giá việc chuyển giao tri thức từ các bộ dữ liệu xuất huyết ngoài (`Instance2022`, `Seg-CQ500`) sang bài toán đích PHE trên tập dữ liệu `PHE-SICH-CT-IDS`.
2. **Thiết kế đa dạng kênh biểu diễn tiên nghiệm (Prior Representation):** Khảo sát và so sánh các dạng mã hóa thông tin không gian khác nhau bao gồm mặt nạ nhị phân (Binary Mask), bản đồ xác suất mềm (Soft Probability Map) và bản đồ biến đổi khoảng cách Euclidean (Distance Transform Map).
3. **Xây dựng nhánh đối chứng Heuristic vật lý (Pseudo-Prior Control):** Đề xuất thuật toán sinh prior tự động từ ảnh CT gốc bằng ngưỡng cường độ vật lý và hình thái học, làm mốc đối chứng mạnh để cô lập và làm rõ nguồn đóng góp thực sự của mô hình học sâu (Deep Learning).
4. **Đánh giá benchmark toàn diện các mạng phân đoạn y sinh:** Cung cấp kết quả so sánh định lượng trực tiếp giữa các kiến trúc mạng 3D SOTA bao gồm `nnU-Net`, `MedNeXt`, `DynUNet`, `SwinUNETR`, và `ResidualUNet3D`.

---

## 🛠 Phương pháp tiếp cận

```mermaid
graph LR
    CT[Ảnh CT gốc] --> Baseline[1. Baseline CT-Only]
    CT --> Teacher[2. Teacher-Prior]
    CT --> Pseudo[3. Pseudo-Prior Control]

    subgraph "Tri thức xuất huyết ngoài"
        Ext[Instance2022 + Seg-CQ500] -->|Pretrain| TModel[Mô hình ICH Teacher]
    end

    TModel -->|Predict| Teacher
    Teacher -->|Trích xuất| T_Prior["ICH Prob (P_ICH) & Khoảng cách (D_ICH)"]
    
    Pseudo -->|Ngưỡng HU + Otsu + Morph| P_Prior["Pseudo-ICH Prob & Khoảng cách"]

    Baseline -->|Đầu vào: CT| Seg[nnU-Net 3d_fullres]
    T_Prior & CT -->|Đầu vào: CT + Prior| Seg
    P_Prior & CT -->|Đầu vào: CT + Pseudo-Prior| Seg

    Seg -->|Đầu ra| PHE[Mặt nạ PHE]
```

### 1. Nhánh Baseline (CT-Only)
- Chỉ sử dụng ảnh CT gốc làm đầu vào. Đánh giá các kiến trúc mạng: `nnU-Net 3d_fullres`, `MedNeXt-S`, `MedNeXt-M`, `DynUNet`, `SwinUNETR`, `ResidualUNet3D`.

### 2. Nhánh Teacher-Prior (ICH-Prior)
- **Mô hình Teacher** (nnU-Net) được huấn luyện trên `Instance2022` và `Seg-CQ500` để nhận diện vùng xuất huyết (ICH).
- Trích xuất 2 kênh tiên nghiệm không gian trên tập PHE-SICH:
  - Bản đồ xác suất xuất huyết ($P_{\text{ICH}}$)
  - Bản đồ biến đổi khoảng cách Euclidean từ biên vùng xuất huyết ($D_{\text{ICH}} = \text{DT}(M_{\text{ICH}})$)
- Mô hình PHE downstream nhận đầu vào 3 kênh: `[CT, P_ICH, D_ICH]`.

### 3. Nhánh Pseudo-Prior Control
- Tạo vùng xuất huyết giả lập trực tiếp từ ảnh CT dựa trên heuristics vật lý:
  $$\text{Mask}_{\text{candidate}} = (\text{CT} \ge 55 \text{ HU}) \cap \text{Local-Otsu}$$
- Áp dụng phép toán hình thái học và sinh bản đồ khoảng cách để làm nhóm đối chứng không dùng học sâu.

---

## 📊 Phân chia dữ liệu thực nghiệm

| Bộ dữ liệu | Nhãn / Phân vùng | Số ca | Vai trò |
| :--- | :--- | :---: | :--- |
| **PHE-SICH-CT-IDS** | NCCT / PHE Mask | 120 | Bài toán đích PHE (99 Train / 10 Val / 11 Test) |
| **Instance2022** | NCCT / ICH Mask | 100 | Huấn luyện ngoài cho ICH Teacher |
| **Seg-CQ500** | NCCT / ICH Mask | 51 | Huấn luyện ngoài cho ICH Teacher |

---

## 📈 Kết quả định lượng

### 1. Đánh giá các backbone phân đoạn (Baseline CT-Only trên PHE-SICH Test Set)
| Kiến trúc mạng | Dice (Mean) | IoU (Mean) | Trạng thái |
| :--- | :---: | :---: | :---: |
| **nnU-Net 3d_fullres** | **0.3941** | **0.2710** | **Baseline mạnh nhất** |
| MedNeXt-S | 0.3627 | 0.2390 | Thấp hơn |
| MedNeXt-M | 0.3600 | 0.2356 | Overfitting |
| DynUNet | 0.3430 | 0.2242 | Thấp hơn |
| ResidualUNet3D | 0.2161 | 0.1294 | Thấp |
| SwinUNETR | 0.1163 | 0.0668 | Không hội tụ |

### 2. Hiệu năng mô hình ICH Teacher trên tập dữ liệu ngoài
| Phiên bản | Dice (Mean) | Dice (Median) | Precision | Recall | Sai số thể tích (Abs) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| ICH Teacher gốc | 0.7330 | 0.8548 | 0.7562 | 0.7598 | 4.117 ml |
| **ICH Teacher cải tiến** | **0.7758** | **0.8581** | **0.8107** | **0.7739** | **3.141 ml** |

### 3. Hiệu năng phân đoạn PHE Downstream (Tập Test)
| Cấu hình thực nghiệm | Dice (Mean) | Dice (Median) | IoU (Mean) | FP (Voxel) | FN (Voxel) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Baseline CT-Only** | 0.3941 | 0.3638 | 0.2710 | **1455.2** | 1127.2 |
| **Teacher-Prior (Binary)**| **0.3999** | **0.5021** | 0.2750 | 1876.0 | 1033.5 |
| **Teacher-Prior (Soft)** | 0.3974 | 0.4560 | 0.2686 | 1513.8 | 1196.4 |
| **Pseudo-Prior (Raw)** | 0.3995 | 0.3699 | 0.2725 | 1354.5 | 1157.5 |
| **Pseudo-Prior (Cải tiến)**| 0.3978 | 0.4339 | **0.2769** | 2324.5 | **899.9** |

---

## 🧪 Kết luận khoa học chính

1. **nnU-Net chiếm ưu thế vượt trội khi dữ liệu nhỏ:** Framework tự cấu hình `nnU-Net 3d_fullres` đạt độ ổn định và kết quả tốt nhất. Các kiến trúc Transformer (SwinUNETR) bị overfit nghiêm trọng.
2. **Sự đánh đổi về độ nhạy của Prior:** Tích hợp tiên nghiệm không gian giúp ổn định phân đoạn các ca khó (Dice Median tăng từ **0.3638 $\rightarrow$ 0.5021**), nhưng đi kèm đánh đổi: mô hình có xu hướng dự đoán PHE rộng hơn thực tế quanh vùng xuất huyết (FP tăng từ 1455.2 lên 2324.5 voxel).
3. **Nút thắt nằm ở phương pháp tích hợp tri thức:** Việc cải tiến chất lượng mô hình ICH Teacher ngoài không giúp tăng Dice của PHE downstream. Điểm mấu chốt nằm ở *cách biểu diễn và dung hợp kênh thông tin không gian* chứ không nằm ở chất lượng dự đoán của Teacher.

---

## 📂 Danh mục mã nguồn

Chi tiết mã nguồn thực nghiệm thực tế được lưu trữ tại thư mục [/notebook](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook):

### 1. Các baseline 2.5D & Dòng mô hình 3DFF-Net
* [02-4-3dff-net-iph-phe-25d-segmentation-racda5a49ad.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/02-4-3dff-net-iph-phe-25d-segmentation-racda5a49ad.ipynb): Kiến trúc 3DFF-Net 2.5D baseline cho IPH + PHE.
* [02-5-lite-3dff-iph-phe-25d-segmentation-refined-ps.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/02-5-lite-3dff-iph-phe-25d-segmentation-refined-ps.ipynb): Phiên bản thu gọn (Lite) của 3DFF-Net.
* [02-6-balanced-3dff-iph-phe-25d-segmentation-refine.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/02-6-balanced-3dff-iph-phe-25d-segmentation-refine.ipynb): Cải tiến cân bằng mẫu cho 3DFF-Net.
* [02-7-phe-pretrained-3dff-iph-phe-refined-pseudo.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/02-7-phe-pretrained-3dff-iph-phe-refined-pseudo.ipynb): 3DFF-Net pretrain kết hợp pseudo labels.
* [02_10_pese_guided_3dff_iph_phe_25d_segmentation_local.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/02_10_pese_guided_3dff_iph_phe_25d_segmentation_local.ipynb): Tích hợp prior dẫn đường từ lát cắt kề cận (PESE-guided).
* [02_10b_pese_guided_3dff_iph_phe_25d_segmentation_oldsplit_local.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/02_10b_pese_guided_3dff_iph_phe_25d_segmentation_oldsplit_local.ipynb): PESE-guided chạy local trên phân chia split cũ (48/48/24).
* [02_11_enhanced_pese_guided_3dff_iph_phe_25d_segmentation_local.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/02_11_enhanced_pese_guided_3dff_iph_phe_25d_segmentation_local.ipynb): Phiên bản nâng cao của PESE-guided 3DFF-Net.
* [option_0d_phe_only_25d_detection.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/option_0d_phe_only_25d_detection.ipynb): Thử nghiệm bài toán phát hiện (detection) 2.5D.
* [option_0s_phe_only_25d_fusion_fpn_segmentation.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/option_0s_phe_only_25d_fusion_fpn_segmentation.ipynb): Phân đoạn 2.5D kết hợp FPN fusion.

### 2. Thực nghiệm nnU-Net Baseline (Kaggle & Local)
* [02-12b-kaggle-nnunet-phe-only-120epochs.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/02-12b-kaggle-nnunet-phe-only-120epochs.ipynb): Huấn luyện nnU-Net PHE-only 120 epochs trên Kaggle.
* [02-15-kaggle-nnunet-phe-only-baseline6c8c5e4146.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/02-15-kaggle-nnunet-phe-only-baseline6c8c5e4146.ipynb): Run baseline nnU-Net trên Kaggle.
* [02-16-kaggle-mednext-s-phe-only-baseline.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/02-16-kaggle-mednext-s-phe-only-baseline.ipynb): Chạy baseline MedNeXt-S PHE-only trên Kaggle.
* [02_12b_kaggle_nnunet_phe_brain_subdural_skullstrip_baseline.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/02_12b_kaggle_nnunet_phe_brain_subdural_skullstrip_baseline.ipynb): Thử nghiệm nnU-Net kết hợp lọc sọ não (skull strip) trên Kaggle.
* [02_12c_kaggle_nnunet_phe_only_19c_split.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/02_12c_kaggle_nnunet_phe_only_19c_split.ipynb): Huấn luyện nnU-Net với tập split 19 ca trên Kaggle.
* [02_12c_nnunet_phe_only_19c_split_local.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/02_12c_nnunet_phe_only_19c_split_local.ipynb): Huấn luyện nnU-Net với tập split 19 ca chạy local.

### 3. Chuyển giao tri thức & Tiên nghiệm không gian (Prior Transfer)
* [02_14c_nnunet_phe_ich_teacher_peri_ring_prior.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/02_14c_nnunet_phe_ich_teacher_peri_ring_prior.ipynb): nnU-Net kết hợp tiên nghiệm không gian dạng vành khuyên (Ring Prior).
* [02_19c_nnunet_phe_pseudo_ich_prior_control_resplit_hardval.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/02_19c_nnunet_phe_pseudo_ich_prior_control_resplit_hardval.ipynb): Nhánh đối chứng Pseudo-prior từ CT gốc trên tập validation khó.
* [03_1.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/03_1.ipynb) & [03_2.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/03_2.ipynb): Phân tích hiệu quả chuyển giao tri thức tiên nghiệm và đánh giá metrics.
* [04_option4_ich_prior_transfer_phe_student.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/04_option4_ich_prior_transfer_phe_student.ipynb): Thực nghiệm transfer tri thức sang mô hình Student.
* [05_option5_ring_prior_warmstart_phe_student.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/05_option5_ring_prior_warmstart_phe_student.ipynb): Khởi động ấm (Warmstart) mô hình student với Ring Prior.

### 4. Đánh giá dữ liệu & Pipeline hỗ trợ
* [Seg_CQ500_EDA.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/Seg_CQ500_EDA.ipynb): Phân tích khám phá dữ liệu bộ Seg-CQ500.
* [phe_sich_seg_cq500_research_pipeline.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/phe_sich_seg_cq500_research_pipeline.ipynb): Pipeline liên kết nghiên cứu Seg-CQ500.
* [q1_phe_ich_prior_transfer_pipeline.ipynb](file:///d:/Thuy_Loi/Nam_3/CT_xuathuyetnao/PHE-ICH-Segmentation-CT/notebook/q1_phe_ich_prior_transfer_pipeline.ipynb): Thiết lập khung pipeline chuyển giao tri thức tiên nghiệm chính.

---

## 🚀 Tái lập thực nghiệm

### Cài đặt thư viện
```bash
pip install numpy pandas matplotlib scipy nibabel tqdm torch torchvision nnunetv2
```

### Tài liệu tham khảo
* **PHE-SICH-CT-IDS:** Ma et al., arXiv:2308.10521.
* **nnU-Net Framework:** Isensee et al., Nature Methods 2021.
* **INSTANCE Challenge:** Li et al., arXiv:2301.03281.
