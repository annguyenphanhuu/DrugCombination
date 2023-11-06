#DeepDDS: deep graph neural network with attention
mechanism to predict synergistic drug combinations

#Introduction
Động lực: Vấn đề đặt ra là có quá nhiều cách để kết hợp các loại thuốc, đến mức mà việc kiểm tra những kết hợp nào là hiệu quả thông qua thí nghiệm trong phòng thí nghiệm trở nên rất khó khăn. Do đó, việc sử dụng máy học để dự đoán các kết hợp thuốc đã trở thành một cách tiếp cận quan trọng.
Việc phát hiện truyền thống về sự kết hợp thuốc chủ yếu dựa trên các thử nghiệm lâm sàng và chỉ giới hạn ở một số loại thuốc, tốn kém chi phí và không thực tế, không đáp ứng được nhu cầu cấp thiết về thuốc chống ung thư.

Một số nguồn dữ liệu:
Độ nhạy của thuốc: CCLE (Bách khoa toàn thư về tế bào ung thư), GDSC (Độ nhạy của thuốc trong ung thư)
Kết hợp thuốc: DrugCombDB

#Materials and methods
Nguồn dữ liệu được sử dụng trong paper:
- SMILES:  là một chuỗi ký tự được sử dụng để biểu diễn một cấu trúc hóa học của một hợp chất hữu cơ dưới dạng văn bản. Sẽ được chuyển đổi thành KG bằng RDKit
- CCLE: Dữ liệu biểu hiện gene của các dòng tế bào ung thư sẽ được lấy ra từ đây
- DrugCombDB

 


Đối với sự kết hợp thuốc theo cặp: Sử dụng GCN hoặc GAT để trích xuất các tính năng của thuốc.
Đối với tế bào ung thư: Sự biểu hiện đặc điểm bộ gen của các tế bào ung thư được mã hóa bằng MLP
⟹ Các vectơ nhúng sau đó được nối lại thành vectơ cuối cùng biểu diễn đặc điểm của từng tổ hợp thuốc-cặp-tế bào, được truyền qua các lớp được kết nối đầy đủ để phân loại nhị phân các tổ hợp thuốc

Biểu diễn thuốc dựa trên GNN:
- Sử dụng phần mềm tin học hóa học nguồn mở RDKit để chuyển đổi SMILES thành biểu đồ phân
tử, trong đó các nút là nguyên tử và các cạnh là liên kết hóa học.
- Đối với mỗi nút trong biểu đồ, chúng tôi sử dụng DeepChem để tính toán một tập hợp các thuộc tính
nguyên tử làm đặc điểm ban đầu của nó. Cụ thể, mỗi nút được biểu diễn dưới dạng một vectơ nhị phân bao gồm năm thông tin: ký hiệu nguyên tử, số nguyên tử liền kề, số hydro liền kề, giá trị ngầm định của nguyên tử và liệu nguyên tử đó có ở cấu trúc Aromatic hay không.
- Trong GNN, quá trình biểu diễn thuốc thực chất là quá trình truyền thông tin giữa mỗi nút tới các nút lân cận.

Bổ sung:
Cấu trúc hoá học hữu cơ có thể được phân thành 2 loại chính: cấu trúc Aliphatic và cấu trúc Aromatic
- Cấu trúc Aliphatic liên quan đến các hợp chất hữu cơ có liên kết cacbon được sắp xếp dài, thẳng hoặc uốn lượn.
- Cấu trúc Aromatic liên quan đến các hợp chất chứa ít nhất một vòng benzen (còn được gọi là vòng aromatic). Cấu trúc benzen là một vòng 6 nguyên tử cacbon có liên kết đôi xen kẽ với liên kết đơn.

Bài báo sử dụng 2 loại mạng là: GCN và GAT
GCN: Sử dụng 3 lớp GCN liên tiếp được kích hoạt bởi hàm ReLU. Sau khi thử nghiệm chúng tôi thấy rằng việc sử dụng lớp Max Pooling trong DeepDDS dựa trên GCN tốt hơn các lớp khác. Do đó, chúng tôi thêm lớp Max Pooling toàn cầu sau lớp GCN cuối cùng để trích xuất biểu diễn.
GAT: Mạng chú ý biểu đồ (GAT) đề xuất kiến trúc dựa trên sự chú ý nhiều đầu để tìm hiểu các tính năng cấp cao hơn của các nút trong biểu đồ bằng cách áp dụng cơ chế tự chú ý.

Trích xuất đặc điểm tế bào:
- Để giảm bớt sự mất cân bằng kích thước giữa các vectơ đặc trưng của thuốc và dòng tế bào, chúng tôi đã chọn các gen quan trọng theo dự án LINCS.
- Áp dụng MLP để trích xuất các tính năng của dòng tế bào MLP bao gồm hai lớp ẩn.

#Result
Cài đặt siêu tham số:
- Kiến trúc thực sự của DeepDDS thực sự được xác định bằng cài đặt siêu tham số. Các siêu tham số bao gồm số lượng lớp và đơn vị của mỗi lớp trong MLP, GCN và GAN, cũng như hàm kích hoạt và learning rate.
- Sau các thử nghiệm five-fold cross-validations on benchmark dataset rút ra được những điều sau:
	GCN mang lại hiệu suất tốt hơn trong việc trích xuất đặc tính thuốc khi cấu trúc của nó có 3 lớp ẩn và số lượng đơn vị lần lượt là 1024, 512 và 156.
	GAN và MLP hoạt động tốt nhất với hai lớp ẩn.
	Đối với hàm kích hoạt, chức năng kích hoạt ELU và ReLU sau các lớp GAT tại DeepDDS-GAT được sử dụng.
	Đối với DeepDDS-GCN, nó cũng có cấu trúc lớp tương tự nhưng chỉ có ReLU được sử dụng làm chức năng kích hoạt.
 
So sánh hiệu suất (Phần này chủ yếu so sánh hiệu suất với các mô hình khác nên không cần đi sâu)
 
 
 
 
 

GAT cho thấy cấu trúc hoá học quan trọng
Mô hình DeepDDS-GAT lặp đi lặp lại các thông điệp giữa các nút để mỗi nút có thể nắm bắt thông tin của các nút lân cận. Trong khi đó, mỗi nơ-ron được kết nối với lớp trên của vùng lân cận thông qua một tập hợp các trọng số có thể học được trong mạng GAT. Kết quả là, biểu diễn đặc điểm thực sự mã
hóa thông tin của cấu trúc hóa học xung quanh nguyên tử, bao gồm điện tích hình thức, độ hòa tan trong nước và các đặc tính hóa lý khác. Điều này thúc đẩy chúng tôi khám phá ý nghĩa của cơ chế chú ý trong việc tiết lộ các cấu trúc hóa học quan trọng.
 
Kết quả là, các vectơ nhúng nguyên tử hiển thị các mẫu đặc điểm rõ ràng trong quá trình huấn luyện, cụ thể là các ma trận tương quan nguyên tử phân cụm rõ ràng thành một số nhóm nguyên tử và mức độ liên kết giữa các nhóm nguyên tử của các loại thuốc khác nhau chuyển từ hỗn loạn sang trật tự.

Dự đoán sự kết hợp mới
Tổng quát: Đạt được kết quả tốt hơn so với các mô hình khác

Thảo luận:
- Vẫn còn hạn chế trên bộ thử nghiệm độc lập vì lượng mẫu đào tạo còn ít.
- Các đặc tính hóa lý của đồ thị phân tử và trọng lượng chú ý giữa các nguyên tử vẫn chưa được hiểu đầy đủ.









