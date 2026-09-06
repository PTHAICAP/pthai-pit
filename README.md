# P.Thai Capital — Kho Point-in-Time & Sổ tín hiệu V20 (bản công khai)

Repo này là **bản sao chỉ-đọc** của hai thứ mà P.Thai Capital công bố để bất kỳ ai cũng
kiểm chứng được, do GitHub Actions tự đẩy sau mỗi phiên giao dịch:

| Thư mục | Nội dung |
|---|---|
| `pit/` | Ảnh chụp thị trường mỗi phiên (~115 mã × 27 trường: giá, vốn hoá, P/E, P/B, ROE, động lượng, thanh khoản, điểm số), **chỉ ghi thêm, không bao giờ sửa** |
| `pit/market/` | VN-Index, trạng thái thị trường, nhiệt kế, mỗi phiên một dòng |
| `pit/revisions/` | Bản ghi lại khi nguồn dữ liệu sửa quá khứ (điều chỉnh hồi tố), bản gốc giữ nguyên |
| `pit/ots/` | Proof OpenTimestamps neo SHA256 của từng tệp vào blockchain Bitcoin |
| `pit/MANIFEST.csv` | Sổ cái: phiên, giờ chụp, trạng thái, tệp, số dòng, SHA256 |
| `pit/CORRECTIONS.csv` | Đính chính (chỉ ghi thêm): tệp nào có vấn đề, dùng tệp nào thay |
| `signals/` | Tín hiệu V20 từng phiên (`signals_<ngày>.json`) và bảng chấm (`scorecard.json`), **công bố trước khi biết kết quả** |

Bản gốc được sinh trong một repo private của chúng tôi; repo này chỉ chứa dữ liệu,
không chứa mã nguồn chiến lược.

## Vì sao kho này tồn tại

Các nguồn dữ liệu công khai ở Việt Nam trả về **giá đã điều chỉnh hồi tố**. Chúng tôi
đo được: cổ phiếu PET phiên 01/04/2026 có giá thực hiện 52,00, nhưng truy vấn cùng nguồn
sau đó trả về 35,38 (lệch 1,47 lần do sự kiện doanh nghiệp). Nghĩa là không ai dựng lại
được P/E, P/B hay điểm số của một ngày trong quá khứ nếu ngày đó không có ai ghi lại.
Dữ liệu point-in-time chỉ có một cách để có: ghi từng ngày, và chứng minh được là đã
ghi vào ngày đó.

## Tự kiểm chứng

**1. Tệp không bị sửa sau khi công bố.** So SHA256 với `pit/MANIFEST.csv`:

```bash
git clone https://github.com/PTHAICAP/pthai-pit.git && cd pthai-pit
sha256sum pit/2026/pit-2026-09-03.csv        # đối chiếu cột sha256 trong pit/MANIFEST.csv
```

Trên Windows, git có thể đổi xuống dòng LF thành CRLF khi checkout và làm hash lệch.
Repo đã có `.gitattributes` ép LF; nếu vẫn lệch, hãy chuẩn hoá trước khi băm:

```bash
tr -d '\r' < pit/2026/pit-2026-09-03.csv | sha256sum
```

**2. Tệp đã tồn tại từ lúc nào.** Hai lớp độc lập:

- Lịch sử git: mỗi phiên là một commit do GitHub Actions tạo (`git log -- pit/2026/pit-2026-09-03.csv`).
- OpenTimestamps: proof trong `pit/ots/` neo SHA256 vào Bitcoin. Kiểm bằng
  `pip install opentimestamps-client` rồi `ots verify pit/ots/2026/pit-2026-09-03.csv.ots -f pit/2026/pit-2026-09-03.csv`,
  hoặc kéo-thả cặp tệp vào https://opentimestamps.org. Proof mới tạo ở trạng thái
  *pending* vài giờ trước khi vào block; `ots upgrade <tệp>.ots` để cập nhật.

**3. Tín hiệu được công bố trước kết quả.** Mỗi `signals/signals_<ngày>.json` ghi
`signal_date` (ngày phát), `execution_date` (phiên thực thi kế tiếp), danh mục, lệnh và
lý do. So `signal_date` với thời điểm commit của tệp đó; so `ref_price` và kết quả với
bảng giá HOSE/HNX. `scorecard.json` là bảng chấm của chúng tôi, bạn không cần tin nó,
tự chấm lại được từ tệp gốc.

## Giới hạn, nói thẳng

- Dữ liệu lấy từ nguồn công khai miễn phí, không có SLA, có thể sai hoặc thiếu; rổ là
  ~115 mã vượt ngưỡng thanh khoản, không phải toàn bộ thị trường.
- Kho bắt đầu ghi từ **03/09/2026**. Giai đoạn 17/04 → 03/09/2026 pipeline không chạy;
  khoảng trống đó không lấp được.
- Kho này chứng minh **"đã công bố trước"**, không chứng minh **"kiếm được tiền"**.
  Hiệu quả thật chỉ có sao kê tài khoản giao dịch thực mới chứng minh được.
- Không phải khuyến nghị mua bán. Xem thêm: https://pthaicapital.io.vn/so-tin-hieu
  và https://pthaicapital.io.vn/phuong-phap.

## Sự cố đã ghi nhận

Xem `pit/CORRECTIONS.csv`. Đáng chú ý: phiên **04/09/2026** có bản gốc chụp lúc 10:04
sáng, tức trong giờ giao dịch, do chạy tay khi mới bật kho. Số đóng cửa thật của phiên
đó nằm ở `pit/revisions/pit-2026-09-04--seen-2026-09-06.csv`. Script ghi kho từ 06/09
từ chối ghi khi phiên chưa đóng cửa, nên lỗi này không lặp lại.

---

## English summary

This repository is a read-only, append-only public mirror of two artefacts published by
P.Thai Capital after every Vietnamese trading session: (1) point-in-time market snapshots
(`pit/`, ~115 liquid stocks × 27 fields, with SHA256 ledger, revision tracking and
OpenTimestamps proofs anchored to Bitcoin), and (2) the V20 strategy's signal files
(`signals/`), published before outcomes are known. Verify file integrity against
`pit/MANIFEST.csv` (normalise line endings to LF first on Windows), verify existence
time via git history and `ots verify`. The store starts on 2026-09-03. It proves
"published before", not "profitable". Free public data, no SLA, not investment advice.
