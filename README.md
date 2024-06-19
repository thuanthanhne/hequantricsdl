**CHƯƠNG TRÌNH QUẢN LÝ BÁN HÀNG CỦA CỬA HÀNG CÀ PHÊ CHUM**



**Họ tên: Phạm Ngọc Thuận Thanh**
**MSSV: K215480106123**
**Lớp: K57KMT**



**1. Các chức năng và yêu cầu đề bài**


* Các chức năng:
- Quản lý nhân viên: Thêm, xóa, sửa thông tin nhân viên
- Quản lý sản phẩm: Thêm, sửa, xóa thông tin hàng hóa
- Quản lý khách hàng: Thêm, sửa, xóa thông tin khách hàng
- Quản lý đơn hàng: Thêm, sửa, xóa thông tin đơn hàng
* Yêu cầu của đề bài:


- Báo cáo doanh thu theo ngày
- Báo cáo doanh thu theo tháng
- Báo cáo các sản phẩm bán chạy nhất của cửa hàng
- Tính toán tiền lương của nhân viên trong 1 tháng



**2. Các bảng của hệ thống**


- Bảng Khachhang:
  ![image](https://github.com/thuanthanhne/hequantricsdl/assets/168764508/c5de3b95-8168-416d-b1dd-0044b0c2f29e)





- Bảng Nhanvien:
  ![image](https://github.com/thuanthanhne/hequantricsdl/assets/168764508/94c3cc57-f78e-486b-aa44-94229f4c30d6)





- Bảng Sanpham:
  ![image](https://github.com/thuanthanhne/hequantricsdl/assets/168764508/33909312-9f5f-44c6-b424-8dcff2fa79f5)





- Bảng Hoadon:
  ![image](https://github.com/thuanthanhne/hequantricsdl/assets/168764508/6168797d-c660-4fda-a461-db4a70fb26cf)





- Bảng Chitiethoadon:
  ![image](https://github.com/thuanthanhne/hequantricsdl/assets/168764508/7f452f8e-25c9-439b-abc2-411250690d37)





- Bảng Luongnhanvien:
  ![image](https://github.com/thuanthanhne/hequantricsdl/assets/168764508/49f98559-76e6-4264-bcce-e09373e6168b)







**3. Thêm dữ liệu vào các bảng cần thiết**

- Bảng Khachhang:
  ![image](https://github.com/thuanthanhne/hequantricsdl/assets/168764508/edbbe424-48c2-4bc3-8d97-5e59adffeddc)





- Bảng Sanpham:
  ![image](https://github.com/thuanthanhne/hequantricsdl/assets/168764508/ebe0c645-18b8-4e23-a763-404afe7dbeb9)





- Bảng Nhanvien:
  ![image](https://github.com/thuanthanhne/hequantricsdl/assets/168764508/aa9c0719-b2fa-43eb-8c14-5c2161298a34)





- Bảng Hoadon:
  ![image](https://github.com/thuanthanhne/hequantricsdl/assets/168764508/541ea3f0-06be-4d27-9dac-f0fed1ccffd0)






  - Bảng Chitiethoadon:
    ![image](https://github.com/thuanthanhne/hequantricsdl/assets/168764508/cc8082c3-3025-475d-8cb7-e0f8d192de36)








**4. Tạo các thủ tục phù hợp với yêu cầu**




--Tạo thủ tục báo cáo doanh thu theo ngày
GO

CREATE PROCEDURE CalculateRevenueByDay

    @Ngay DATE
    
AS

BEGIN

    SELECT
    
        h.NgayHoadon,
        
        SUM(ct.Soluong * ct.Gia) AS TongDoanhThu
        
    FROM
    
        Hoadon h
        
        JOIN Chitiethoadon ct ON h.HoadonID = ct.HoadonID
        
    WHERE
    
        h.NgayHoadon = @Ngay
        
    GROUP BY
    
        h.NgayHoadon;
        
END;


--Tác giả: Phạm Ngọc Thuận Thanh ~~





--Tạo thủ tục báo cáo doanh thu theo tháng--
 GO

 
 CREATE PROCEDURE RevenueByMonth
 
    @Thang INT,
    
    
    
    @Nam INT
    
AS


BEGIN

    SELECT SUM(TongTien) AS TongDoanhThu
    
    FROM Hoadon
    
    WHERE MONTH(NgayHoadon) = @Thang AND YEAR(NgayHoadon) = @Nam
END;







--Tạo thủ tục báo cáo sản phẩm bán chạy nhất--
GO
CREATE PROCEDURE TopSellingProduct

AS

BEGIN

    SELECT sp.TenSanpham, SUM(ct.Soluong) AS TongSoluong
    
    FROM Chitiethoadon ct
    
    JOIN Sanpham sp ON ct.SanphamID = sp.SanphamID
    
    GROUP BY sp.TenSanpham
    
    ORDER BY TongSoluong DESC
    
    
END;




-- Sử dụng Trigger và Cursor để tính toán tổng tiền của các chi tiết hoá đơn sau khi chúng được thêm mới vào bảng Chitiethoadon, và sau đó cập nhật tổng tiền này vào bảng Hoadon--


GO

CREATE TRIGGER CalculateRevenue

ON Chitiethoadon

AFTER INSERT

AS

BEGIN

    DECLARE @HoadonID INT
    
    DECLARE @TongTien MONEY
    

    -- Khai báo cursor để duyệt từng dòng trong bảng inserted
    DECLARE Cur CURSOR FOR
    
    SELECT HoadonID, SUM(Soluong * Gia) AS TongTien
    
    FROM inserted
    
    GROUP BY HoadonID
    

    -- Mở cursor
    
    OPEN Cur
    

    -- Đọc dữ liệu từ cursor
    
    FETCH NEXT FROM Cur INTO @HoadonID, @TongTien
    

    -- Vòng lặp xử lý từng dòng
    
    WHILE @@FETCH_STATUS = 0
    
    BEGIN
        -- Kiểm tra giá trị NULL trước khi cập nhật
        
        IF @TongTien IS NOT NULL AND @HoadonID IS NOT NULL
        
        BEGIN
        
            -- Cập nhật bảng Hoadon với tổng tiền tương ứng
            
            UPDATE Hoadon
            
            SET TongTien = @TongTien
            
            WHERE HoadonID = @HoadonID
            
        END
        

        -- Đọc dòng tiếp theo
        
        FETCH NEXT FROM Cur INTO @HoadonID, @TongTien
        
    END
    

    -- Đóng cursor
    
    CLOSE Cur
    

    -- Hủy cursor
    
    DEALLOCATE Cur
    
END

GO







--Tạo thủ tục tính lương nhân viên--
GO

CREATE PROCEDURE CalculateEmployeeSalary

    @Thang INT,
    
    @Nam INT
    
AS

BEGIN

    -- Tính tổng doanh thu của cửa hàng trong tháng
    
    DECLARE @TongDoanhThu MONEY;
    
    
    SELECT @TongDoanhThu = SUM(TongTien)
    
    
    FROM Hoadon
    
    
    WHERE MONTH(NgayHoadon) = @Thang AND YEAR(NgayHoadon) = @Nam;
    

    -- Cập nhật bảng Luongnhanvien với lương tính toán
    
    INSERT INTO Luongnhanvien (NhanvienID, Thang, Nam, LuongCoban, Thuong, TongLuong)
    
    
    SELECT nv.NhanvienID, @Thang, @Nam, nv.LuongNhanvien, 0.05 * @TongDoanhThu, nv.LuongNhanvien + 0.05 * @TongDoanhThu
    
    
    FROM Nhanvien nv;
    
END;






**5.Thực hiện các yêu cầu của đề tài**




--Yêu cầu 1: Báo cáo doanh thu theo ngày--


EXEC CalculateRevenueByDay '2024-06-01';



--Yêu cầu 2: Báo cáo doanh thu theo tháng--


EXEC RevenueByMonth 6, 2024;




--Yêu cầu 3: Báo cáo sản phẩm bán chạy nhất--


EXEC TopSellingProduct;



--Yêu cầu 4: Tính lương tháng cho nhân viên--


EXEC CalculateEmployeeSalary 6, 2024;




--Hiển thị lương tháng cho nhân viên--

SELECT * FROM Luongnhanvien

WHERE Thang = 6 AND Nam = 2024;

  

  






