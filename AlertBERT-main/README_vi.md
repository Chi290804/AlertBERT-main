# AlertBERT

Đây là kho mã nguồn cho bài báo “AlertBERT: A noise-robust alert grouping framework for simultaneous attacks” (bài báo hiện đang chờ xuất bản).

## Mô tả

AlertBERT là một phương pháp gom nhóm cảnh báo (alert grouping) tự giám sát (self-supervised) hiện đại, dựa trên mô hình masked-language-model để thu được các vector biểu diễn (embedding) cho cảnh báo, và sử dụng phân cụm kết tụ (agglomerative clustering) để hoạt động tốt trong điều kiện nhiễu cao và nhiều cuộc tấn công mạng diễn ra đồng thời.
Để biết thêm chi tiết về khung AlertBERT, xin tham khảo bài báo của chúng tôi (hiện đang chờ xuất bản).

Để tái tạo các thí nghiệm được báo cáo trong bài báo, trước tiên hãy chạy module `alertbert.train_mlm` để huấn luyện một masked-language-model.
Tại đó, các tham số duy nhất cần điều chỉnh trong `MaskedLangModelParams` truyền vào hàm `main` là số lượng attention head và cấu hình AIT-ADS-A được sử dụng.
Sau khi hoàn tất huấn luyện masked-language-model, hãy chạy module `alertbert.eval_grouping` để tính các đường cong ROC tương ứng bằng cách cung cấp `model_id` cho hàm `model_config_generator`.
Để xem kết quả, hãy tham khảo các đồ thị ROC được cung cấp trong các notebook trong thư mục `roc_results`.

Các kết quả này sẽ tương tự như đồ thị ROC sau đây thu được trên cấu hình `simul-attacks` của AIT-ADS-A.

![ROC-plot of AlertBERT on the `simul-attacks` configuration of AIT-ADS-A](./roc_results/ROC_mlm_1l_2h_16d_simul-attacks_1_60k_simul-attacks_output_emb_2_dim_excl_noise.png)

Các module riêng lẻ trong `alertbert` có mục đích như sau:

+ `alertbert.aitads` cung cấp tập dữ liệu,

+ `alertbert.eval_grouping` hiện thực việc đánh giá gom nhóm cảnh báo,

+ `alertbert.eval_mlm` hiện thực việc đánh giá quá trình huấn luyện masked-language-model,

+ `alertbert.model_eval_utils` cung cấp các tiện ích đánh giá,

+ `alertbert.models` hiện thực các mô hình gom nhóm cảnh báo,

+ `alertbert.preprocessing` hiện thực từ vựng và việc token hóa dữ liệu,

+ `alertbert.train_mlm` thực hiện huấn luyện masked-language-model, và

+ `alertbert.utils` cung cấp các tiện ích chung.

## Cài đặt

Để chạy các thí nghiệm, hãy làm theo các bước sau:

### Môi trường Python

Để thiết lập môi trường Python, chạy `pip install -r requirements.txt`.
Ngoài ra, cần cài đặt thư viện [graph-tools](https://graph-tool.skewed.de), thư viện này không có sẵn trên pip.

### Tải và chuẩn bị dữ liệu

Trong các thí nghiệm của chúng tôi, chúng tôi sử dụng tập dữ liệu AIT-ADS-A, là phiên bản tăng cường (augmented) của [AIT Alert Dataset](https://doi.org/10.5281/zenodo.8263181).
Các file để xây dựng tập dữ liệu này là một phần của kho mã và cần có trong thư mục `aitads_augmented/data`.

Nếu điều kiện trên đã được đáp ứng, quá trình thiết lập đã hoàn tất và các bước còn lại là không cần thiết!

Để xây dựng tập dữ liệu từ mã nguồn, hãy làm theo các bước sau:

1. Tải về và giải nén ba tập dữ liệu vào đúng thư mục tương ứng như liệt kê bên dưới.
   Sau bước này, thư mục `alerts_json` phải chứa các file `scenario_aminer.json` và `scenario_wazuh.json` cho mỗi trong tám kịch bản (scenario) của AIT-ADS.
    + [AIT Alert Dataset](https://doi.org/10.5281/zenodo.8263180) ➡️ `alerts_json`
    + [AIT Log Dataset V2.0](https://doi.org/10.5281/zenodo.5789063) ➡️ `aitldsv2`
    + [AIT Netflow Dataset](https://doi.org/10.5281/zenodo.6610488) ➡️ `aitnds`

2. Chạy `preprocess.py`.
   Script này sẽ đọc thông tin từ ba tập dữ liệu, sử dụng chúng để gán nhãn cho các cảnh báo trong AIT-ADS và lưu nhãn vào các file `alerts_csv/scenario_alerts.csv` cho mỗi kịch bản.

Tại thời điểm này, đối với mỗi kịch bản, chúng ta có tình huống sau:
Các file `alerts_json/scenario_wazuh.json` và `alerts_json/scenario_aminer.json` chứa dữ liệu cảnh báo được sắp xếp theo mốc thời gian, nhưng tách biệt cho hai IDS.
Trong khi đó, các file `alerts_csv/scenario_alerts.csv` chứa nhãn cho các cảnh báo, nhưng ở đó các cảnh báo được sắp xếp theo thứ tự tương ứng với phép nối (concatenation) của `alerts_json/scenario_wazuh.json` và `alerts_json/scenario_aminer.json`.
Vì vậy, bước tiếp theo:

3. Chạy `unite_alerts_labels.py`.
   Script này sẽ đơn giản hóa tình huống trên bằng cách kết hợp tất cả cảnh báo và nhãn của chúng, được sắp theo mốc thời gian, vào các file `alerts_json/scenario.json` cho mỗi kịch bản.
   Ngoài ra, script sẽ tạo các file `alerts_json/scenario_light.json` có nội dung giống nhau ngoại trừ việc bỏ dữ liệu thô của cảnh báo, có thể dùng để tăng tốc độ tải dữ liệu nếu không cần đến dữ liệu cảnh báo thô.

4. Cuối cùng, để tạo các file cần thiết cho việc xây dựng AIT-ADS-A, hãy chạy `build_augment_files.py`.
   Để biết thêm thông tin về việc thiết lập AIT-ADS-A, xin tham khảo file README trong thư mục `aitads_augmented`.

## Cách sử dụng

+ Các module `alertbert/module.py` phải được chạy thông qua `python -m alertbert.module`.
+ Tài liệu của mã nguồn có trong các docstring tương ứng của hàm, lớp và module.
