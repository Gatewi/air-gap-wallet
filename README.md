# Cardano Air-Gap Wallet (Ví Cardano Ngoại Tuyến)

Dự án này chia các kịch bản Bash quản lý tài sản Cardano trước đây thành hai phần độc lập: **Online (Hot)** và **Offline (Cold)**. Hệ thống cho phép bạn xây dựng giao dịch trực tuyến, ký giao dịch ngoại tuyến (bảo mật khóa riêng tư bằng mật khẩu) và gửi giao dịch lên mạng lưới mà không bao giờ để lộ khóa riêng tư (Private Key) ra internet.

Hệ thống hỗ trợ quản lý **nhiều ví độc lập** thông qua cấu trúc phân mục theo tên ví (ví dụ: `wallets/FIMI/`).

Dữ liệu giao dịch được chuyển đổi qua lại giữa hai máy tính bằng **Mã QR** (hoặc tệp văn bản/chuỗi hex nếu không có camera hoặc truyền qua USB).

---

## Cấu trúc Thư mục

```text
cold-wallet/
├── docs/                        # Thư mục tài liệu kỹ thuật & vận hành
│   ├── README.md                # Tài liệu hướng dẫn sử dụng & tổng quan hệ thống
│   ├── PROCESS_AND_SECURITY.md  # Quy trình giao dịch & cơ chế bảo mật khóa
│   ├── Tech-document.md         # Đặc tả kỹ thuật toàn diện (v5 Conway Native)
│   ├── tech-document-simple.md  # Đặc tả kiến trúc tối giản (MVP Lite)
│   ├── WALLET_GENERATION_GUIDE.md # Hướng dẫn chi tiết sinh khóa CIP-1852
│   ├── guide-install-mobile.md  # Hướng dẫn kết nối USB & cài đặt APK Android
│   ├── MIDNIGHT_INTEGRATION_REFERENCE.md # Nghiên cứu tích hợp Midnight ZK-Proof
│   ├── MULTI_CHAIN_BTC_ETH_REFERENCE.md  # Thiết kế tích hợp đa chuỗi BTC & ETH
│   ├── security_audit.html      # Báo cáo kiểm toán bảo mật (HTML)
│   └── security_audit.pdf       # Báo cáo kiểm toán bảo mật (PDF)
│
├── mobile/                      # Ứng dụng Di Động Android Cold Wallet (Offline - Air-Gap APK v5)
│   ├── lib/                     # Mã nguồn Flutter (CIP-1852, Conway Signer, AirGapShield)
│   ├── android/                 # Cấu hình Android Native (Loại bỏ 100% quyền INTERNET)
│   └── pubspec.yaml             # Thư viện phụ thuộc (version: 3.0.0+5)
│
├── mobile_online/               # Ứng dụng Di Động Android Hot Wallet (Online - Watch-Only APK v5)
│   ├── lib/                     # Mã nguồn Flutter (Blockfrost API, Conway TxBuilder, Submit)
│   ├── android/                 # Cấu hình Android Native (Quyền INTERNET & CAMERA)
│   └── pubspec.yaml             # Thư viện phụ thuộc (version: 3.0.0+4)
│
├── .github/workflows/           # Tự động hóa CI/CD
│   └── build-apk.yml            # Tự động build và xuất file APK Release qua GitHub Actions
│
├── online/                      # Phần chạy trên Máy tính Online (Hot)
│   ├── config.env               # Cấu hình Blockfrost API & Mạng lưới
│   ├── blockfrost_helper.sh     # Hàm bổ trợ gọi API Blockfrost
│   ├── online_main.sh           # Menu chính phần Online (Tạo Tx, Submit)
│   └── test_e2e_withdrawal.sh   # Kịch bản kiểm thử tự động rút thưởng Staking
│
└── offline/                     # Phần chạy trên Máy tính Offline (Cold)
    ├── config.env               # Cấu hình Mạng lưới (Ngoại tuyến)
    ├── wallet-generate.sh       # Kịch bản tạo mới/khôi phục ví
    ├── offline_main.sh          # Menu chính phần Offline (Ký Tx ngoại tuyến)
    └── wallets/                 # Thư mục lưu trữ các ví đã tạo
        └── <WALLET_NAME>/       # Thư mục riêng của từng ví (Ví dụ: FIMI)
            ├── payment.addr     # Địa chỉ nhận tiền công khai
            ├── stake.addr       # Địa chỉ ủy quyền stake
            ├── payment.vkey     # Khóa xác minh thanh toán công khai
            ├── stake.vkey       # Khóa xác minh ủy quyền công khai
            ├── payment.pub      # Khóa công khai thô
            ├── payment.skey.enc # Khóa ký thanh toán ĐÃ MÃ HÓA
            ├── stake.skey.enc   # Khóa ký ủy quyền ĐÃ MÃ HÓA
            ├── phrase.prv.enc   # Cụm 24 từ khôi phục ĐÃ MÃ HÓA
            ├── tx_raw.txt       # File giao dịch thô
            ├── tx_raw_qr.png    # Ảnh QR giao dịch thô
            ├── tx_signed.txt    # File giao dịch đã ký
            └── tx_signed_qr.png # Ảnh QR giao dịch đã ký
```

---

## Yêu cầu Hệ thống & Công cụ

Đảm bảo các công cụ sau được cài đặt trên các máy tương ứng. Trên Ubuntu, các công cụ quét và tạo mã QR mặc định sẽ không có sẵn, bạn cần cài đặt bằng lệnh:

```bash
sudo apt update
sudo apt install -y zbar-tools qrencode
```

### Máy Online (Hot):
*   `curl` & `jq` (Dùng để truy vấn dữ liệu từ API Blockfrost).
*   `cardano-cli` (Phiên bản Conway trở lên - dùng để xây dựng cấu trúc giao dịch raw). https://github.com/IntersectMBO/cardano-node/releases/tag/11.0.1
*   `qrencode` (Dùng để chuyển giao dịch raw thành mã QR hiển thị trên Terminal).
*   *Tùy chọn:* `zbar-tools` (Gồm `zbarcam` và `zbarimg` để quét mã QR đã ký từ máy Offline).

### Máy Offline (Cold):
*   `cardano-cli` (bắt buộc) và `cardano-address` (tùy chọn - nếu không tìm thấy cardano-address, hệ thống sẽ tự động dùng cardano-cli để sinh khóa, địa chỉ và ký giao dịch).
*   `openssl` (Hỗ trợ mã hóa khóa riêng tư).
*   `qrencode` (Chuyển giao dịch đã ký thành mã QR).
*   *Tùy chọn:* `zbar-tools` (Quét mã QR giao dịch raw từ máy Online).

---

## Cơ chế Bảo mật khóa riêng tư

1.  **Mã hóa AES-256-CBC với PBKDF2**: Tất cả các tệp chứa khóa riêng tư (`phrase.prv`, `payment.skey`, `stake.skey`) đều được mã hóa bằng OpenSSL với mật khẩu người dùng tự chọn. Số vòng lặp băm PBKDF2 được đặt ở mức cực kỳ bảo mật là `100,000` lần.
2.  **Xóa sạch dấu vết trên RAM Disk**: Sau khi tạo khóa hoặc ký xong giao dịch, các tệp chứa khóa không mã hóa trong bộ nhớ đệm `/dev/shm` (RAM Disk) sẽ lập tức bị ghi đè nhiều lần và xóa vĩnh viễn khỏi ổ cứng bằng lệnh `shred -u` để tránh bị khôi phục dữ liệu:
    ```bash
    shred -u /dev/shm/cardano-airgap-sign-XXXX/payment.skey.tmp
    ```
3.  **Cô lập và Phân vùng**: Mỗi ví được lưu trong một thư mục con riêng biệt dưới `offline/wallets/<WALLET_NAME>/`. Điều này ngăn chặn việc nhầm lẫn hoặc ghi đè khóa giữa các ví khác nhau.

---

## Quy trình Thực hiện Giao dịch (6 Bước)

### Bước 1: Tạo/Khôi phục ví (Offline)
1.  Truy cập thư mục `offline/`.
2.  Chạy `./offline_main.sh` và chọn **Option 1 (Tạo mới / Khôi phục ví)**.
3.  Nhập tên ví (ví dụ: `FIMI`). Thư mục `wallets/FIMI/` sẽ được tự động tạo.
4.  Lựa chọn tạo mới hoặc nhập cụm 24 từ khôi phục.
5.  Nhập mật khẩu bảo vệ ví.
6.  Hệ thống sinh ra địa chỉ ví và các tệp mã hóa khóa riêng tư `.enc` trong `wallets/FIMI/`. Khi tạo ví mới, hệ thống tự động hiển thị 24 từ khôi phục theo dạng bảng (4 từ/hàng, hiển thị 3 hàng mỗi trang) để bạn lưu trữ an toàn.
7.  *Tùy chọn:* Khi muốn xem lại cụm từ khôi phục của ví đã tạo bất kỳ lúc nào, bạn chọn **Option 3 (Xem / Hiển thị 24 từ khôi phục)** trên menu `offline_main.sh` và nhập mật khẩu bảo mật ví để giải mã.

### Bước 2: Cấu hình API Blockfrost (Online)
1.  Tạo tệp cấu hình `online/config.env` bằng cách sao chép từ tệp ví dụ mẫu:
    ```bash
    cp online/config.env.example online/config.env
    ```
2.  Đăng ký tài khoản và lấy Project ID miễn phí tại [Blockfrost.io](https://blockfrost.io).
3.  Mở tệp `online/config.env` và điền khóa của bạn vào biến:
    ```bash
    BLOCKFROST_API_KEY="preprod_xxxxxxxxxxxxxxxx"
    ```

### Bước 3: Kiểm tra số dư & UTXO (Online)
1.  Chạy `./online_main.sh` trên máy online và chọn **Option 1 (Check Balance / UTXOs)**.
2.  Nhập tên ví Cardano từ máy offline (ví dụ: `FIMI`). Hệ thống sẽ tự động tìm kiếm địa chỉ ví tại `../offline/wallets/FIMI/payment.addr`.
3.  Xem danh sách các UTXO hiện tại và tổng số dư khả dụng.

### Bước 4: Tạo giao dịch trực tuyến (Online)
1.  Chọn **Option 2 (Build Raw Transaction)** trên menu `online_main.sh`.
2.  Nhập tên ví `FIMI` để tự động xác định địa chỉ gửi.
3.  Chọn số thứ tự UTXO muốn tiêu dùng từ danh sách UTXO hiển thị.
4.  Nhập địa chỉ ví người nhận và số ADA muốn chuyển.
5.  Kịch bản sẽ tự động tạo giao dịch thô và xuất ra file `tx_raw.txt` và `tx_raw_qr.png` trực tiếp vào thư mục ví `../offline/wallets/FIMI/` (nếu chạy trên cùng workspace) hoặc tại thư mục hiện hành.

### Bước 5: Ký giao dịch ngoại tuyến (Offline)
1.  Chạy `./offline_main.sh` trên máy offline, chọn **Option 2 (Sign Raw Transaction)**.
2.  Nhập tên ví thực hiện ký (ví dụ: `FIMI`).
3.  Chọn phương thức nhập giao dịch: Quét trực tiếp bằng Webcam, Đọc file ảnh mã QR, Dán chuỗi văn bản, hoặc **chọn Option 4 để đọc trực tiếp từ tệp văn bản** (mặc định sẽ tải từ `wallets/FIMI/tx_raw.txt`).
4.  **Xem chi tiết Giao dịch & Xác nhận**: Hệ thống sẽ giải mã cấu trúc giao dịch thô bằng `cardano-cli debug transaction view` để hiển thị chi tiết Kỷ nguyên (Era), Phí giao dịch (Fee), các UTXO đầu vào (Inputs) và địa chỉ nhận cùng số lượng ADA/Tokens đầu ra (Outputs). Người dùng cần kiểm tra kỹ và nhập `y` để xác nhận đồng ý ký trước khi tiếp tục.
5.  Nhập mật khẩu ví để giải mã khóa và tiến hành ký giao dịch ngoại tuyến.
6.  Hệ thống ký giao dịch và tự động lưu tệp đã ký vào `wallets/FIMI/tx_signed.txt` và mã QR `wallets/FIMI/tx_signed_qr.png`.

### Bước 6: Đọc giao dịch đã ký & Gửi lên Blockchain (Online)
1.  Chọn **Option 3 (Read Signed Transaction & Submit)** trên menu `online_main.sh`.
2.  Hệ thống hỏi tên ví, hãy nhập `FIMI`.
3.  Chọn **Option 4 để đọc trực tiếp từ tệp văn bản** (mặc định sẽ kiểm tra `../offline/wallets/FIMI/tx_signed.txt`).
4.  Giao dịch sẽ được truyền lên mạng lưới Cardano qua Blockfrost API. Khi thành công, liên kết Cardanoscan sẽ hiển thị để bạn theo dõi.

---

## Quy trình Ủy thác Stake Pool & Bỏ phiếu DRep (Conway Era)

Trong kỷ nguyên Conway, Cardano gộp việc đăng ký khóa stake, ủy quyền biểu quyết DRep và ủy thác Stake Pool thành các chứng chỉ thống nhất. Dự án hỗ trợ quy trình này hoàn toàn ngoại tuyến:

### 1. Trên Ứng dụng Di Động (Cardano Hot Wallet APK):
* Truy cập nút **"ỦY THÁC STAKE POOL & DREP (CONWAY)"** ngay trên Màn hình chính (`HomeScreen`).
* **Hình thức ủy thác**: Mặc định là **Chỉ Ủy Thác Stake Pool**, ngoài ra hỗ trợ **Cả Pool & DRep** hoặc **Chỉ DRep**.
* **Stake Pool cài sẵn**:
  - **FIMI (Mặc định)**: `pool1rct5guwnp6nk9f4ldn5tr3s42j0nm5rxeayv56lxua5ck47ex39`
  - **FIDA**: `pool1jj2hvts3ryffy5txm7w0am3zt7s7sehdf8rrwfesjmr7shv0mgl`
  - **GRAB**: `pool1fmgus36uc0jf0wlspfvzwfn04mz34kwrensmqgfzkv7t58dejw4`
  - Hoặc nhập/quét mã QR Pool ID bất kỳ.
* **DRep cài sẵn**:
  - **DRep FIMI (Mặc định)**: `drep1xgzlpyy3pvywm5wvhkvyz9cpga5wylme6wdu6l9vn5g6cp38jdj`
  - **Always Abstain**: Bỏ phiếu trắng.
  - **Always No Confidence**: Luôn bất tín nhiệm.
  - Hoặc nhập/quét mã QR DRep ID bất kỳ.
* **Tự động tính cọc & phí**: Nhận diện khóa Stake đã đăng ký (0 ADA cọc) hay chưa đăng ký (nộp 2 ADA cọc) và sinh mã QR giao dịch thô (`TxBodyConwayDelegation`).

### 2. Trên Máy Tính (Bash CLI):
1.  **Khởi tạo Giao dịch Ủy thác (Online)**:
    *   Chạy `./online_main.sh` trên máy online và chọn **Option 4 (Ủy thác Stake Pool & DRep)**.
    *   Nhập tên ví (ví dụ: `FIMI`). Hệ thống tự động đọc khóa `stake.vkey` và địa chỉ stake `stake.addr`.
    *   Hệ thống gọi API Blockfrost để kiểm tra xem khóa ủy thác đã được đăng ký trên chuỗi chưa:
        *   **Chưa đăng ký**: Hệ thống tự động tính thêm 2 ADA tiền cọc khóa (Key Deposit) và chọn tạo chứng chỉ `registration-stake-and-vote-delegation-certificate`.
        *   **Đã đăng ký**: Chỉ cần tạo chứng chỉ ủy quyền `stake-and-vote-delegation-certificate` không mất tiền cọc.
    *   Chọn phương án ủy quyền **DRep** (Mặc định là **DRep FIMI** - ID: `drep1xgzlpyy3pvywm5wvhkvyz9cpga5wylme6wdu6l9vn5g6cp38jdj`).
    *   Chọn **Stake Pool** để nhận phần thưởng (Mặc định: **Pool FIMI** - ID: `pool1rct5guwnp6nk9f4ldn5tr3s42j0nm5rxeayv56lxua5ck47ex39`).
    *   Hoặc chọn các Pool cài sẵn khác: **Pool FIDA** (ID: `pool1jj2hvts3ryffy5txm7w0am3zt7s7sehdf8rrwfesjmr7shv0mgl`) hoặc **Pool GRAB** (ID: `pool1fmgus36uc0jf0wlspfvzwfn04mz34kwrensmqgfzkv7t58dejw4`), hoặc nhập/quét Pool ID tùy ý.
    *   Chọn UTXO thanh toán phí và tiền đặt cọc (nếu có).
    *   Hệ thống xuất giao dịch thô chứa chứng chỉ ủy thác vào `wallets/FIMI/tx_raw.txt` và hiển thị mã QR.

2.  **Ký giao dịch ngoại tuyến (Offline)**:
    *   Chạy `./offline_main.sh` trên máy offline và chọn **Option 2 (Sign Raw Transaction)**.
    *   Nhập tên ví `FIMI` và mật khẩu.
    *   Hệ thống sẽ **tự động phát hiện** tệp `stake.skey.enc` bên cạnh `payment.skey.enc`.
    *   Hệ thống tiến hành giải mã an toàn cả hai khóa thô vào RAM Disk (`/dev/shm`), thực hiện ký đồng thời bằng cả khóa thanh toán và khóa ủy quyền (vì giao dịch ủy thác yêu cầu chữ ký của khóa Stake để xác thực quyền sở hữu).
    *   Sau khi ký, các khóa thô lập tức bị xóa sạch bằng `shred` để bảo mật tuyệt đối. Giao dịch đã ký được xuất ra `wallets/FIMI/tx_signed.txt` dưới dạng QR/Hex.

3.  **Gửi giao dịch đã ký (Online)**:
    *   Thực hiện tương tự **Bước 6** thông thường để gửi giao dịch đã ký lên mạng lưới thông qua Blockfrost.

---

## Quy trình Rút Thưởng Staking & Hủy Cọc Khóa Stake (Conway Era)

Hệ thống hỗ trợ tạo lệnh rút phần thưởng Staking trực tuyến trên **Hot Wallet** (`online_main.sh` & ứng dụng di động `mobile_online`) và duyệt ký an toàn ngoại tuyến trên **Cold Wallet** (`offline_main.sh` & ứng dụng di động `mobile`):

### 1. Trên Ứng dụng Di Động (Flutter Mobile):
* **Hot Wallet (Watch-Only)**:
  * Ngay trên Màn hình chính, Thẻ **Phần Thưởng Staking Khả Dụng** tự động cập nhật số dư thưởng từ Blockfrost.
  * Nhấn nút **Rút Thưởng** để mở màn hình cấu hình lệnh rút.
  * Hỗ trợ 2 chế độ:
    1. **Chỉ rút thưởng**: Tiếp tục ủy thác để nhận thưởng các epoch kế tiếp.
    2. **Rút thưởng & Hủy cọc khóa Stake (+2 ADA)**: Thu hồi 2 ADA tiền cọc ban đầu.
  * Tùy chọn chuyển về chính ví hoặc gửi đến địa chỉ người nhận bất kỳ. Phí giao dịch được tự động trừ từ số tiền thưởng rút.
  * Ứng dụng sinh mã QR giao dịch thô (`TxBodyConway`).
* **Cold Wallet (Offline)**:
  * Quét mã QR giao dịch thô. Ứng dụng tự động phân tích bóc tách **Key 5 (Withdrawals)** và hiển thị rõ ràng số ADA thưởng rút, số tiền cọc hoàn lại và địa chỉ nhận.
  * Cold Wallet tự động phát hiện và kích hoạt chữ ký của cả **Payment Key** lẫn **Stake Key** (Stake Witness) khi người dùng mở khóa bằng mật khẩu/sinh trắc học.
  * Xuất mã QR giao dịch đã ký (`TxConway`) để Hot Wallet quét và submit lên blockchain.

### 2. Trên Bộ Công Cụ Dòng Lệnh Bash:
* **Online (Hot)**: Chạy `./online_main.sh` và chọn **Option 5 (Rút thưởng Staking)**.
* **Offline (Cold)**: Chạy `./offline_main.sh` và chọn **Option 2 (Sign Raw Transaction)**. Hệ thống tự động nhận diện lệnh rút thưởng trong `debug transaction view` và tự động dùng thêm khóa Stake để ký.
* **Kiểm thử CLI**: Chạy `./test_e2e_withdrawal.sh` trong thư mục `online/` để xác minh nhanh toàn bộ cú pháp và hàm.

---

## Kiểm thử Tự động Luồng Giao dịch E2E (Cục bộ)

Để hỗ trợ xác minh nhanh tính toàn vẹn của mã nguồn trước khi cập nhật dự án mà không cần thực hiện thủ công các bước trung gian, kịch bản kiểm thử tự động End-to-End (E2E) được cung cấp tại: `online/test_e2e_transfer.sh`.

> [!NOTE]
> Kịch bản kiểm thử này được thiết lập trong `.gitignore` nhằm tránh việc vô tình đồng bộ lên GitHub công khai, giúp bảo vệ an toàn cho thông tin và cấu hình môi trường phát triển cục bộ.

### Yêu cầu chuẩn bị:
1. Thiết bị chạy thử nghiệm cần có kết nối mạng và đã cấu hình chính xác Blockfrost API Key trong `online/config.env`.
2. Ví dùng để kiểm thử (ví dụ: `FIMI` trên mạng Cardano Preprod) cần có số dư khả dụng tối thiểu là **11.5 ADA** (để đáp ứng yêu cầu UTXO tối thiểu của kỷ nguyên Conway và phí giao dịch khi tự gửi cho chính mình).
3. Đã có ví ngoại tuyến được tạo sẵn (tức là đã có thư mục ví tại `offline/wallets/<WALLET_NAME>`).

### Cách sử dụng:
1. Di chuyển vào thư mục `online/`:
   ```bash
   cd online
   ```
2. Khởi chạy kịch bản kiểm thử:
   ```bash
   ./test_e2e_transfer.sh
   ```
3. Kịch bản sẽ quét và hiển thị danh sách các ví khả dụng có trong thư mục `offline/wallets/`. Vui lòng chọn ví bạn muốn dùng để test.
4. Menu chính cung cấp các chức năng sau:
   * **1. CHẠY TOÀN BỘ LUỒNG TỰ ĐỘNG (Build -> Sign -> Submit)**: Tự động chạy tuần tự cả 3 bước (Tạo giao dịch thô -> Yêu cầu nhập mật khẩu giải mã để ký -> Gửi giao dịch đã ký lên Cardano Preprod).
   * **2. Chạy Bước 1: Khởi tạo giao dịch thô (Online)**: Chỉ khởi tạo giao dịch tự chuyển khoản 10 ADA cho chính ví đó và xuất file `tx_raw.txt`.
   * **3. Chạy Bước 2: Ký giao dịch ngoại tuyến (Offline)**: Yêu cầu nhập mật khẩu để ký ngoại tuyến file `tx_raw.txt` và xuất file `tx_signed.txt`.
   * **4. Chạy Bước 3: Gửi giao dịch đã ký lên mạng (Online)**: Gửi file `tx_signed.txt` lên mạng lưới thông qua Blockfrost.
   * **5. Chọn ví khác**: Thay đổi ví thực hiện kiểm thử.
   * **6. Thoát kịch bản kiểm thử**.


---

## 7. PHIÊN BẢN v5 - NÂNG CẤP BẢO MẬT CHỮ KÝ, CHUẨN CIP-1852 & BIP-39 PASSPHRASE

*(Tệp phân phối: `cardano-cold-wallet_v5.apk` / `cardano-cold-wallet.apk` • Bản build phát hành: **02/11/2026** • Kích thước: **80.2 MB**)*

Ở phiên bản **v5** (đã hoàn thành kiểm toán bảo mật và vượt qua 31/31 unit test):
- **Chuẩn Quốc Tế Bắt Buộc**: Bắt buộc tạo địa chỉ ví theo chuẩn **CIP-1852 (BIP32-Ed25519)**, đồng bộ và tương thích 100% với ví chính thức của IOG (**Lace**), Daedalus, Eternl, Yoroi và cardano-cli.
- **Hỗ Trợ Đa Độ Dài Mnemonic**: Hỗ trợ 12 từ, 15 từ, và 24 từ BIP-39 tiêu chuẩn.
- **Tính Năng "Từ Thứ 25" Không Trạng Thái & Khóa Cứng (Phương Án B)**: Tuyệt đối không lưu Private Key của ví có Passphrase trên máy; yêu cầu nhập lại từ thứ 25 khi ký. Đặc biệt áp dụng **Quy chuẩn bảo vệ cứng (Strict Enforcement)**: Bắt buộc Passphrase $\ge 12$ ký tự gồm đủ 4 nhóm (hoa, thường, số, ký tự đặc biệt) để khắc phục điểm yếu 2048 vòng băm của BIP-39 trước các dàn máy đào GPU/ASIC.
- **Cơ chế Kép (Dual Derivation)**: Hỗ trợ khôi phục ví cũ theo chuẩn "Legacy v1.0", đảm bảo không bao giờ gián đoạn quyền truy cập tài sản của người dùng.
- **Vá Lỗ Hổng SEC-02 & Strict Change-Address Guard**: Cơ chế phòng vệ chống tấn công đánh tráo tiền thừa (Change Hijacking). Đã quét kiểm tra toàn diện tất cả các output từ vị trí 1 đến cuối cùng trong `tx_signer.dart`, tự động chặn cứng quá trình ký nếu phát hiện tiền thừa chảy về địa chỉ lạ ngoài danh sách ví.
- **Deep Memory Zeroization**: Tự động xóa sạch mọi mảng byte nhạy cảm (`paymentPrivateKey`, `stakePrivateKey`, `seed` 512-bit, `entropyBytes`, `inputBytes`, `keyAndIv`, `decryptedBytes`) trong bộ nhớ RAM ngay sau khi xử lý xong (`0x00`).
- **Triệt Tiêu Rò Rỉ Clipboard**: Tự động xóa sạch hoàn toàn Clipboard hệ điều hành (`SafeClipboard.clear()`) ngay khi dán cụm từ khôi phục; loại bỏ hoàn toàn nút copy ở các màn hình hiển thị seed-phrase.
- **Vá Lỗ Hổng SEC-01 & SEC-04 (Hệ thống Script)**:
  - **SEC-01**: Tích hợp Strict Change Guard vào `offline/offline_main.sh`, cảnh báo đỏ và yêu cầu gõ xác nhận rủi ro nếu phát hiện output tiền thừa lạ.
  - **SEC-04**: Đưa toàn bộ ví vào thư mục cô lập `offline/wallets/<WALLET_NAME>` và cấu hình `.gitignore` chặn rò rỉ khóa lên kho mã nguồn.
- **Mã Hóa Tăng Cường**: Nâng số vòng lặp PBKDF2 AES-256-CBC lên tới 50.000–100.000 vòng lặp (50.000 trên Flutter Mobile App, 100.000 trên Bash Script).

---

## 8. TÀI LIỆU NGHIÊN CỨU & MỞ RỘNG KIẾN TRÚC

Hệ thống cung cấp các tài liệu thiết kế và nghiên cứu kỹ thuật chuyên sâu phục vụ lộ trình mở rộng dài hạn:

* **Tích Hợp Mạng Midnight (Privacy Partner Chain)**: Xem chi tiết phân tích mạch ZK-SNARKs, Dual-Ledger (Public/Private State), Compact Smart Contracts và giao thức Proof-Server ngoại tuyến tại [MIDNIGHT_INTEGRATION_REFERENCE.md](file:///c:/setup/Antigravity/cold-wallet/docs/MIDNIGHT_INTEGRATION_REFERENCE.md).
* **Thiết Kế Ví Lạnh Đa Chuỗi (Cardano + Bitcoin + Ethereum)**: Xem chi tiết đặc tả mật mã secp256k1/ed25519, giao thức ký ngoại tuyến PSBT (BIP-174) và EIP-1559 RLP tại [MULTI_CHAIN_BTC_ETH_REFERENCE.md](file:///c:/setup/Antigravity/cold-wallet/docs/MULTI_CHAIN_BTC_ETH_REFERENCE.md).
* **Báo Cáo Kiểm Toán An Ninh Mật Mã**: Xem kết quả thẩm định và rà soát lỗ hổng tại [security_audit.html](file:///c:/setup/Antigravity/cold-wallet/docs/security_audit.html) hoặc tệp in ấn [security_audit.pdf](file:///c:/setup/Antigravity/cold-wallet/docs/security_audit.pdf).
