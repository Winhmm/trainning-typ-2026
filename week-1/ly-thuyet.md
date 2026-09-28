# PHẦN 1: INDEX

## I. Index là gì?

Index (chỉ mục) trong Database là một cấu trúc dữ liệu đặc biệt được tạo dựa trên một hoặc nhiều cột của bảng, giúp Database tìm kiếm và truy xuất các bản ghi nhanh hơn, từ đó có thể giảm lượng dữ liệu cần kiểm tra.

Có thể hiểu đơn giản rằng Index giống như mục lục của một cuốn sách. Ví dụ, một cuốn sách có 500 trang. Nếu muốn tìm một chủ đề mà không có mục lục, chúng ta có thể phải lật từng trang để tìm. Ngược lại, nếu có mục lục, chúng ta có thể nhanh chóng xác định chủ đề nằm ở trang nào và đi trực tiếp đến đó.

Database cũng tương tự, Index cung cấp cho Database một cấu trúc được tổ chức để có thể nhanh chóng tìm kiếm các giá trị cần thiết thay vì phải kiểm tra lần lượt toàn bộ dữ liệu trong bảng.

## II. Index hoạt động như thế nào?

Khi một Index được tạo trên một cột trong Database, hệ quản trị cơ sở dữ liệu sẽ xây dựng một cấu trúc dữ liệu riêng dựa trên các giá trị của cột đó. Cấu trúc này cho phép Database nhanh chóng tìm kiếm và xác định các bản ghi phù hợp, từ đó có thể giảm lượng dữ liệu cần kiểm tra so với việc quét toàn bộ bảng.

Về cơ bản, có thể hình dung Index gồm hai thành phần chính:

Key: giá trị được lấy từ cột mà Index được tạo trên đó, dùng để tìm kiếm trong Index.

Pointer: thông tin tham chiếu giúp Database xác định dữ liệu tương ứng với Key.

## III. Cấu trúc của Index

Index có thể được xây dựng dựa trên nhiều loại cấu trúc dữ liệu khác nhau. Mỗi cấu trúc có cách tổ chức dữ liệu và đặc điểm tìm kiếm riêng và loại thường được nhắc đến khi tìm hiểu Index là B-Tree / B+Tree.

# PHẦN 2: INDEX

## I. B+ Tree là gì?

B+ Tree (B+ Tree) là một cấu trúc dữ liệu dạng cây tìm kiếm cân bằng, được sử dụng phổ biến trong các hệ quản trị cơ sở dữ liệu và hệ thống lưu trữ để tổ chức và truy xuất dữ liệu hiệu quả. Trong B+ Tree, các nút trung gian chủ yếu chứa các khóa dùng để định hướng quá trình tìm kiếm, trong khi dữ liệu hoặc con trỏ đến dữ liệu được lưu tại các nút lá. Các nút lá được liên kết với nhau theo thứ tự tăng dần của khóa. Đặc điểm này giúp B+ Tree hỗ trợ hiệu quả các thao tác tìm kiếm theo khoảng và duyệt tuần tự dữ liệu.

## II. Cấu trúc của B+ Tree?

Một B+ Tree gồm ba thành phần chính:

Nút gốc (Root Node): là nút ở vị trí cao nhất của cây, chứa các khóa và các con trỏ đến các nút con, giúp xác định nhánh cần đi xuống trong quá trình tìm kiếm.

Nút trung gian (Internal Node): là nơi chứa các khóa phân chia phạm vi giá trị và các con trỏ đến các nút con, được sử dụng để định hướng quá trình tìm kiếm, không phải nơi lưu trữ bản ghi dữ liệu cuối cùng.

Nút lá (Leaf Node): là nơi chứa các khóa của dữ liệu và thường chứa con trỏ đến bản ghi tương ứng trong cơ sở dữ liệu.

## III. Một số tính chất của B+ Tree

B+ Tree luôn duy trì trạng thái cân bằng và tất cả các nút lá đều nằm trên cùng một mức của cây. Dữ liệu hoặc con trỏ đến dữ liệu được lưu tại các nút lá trong khi các nút trung gian chủ yếu chứa khóa và con trỏ dùng để điều hướng quá trình tìm kiếm. Các nút lá được liên kết theo thứ tự của khóa. Các khóa trong B+ Tree được duy trì theo thứ tự.

Khác với cây nhị phân, một nút trong B+ Tree có thể chứa nhiều khóa và có nhiều nút con. Do các nút lá được liên kết tuần tự và các khóa được sắp xếp, B+ Tree hỗ trợ tốt các thao tác tìm kiếm theo một khoảng giá trị. Khi thực hiện thao tác chèn hoặc xóa, B+ Tree có cơ chế phân chia, gộp hoặc phân phối lại các nút để duy trì các điều kiện của cây.

# PHẦN 3: INDEX

##I. Clustered Index là gì?
Clustered Index là một loại Index dùng để tổ chức dữ liệu trong bảng theo thứ tự của một cột. Có thể hiểu đơn giản là nó vừa là Index để tìm kiếm, vừa quyết định cách dữ liệu trong bảng được tổ chức theo thứ tự của Index đó.

Ví dụ, nếu sử dụng cột ID làm Clustered Index thì dữ liệu sẽ được tổ chức theo thứ tự của ID. Nhờ dữ liệu được tổ chức theo thứ tự này, Database có thể tìm kiếm dữ liệu dựa trên ID hiệu quả hơn.

##II. Đặc điểm của Clustered Index

1. Dữ liệu được tổ chức theo Index

- Clustered Index gắn trực tiếp với cách dữ liệu trong bảng được tổ chức.
- Nếu Clustered Index sử dụng cột ID, dữ liệu sẽ được tổ chức theo thứ tự của ID.

2. Một bảng chỉ có một Clustered Index

- Lý do là dữ liệu trong bảng chỉ có thể được tổ chức theo một thứ tự chính.
- Ví dụ, không thể vừa tổ chức dữ liệu chính theo ID, vừa tổ chức dữ liệu chính theo Name.

3. Có thể được tạo trên Primary Key

- Primary Key dùng để xác định duy nhất một bản ghi.
- Primary Key và Clustered Index không phải là cùng một khái niệm.

# PHẦN 4: INDEX

##I. Non-Clustered Index là gì?

Non-Clustered Index là một loại Index được tạo riêng bên ngoài dữ liệu của bảng, dùng để giúp Database tìm kiếm dữ liệu nhanh hơn. Có thể hiểu đơn giản là Non-Clustered Index giống như một bảng tra cứu, lưu giá trị của cột được đánh Index và thông tin giúp Database tìm đến bản ghi tương ứng trong bảng.

Ví dụ, nếu tạo Non-Clustered Index trên cột Name, Index sẽ lưu các giá trị của Name theo thứ tự và thông tin để Database tìm đến dữ liệu tương ứng trong bảng.

##II. Đặc điểm của Non-Clustered Index

1. Không tổ chức trực tiếp dữ liệu trong bảng

- Non-Clustered Index được lưu riêng với dữ liệu của bảng.
- Nó không quyết định thứ tự tổ chức dữ liệu chính trong bảng.
- Index lưu thông tin giúp Database tìm đến bản ghi cần tìm.

2. Một bảng có thể có nhiều Non-Clustered Index

- Một bảng có thể có nhiều Non-Clustered Index.
- Mỗi Non-Clustered Index có thể được tạo trên một hoặc nhiều cột khác nhau.
- Điều này giúp Database tối ưu nhiều loại truy vấn tìm kiếm khác nhau.

3. Không giống Clustered Index

- Clustered Index tổ chức dữ liệu của bảng theo Index.
- Non-Clustered Index được lưu riêng và dùng để tìm đến dữ liệu trong bảng.
- Một bảng chỉ có tối đa một Clustered Index nhưng có thể có nhiều Non-Clustered Index.

4. Có thể chứa nhiều cột

- Non-Clustered Index có thể được tạo trên một cột hoặc nhiều cột.
- Khi Index được tạo trên nhiều cột, Database có thể sử dụng Index đó cho các truy vấn liên quan đến các cột được đánh Index.
