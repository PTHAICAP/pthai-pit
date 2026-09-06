# Kho Point-in-Time — P.Thai Capital

Ảnh chụp thị trường mỗi phiên, **chỉ ghi thêm, không bao giờ sửa**.

## Vì sao kho này tồn tại

Các nguồn dữ liệu công khai ở Việt Nam (VNDirect, TCBS…) trả về **giá đã điều
chỉnh hồi tố**. Chúng tôi đo được cụ thể: cổ phiếu PET phiên 01/04/2026 có giá
thực hiện **52,00**, nhưng API truy vấn hôm nay trả về **35,38** — lệch **1,47
lần** do sự kiện doanh nghiệp.

Hệ quả trực tiếp:

> Không ai có thể dựng lại P/E, P/B hay điểm số định lượng của một cổ phiếu vào
> một ngày trong quá khứ, nếu ngày đó không có ai ghi lại.

Dữ liệu point-in-time vì thế **chỉ có một cách duy nhất để sở hữu: ghi từng
ngày**. Không ghi là mất vĩnh viễn — không có tiền nào mua lại được.

Kho này đã đứt **17/04/2026 → 03/09/2026** (4,5 tháng) khi pipeline ngừng chạy.
Khoảng trống đó không thể lấp. Từ 03/09/2026 trở đi, kho chạy tự động mỗi phiên.

## Cấu trúc

```
pit/
├── MANIFEST.csv          sổ cái: as_of, captured_at, status, file, rows, sha256
├── CORRECTIONS.csv       đính chính, chỉ ghi thêm: tệp nào có vấn đề, dùng tệp nào thay
├── <năm>/pit-<ngày>.csv  115 mã × 27 trường, một file mỗi phiên
├── market/<năm>.csv      VN-Index, regime, nhiệt kế — mỗi phiên một dòng
├── revisions/            bản ghi khi nguồn sửa lại quá khứ (xem bên dưới)
└── ots/                  proof OpenTimestamps neo SHA256 của từng tệp vào Bitcoin
```

Mỗi file phiên gồm: giá, vốn hoá, P/E, P/B, ROE, hiệu suất 3T/6T/12T, vị thế so
với MA50/MA200, RSI, beta, biến động năm, thanh khoản bình quân, khoảng cách
đỉnh/đáy 52 tuần, và 4 điểm trục (định giá · chất lượng · động lượng · thanh
khoản) cùng điểm tổng hợp.

## Điều chỉnh hồi tố được ghi lại, không bị xoá

Nếu chạy lại và thấy dữ liệu của một phiên đã thay đổi so với bản đã lưu, bản
gốc **được giữ nguyên**; bản mới lưu vào `revisions/` kèm ngày phát hiện.

Bản thân việc điều chỉnh cũng là dữ liệu point-in-time có giá trị — nó cho biết
nguồn đã sửa gì và sửa lúc nào.

## Chỉ ghi sau giờ đóng cửa

Từ 06/09/2026 script từ chối ghi khi `as_of` là hôm nay mà giờ chụp trước 15:30
(giờ VN). Lý do: phiên 04/09 được chạy tay lúc 10:04 sáng và bản gốc ghi số **trong
phiên** (VN-Index 1849,74 thay vì đóng cửa 1853,08); lượt 19:58 cùng ngày có số đóng
cửa thật lại bị bỏ vì tên revision chỉ có ngày. Nay tên revision kèm giờ phút, và bản
mới trùng nội dung với bất kỳ bản nào đã có của phiên thì bỏ qua. Bản 04/09 bị lỗi được
giữ nguyên (nguyên tắc chỉ ghi thêm) và ghi vào `CORRECTIONS.csv`.

## Bản công khai

Repo này là private, nên lịch sử git ở đây không phải bằng chứng cho người ngoài.
Sau mỗi phiên, `ops/mirror_pit.sh` đẩy nguyên `pit/` và `pthai/strategy_v2/signals/`
sang repo public **https://github.com/PTHAICAP/pthai-pit** (deploy key chỉ có quyền
ghi vào repo đó). Mọi hướng dẫn kiểm chứng dưới đây áp dụng cho bản công khai.

## Kiểm chứng tính toàn vẹn

Mỗi file có SHA256 trong `MANIFEST.csv`, và mỗi phiên là một commit git riêng
do GitHub Actions tạo, mang dấu thời gian của GitHub — không phải của chúng tôi.
Muốn kiểm tra một file có bị sửa sau khi công bố hay không:

```bash
sha256sum pit/2026/pit-2026-09-03.csv     # đối chiếu với MANIFEST.csv
git log --follow -- pit/2026/pit-2026-09-03.csv
```

Trên Windows, nếu hash lệch thì gần như chắc chắn là do xuống dòng CRLF; chuẩn hoá
trước khi băm: `tr -d '\r' < <tệp> | sha256sum`. `.gitattributes` đã ép LF cho `pit/`.

Lớp thứ hai, độc lập với git và với chúng tôi: `ops/ots_stamp.py` đóng dấu
OpenTimestamps cho từng tệp phiên và cho MANIFEST (proof trong `ots/`). Kiểm bằng
`ots verify ots/2026/pit-2026-09-03.csv.ots -f 2026/pit-2026-09-03.csv` hoặc kéo-thả
tại opentimestamps.org. Proof mới ở trạng thái pending vài giờ; `ots upgrade` để cập nhật.

## Giới hạn cần biết

- Dữ liệu lấy từ nguồn công khai miễn phí, **không có SLA**, có thể sai hoặc thiếu.
- Rổ là ~115 mã vượt ngưỡng thanh khoản, **không phải toàn bộ thị trường**.
- Chỉ là dữ liệu nghiên cứu — **không phải khuyến nghị mua bán**.
