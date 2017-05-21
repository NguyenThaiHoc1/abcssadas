## Policy 1: Tạo tài khoản cho các nhân viên trong bảng nhân viên

### Tạo các user ứng với mã nhân viên, trong đó:

>`User NV000 là Giám Đốc`

>`User NV001, NV101, NV201, NV301,... là các Trưởng chi nhánh`

>`User NV011, NV021, NV031, NV NV011, NV021, NV031, NV041, NV111, NV121, NV131,... là các Trưởng phòng`

>`User NV112, NV212, NV142, NV332, NV422,... là các Trưởng đề án`

>`User NV012, NV022, NV042, NV043, NV113, NV122, NV143, NV213, NV232, NV243, NV333 là các Nhân viên bình thường`

+ #### Ví dụ từ mã nguồn:

>`create user NV401 identified by 123;`

## Policy2: Tạo role cho các vị trí phù hợp của công ty

### Tạo ra 5 role gồm: GiamDoc, Truong_CN_CTY, Truong_Phong_CTY, Truong_DA_CTY, NV_BT_CTY

+ #### Ví dụ từ mã nguồn:

>`create role GiamDoc;`

### Sau đó gán từng User vừa tạo ở policy 1 cho vào từng role tương ứng.

+ #### Ví dụ từ mã nguồn:

>`grant GiamDoc to NV000;`

## Policy 3: Chỉ trưởng phòng được phép cập nhật và thêm thông tin vào dự án (DAC) 

### Giải pháp: Gán quyền update và insert cho role Truong_Phong_CTY trên bảng DuAn

### Tận dụng role Truong_Phong_CTY đã được tạo từ policy 2, ta sẽ gán quyền Gán quyền update và insert cho role Truong_Phong_CTY trên bảng DuAn

+ #### Ví dụ từ mã nguồn:

>`grant update, insert on DuAn to Truong_Phong_CTY;`

## Policy 4: Giám đốc được phép xem thông tin dự án gồm (mã dự án, tên dự án, kinh phí, tên phòng chủ trì, tên chi nhánh chủ trì, tên trưởng dự án và tổng chi) (DAC).

### Giải pháp: Tạo ra 1 view và sau đó cấp quyền truy cập trên view đó cho role GiamDoc

### Tạo view có tên là View_GiamDoc, sử dụng kết bằng từ những bảng DuAn, ChiNhanh, NhanVien, PhongBan, ChiTieu để lấy ra những thông tin thỏa mãn yêu cầu

+ #### Ví dụ từ mã nguồn: 

>`create or replace view View_GiamDoc `

>`as select da.maDA, da.tenDA, da. kinhPhi, pb.tenphong as tenphongchutri, cn.tenCN as tenchinhanh, nv.hoTen as truongduan , sum(ct.soTien) as ChiPhi`

>`from DuAn da, ChiNhanh cn, NhanVien nv, PhongBan pb, ChiTieu ct`

>`where da.phongChuTri = pb.maPhong and pb.chiNhanh = cn.maCN and da.truongDA = nv.maNV and da.maDA = ct.duAn`

>`group by da.maDA, da.tenDA, da. kinhPhi, pb.tenphong, cn.tenCN, nv.hoTen;`

+ #### Câu lệnh:

>`grant  EXEMPT ACCESS POLICY to GiamDoc;// Người được cấp quyền này sẽ được miễn khỏi tất cả các function RLS`

### Thực hiện cấp quyền cho role GiamDoc trên view View_GiamDoc:

+ #### Ví dụ từ mã nguồn: 

>`grant select on View_GiamDoc to GiamDoc;`

## Policy 5: Chỉ trưởng phòng, trưởng chi nhánh được cấp quyền thực thi stored procedure cập nhật thông tin phòng ban của mình (ĐẶC).

### Giải pháp: Tạo ra 2 procedure và sau đó cấp quyền thực thi cho rơle Trưởng_CN_CTY và Trưởng_Phòng_CTY.

+ #### Procedure 1 dành cho những ai là trưởng chi nhánh thì được phép cập nhật thông tin phòng ban của chi nhánh mình
+ #### Procedure 2 dành cho những ai là trưởng phòng thì được phép cập nhật thông tin phòng ban của mình quản lý

### Tạo procedure 1 có tên là CapNhatThongTin_TruongCN, trong đó dữ liệu đầu vào sẽ là mã phòng cần cập nhật (maPhongCN), tên phòng cập nhật (tenPhongNew) và số lượng nhân viên mà mình mong muốn cập nhật (soNhanVienNew). Kiểm tra nếu là user trưởng chi nhánh thì sẽ lấy ra mã chi nhánh tương ứng, sau đó cập nhật dựa trên mã chi nhánh và mã phòng.

### Tạo procedure 2 có tên là CapNhatThongTin_TruongPhong, trong đó dữ liệu đầu vào sẽ là tên phòng cập nhật (tenPhongNew) và số lượng nhân viên mà mình mong muốn cập nhật (soNhanVienNew). Kiểm tra nếu là trường phòng thì sẽ lấy ra mã phòng tương ứng, sau đó cập nhật dựa trên mã phòng mới được lấy ra.

### Cấp quyền thực thi:

+ #### Ví dụ từ mã nguồn:

>`grant execute on CapNhatThongTin_TruongCN to Truong_CN_CTY;`

>`grant execute on CapNhatThongTin_TruongPhong to Truong_Phong_CTY;`

## Policy 6: Tất cả nhân viên bình thường (trừ trưởng phòng, trưởng chi nhánh và các trưởng dự án) chỉ được phép xem thông tin nhân viên trong phòng của mình, chỉ được xem lương của bản thân (VPD).

### Giải pháp: Tạo ra 2 function (Select_NhanVien, Select_Phong) tương ứng cho 2 policy (S_NhanVien, Phong_NhanVien)

+ #### Function 1 (Select_NhanVien): Trả về vị từ là user là nhân viên bình thường, nếu user là trưởng phòng hoặc là trưởng chi nhánh hoặc là trưởng đề án thì được xem hết dữ liệu ở bảng NhanVien và sau đó gắn vào policy (S_NhanVien) áp dụng cho bảng NhanVien. Ý nghĩa: Cho phép nhân viên chỉ được phép xem lương của chính bản thân mình

+ ### Ví dụ từ mã nguồn: 

>`Select count (*) into num from PhongBan, ChiNhanh, DuAn where user = truongPhong or user = truongChiNhanh or user = truongDA;`

>` if (num > 0) then`

>`  RETURN '';`

+ #### Function 2 (Select_Phong): Trả về vị từ là maPhong ứng với user là nhân viên bình thường, nếu user là trưởng phòng hoặc là trưởng chi nhánh hoặc là trưởng đề án thì được xem hết dữ liệu ở bảng NhanVien và sau đó gắn vào policy (Select_Phong) áp dụng cũng cho bảng NhanVien. Ý nghĩa: Lấy ra những nhân viên cùng phòng ban.

## Policy 7: Trưởng dự án chỉ được phép đọc, ghi thông tin chi tiêu của dự án mình quản lý (VPD).

### Giải pháp: Tạo ra 1 function (Select_CHITIEU) trả về vị từ là DUAN ứng với user là trưởng dự án, nếu user là trưởng phòng thì được xem hết thông tin ở bảng ChiTieu và sau đó gắn vào policy (Select_CHITIEU) để áp dụng cho bảng ChiTieu. Ý nghĩa: Trưởng dự án chỉ được phép đọc, ghi thông tin chi tiêu của dự án mình quản lý

+ ### Ví dụ từ mã nguồn: 

>`Select count (*) into num from PhongBan where user = truongPhong`
>` if (num > 0) then`
>`  RETURN '';`

## Policy 8: Trưởng phòng chỉ được phép đọc thông tin chi tiêu của dự án trong phòng ban mình quản lý. Với những dự án không thuộc phòng ban của mình, các trưởng phòng được phép xem thông tin chi tiêu nhưng không được phép xem số tiền cụ thể (VPD).

### Giải pháp: Tạo ra 1 function (Select_ChiTieu_TruongPhong) trả về vị từ là DUAN ứng với user là trưởng phòng của dự án trong phòng ban mình và sau đó gắn vào policy (S_ChiTieu_TruongPhong) để áp dụng cho bảng ChiTieu. Ý nghĩa: Trưởng phòng chỉ được phép đọc thông tin chi tiêu của dự án trong phòng ban mình quản lý

## Policy 9: Mỗi dự án trong công ty có các mức độ nhạy cảm được đánh dấu bao gồm “Thông thường”, “Giới hạn”, “Bí mật”, “Bí mật cao”. Mỗi dự án có thể thuộc quyền quản lýcủa tổng công ty hoặc của 1 trong 3 chi nhánh “Tp.Hồ Chí Minh”, “Hà Nội”, “Đà Nẵng”. Mỗi dự án có thể liên quan đến các phòng ban: “Nhân sự”, “Kế toán”, “Kế hoạch”. Trưởng chi nhánh được phép truy xuất tất cả dữ liệu chi tiêu của dự án của tất cả các phòng ban thuộc quyền quản lý của mình. Trưởng chi nhánh Hà Nội được phép truy xuất dữ liệu của chi nhánh Hà Nội và tất cả các chi nhánh khác. Trưởng phòng được phép đọc dữ liệu dự án của tất cả phòng ban nhưng chỉ được phép ghi dữ liệu dự án thuộc phòng của mình. Nhân viên chỉ được đọc dữ liệu dự mình tham gia (OLS).
### 9.1 Tạo các thành phần policy.

#### Tạo các level “Thông thường”, “Giới hạn”, “Bí mật”, “Bí mật cao”.

+ ### Ví dụ từ mã nguồn: 

>`BEGIN`

>`sa_components.create_level (policy_name => 'ACCESS_DUAN', long_name => 'THONGTHUONG', short_name => 'TT',level_num => 500);`

>`END;`

#### Tạo các compartment “Nhân sự”, “Kế toán”, “Kế hoạch”.

+ ### Ví dụ từ mã nguồn:

>`EXECUTE sa_components.create_compartment ('ACCESS_DUAN',1000,'NS','NHANSU');`

#### Tạo 1 Group cha (TONGCTY) và 3 group con (DaNang, HoChiMinh, HN).

+ ### Ví dụ từ mã nguồn:

>`EXECUTE sa_components.create_group ('ACCESS_DUAN',2000,'DN','DANANG','TCT');`

#### Tạo ra các label tương ứng

+ ### Ví dụ từ mã nguồn:

>`EXECUTE sa_label_admin.create_label ('ACCESS_DUAN',100,'GH');`

### 9.2 Trưởng chi nhánh được phép truy xuất tất cả dữ liệu chi tiêu của dự án của tất cả các phòng ban thuộc quyền quản lý của mình.

#### Gán nhãn cho dữ liệu bảng ChiTieu

+ ### Ví dụ từ mã nguồn:

>`UPDATE CHITIEU SET OLS_DUAN = char_to_label ('ACCESS_DUAN', 'GH:KH:DN')`

>`WHERE MACHITIEU = 'CT101';`

##### Sau đó gán nhãn cho các user là Trưởng chi nhánh

+ ### Ví dụ từ mã nguồn:

>`BEGIN`

>`sa_user_admin.set_user_labels`

>`(policy_name => 'ACCESS_DUAN',`

>`user_name => 'NV001',`

>`max_read_label => 'BMC:NS,KT,KH:DN',`

>`max_write_label => 'BMC:NS,KT,KH:DN',`

>`min_write_label => 'GH',`

>`def_label => 'BMC:NS,KT,KH:DN',`

>`row_label => 'BMC:NS,KT,KH:DN');`

>`END;`

#### Cấp quyền select trên bảng ChiTieu cho các Trưởng chi nhánh

+ ### Ví dụ từ mã nguồn:

>`grant select on ChiTieu to Truong_CN_CTY;`

### 9.3 Trưởng chi nhánh Hà Nội được phép truy xuất dữ liệu của chi nhánh Hà Nội và tất cả các chi nhánh khác.

### 9.4 Trưởng phòng được phép đọc dữ liệu dự án của tất cả phòng ban nhưng chỉ được phép ghi dữ liệu dự án thuộc phòng của mình.

#### Gán nhãn cho dữ liệu bảng DuAn

+ ### Ví dụ từ mã nguồn:

>`UPDATE DUAN SET OLS_DUAN = char_to_label ('ACCESS_DUAN', 'GH:KH:DN')`

>`WHERE mada = 'DA002';`

##### Sau đó gán nhãn cho các user là Trưởng phòng

+ ### Ví dụ từ mã nguồn:

>`BEGIN`

>`sa_user_admin.set_user_labels`

>`(policy_name => 'ACCESS_DUAN',`

>`user_name => 'NV031',`

>`max_read_label => 'BMC:NS,KT,KH:DN',`

>`max_write_label => 'BMC:NS:DN',`

>`min_write_label => 'GH',`

>`def_label => 'BMC:NS,KT,KH:DN',`

>`row_label => 'BMC:NS:DN');`

>`END;`

### 9.5 Nhân viên chỉ được đọc dữ liệu dự mình tham gia.

## Policy 10: Mỗi thông tin thu chi sẽ được đánh dấu các mức độ “Nhạy cảm”, “Không nhạy cảm”, “Bí mật” và thuộc các nhóm như “Lương”, “Quản lý”, “Vật liệu”. Nhân viên phụ trách đủ các lĩnh vực, có cấp độ phù hợp mới được phép truy xuất dữ liệu thu chi. Ngoài ra, mỗi thông tin thu chi còn quy định cấp “Quản lý” hay “Nhân viên” để xác định dữ liệu này thuộc cấp quản lý của nhân viên hay quản lý dự án. Quản lý có thể xem tất cả thông tin thu chi của nhân viên (OLS).

#### Tạo các level “Nhạy cảm”, “Không nhạy cảm”, “Bí mật”.

+ ### Ví dụ từ mã nguồn: 

>`EXECUTE sa_components.create_LEVEL ('ACCESS_CHITIEU',1000,'KNC','KHONGNHAYCAM');`

#### Tạo các compartment “Lương”, “Quản lý”, “Vật liệu”.

+ ### Ví dụ từ mã nguồn: 

>`EXECUTE sa_components.create_COMPARTMENT ('ACCESS_CHITIEU',1000,'LU','LUONG');`

### 10.1 Nhân viên phụ trách đủ các lĩnh vực, có cấp độ phù hợp mới được phép truy xuất dữ liệu thu chi.



### 10.2 Ngoài ra, mỗi thông tin thu chi còn quy định cấp “Quản lý” hay “Nhân viên” để xác định dữ liệu này thuộc cấp quản lý của nhân viên hay quản lý dự án. Quản lý có thể xem tất cả thông tin thu chi của nhân viên.
