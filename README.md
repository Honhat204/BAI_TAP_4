# BAI_TAP_4
# KHAI THÁC N8N ĐỂ TỰ ĐỘNG ĐĂNG BÀI LÊN WORDPRESS
1. SỬ DỤNG DOCKER TRÊN UBUNTU ĐỂ TẠO 1 file docker-compose.yml chứa:

- Mariadb: sử dụng image: mariadb:latest để làm hệ quản trị csdl cho wordpress, thêm các biến môi trường: TZ: "Asia/Ho_Chi_Minh", MARIADB_ROOT_PASSWORD, 
MARIADB_DATABASE, MARIADB_USER, MARIADB_PASSWORD (giá trị tuỳ ý)

- Phpmyadmin: sử dụng image: phpmyadmin:latest để đăng nhập vào mariadb rồi tạo csdl trống (chỉ để xem, ko cần tạo bảng từ đây, wordpress sẽ làm hết), khai báo biến môi trường: PMA_HOST: <tên service mariadb>, PMA_ARBITRARY: 1

- WordPress: sử dụng image: wordpress:latest, truyền các tham số môi trường cho wordpress là các thông tin truy cập csdl mariadb, tạo bởi Phpmyadmin, khai báo biến môi trường: WORDPRESS_DB_HOST: <tên service mariadb>, WORDPRESS_DB_NAME, WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD (giá trị theo mariadb đã khai báo)

- Cloudflared: sử dụng image: cloudflare/cloudflared:latest , full command và token lấy từ dashboard của cloudflare, dùng AI chuyển sang dạng docker compose
N8n : sử dụng image: n8nio/n8n:latest, nhớ truyền biến môi trường WEBHOOK_URL theo sub-domain đã add router cho cloudflared tunnel (ví dụ: WEBHOOK_URL=https://k58-n8n.tdh.io.vn/ )

- Yêu cầu: sau khi có 5 service này trong file docker-compose.yml :

- pull các images về và chạy chúng (up -d)

- Kiểm tra các service đã running ok (ko bị restart liên tục)
  
- Cấu hình cloudflare tunnel add router để public wordpress lên sub-domain1 (dùng để truy cập wordpress)

- Cấu hình cloudflare tunnel add router để public Phpmyadmin lên sub-domain2 (dùng để truy cập phpmyadmin)

- Cấu hình cloudflare tunnel add router để public n8n này lên sub-domain3 (dùng để truy cập và cấu hình n8n)

- Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu chưa có bảng nào!
   
- Truy cập sub-domain1 để cài đặt wordpress (làm theo hướng dẫn của wordpress)

- Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu có những bảng dữ liệu nào sau khi cài wp

- Tạo 1 bài viết trong wordpress giới thiệu về bản thân sinh viên: thông tin cá nhân, sở thích, ... bài viết có thể chứa hình ảnh, âm thanh, video, ...

- Tạo 1 bài viết trong wordpress giới thiệu về nhữn kiến thức mà em đã học được ở môn Phát triển ứng dụng với mã nguồn mở

- Truy cập sub-domain3 để cấu hình n8n:
  
- tạo tài khoản admin : nhớ điền đúng email

- Send me a Licence key, bước này điền đủ thông tin, làm chậm sẽ thấy mục gửi License key về mail (n8n sẽ gửi email KEY cho dùng), check email để lấy KEY
Activate License key: vào trang chủ => SETTING (góc dưới trái) => Usage and plan => Enter activation key: paste key từ email vào đây => Activate => sẽ nhận đc thông báo (góc dưới phải) Your Registered Community Edition has been successfully activated.

- Create workflow (home page => overview => Create workflow)

- Add trigger node: tìm node: Telegram => OnMessage ; cấu hình Credential: Set up Credential => cần Nhập Access Token

- Access Token thì lấy ở Telegram qua việc chát với @BotFather

- Cần chát với bot @BotFather để đẻ ra bot mới của riêng mình. bot này sẽ là nơi nhận lệnh (promt) để AI sinh html => n8n sẽ dùng html này để đăng bài lên wp

- Sau khi tạo bot mới cần copy lấy Token, và chát lần đầu với bot mới này, nội dung bất kỳ (bước này quan trọng!)

- Add (nối tiếp vào sau node Telegram Trigger) node: AI Google Gemini => Message a model => Set up Credential => cần Nhập API KEY
  
- Lấy API KEY tại trang: https://aistudio.google.com => https://aistudio.google.com/api-keys
 
- cần tạo project mới, sẽ lấy được API KEY

- Nhập API Key lên giao diện n8n

- kéo thả nội dung đã chát với bot của telegram (phía bên trái) vào nội dung phần PROMPT kết quả được {{ $json.message.text }}, cần gõ thêm vào sau {{ $json.message.text }} để promt dài hơn : vd ({{ $json.message.text }}. Kết quả sinh ra ở định dạng HTML+CSS để tôi dùng HTML+CSS này tạo bài viết cho wordpress.)

- Turn on Output Content as JSON : để kết quả trả về dạng json

- Có thể thử nghiệm các thành phần khác trong Options (add Options: System message, ...) => đưa ra cái nào đáng dùng?

- Add (nối tiếp vào sau node Message a model) node: Code in JavaScript

- Code js ở dạng này, có thể phải thay đổi tuỳ theo json AI trả về.
```text
// 1. lấy dữ liệu gốc
const rawText = $input.first().json.content.parts[0].text;
// 2. Chuyển đổi chuỗi (đã được bọc JSON) thành Object trong JavaScript
const cleanData = JSON.parse(rawText);
// 3. Trả về kết quả định dạng lại gọn gàng cho n8n sử dụng
return {
  title: cleanData.post_title,
  content: cleanData.post_content
};
```
- Add (nối tiếp vào sau node Code in JavaScript) node: WordPress => Create a Post

- Set up Credential: vào wp tại url: https://sub-domain1/wp-admin => vào mục Tài Khoản => chọn user đã tạo lúc setup wordpress => Mật khẩu ứng dụng => Nhập n8n và bấm "Thêm mật khẩu ứng dụng" => copy chuỗi 24 kí tự : Đây là mật khẩu ứng dụng => paste vào mục Password của n8n Credential

- Wordpress URL: điền giá trị https://sub-domain1/ (giá trị này cũng khai báo trong biến môi trường WEBHOOK_URL của n8n)
- Ignore SSL Issues (Insecure): TURN ON
- Cấu hình node Create a Post: bấm nút Execute previous nodes để thấy trường giá trị của node trước trả về, kéo nội dung phần title (bên trái) vào trường title, tương tự kéo nội dung content vào content

- Add field (Thêm thuộc tính): Status == Publish (bài đăng sẽ ở trạng thái xuất bản ngay lập tức, mặc định nó ở giá trị Draft bản nháp)

- PUBLISH flow (góc trên phải) Nút này thực hiện việc xuất bản flow <=> flow sẽ tự động thực thi khi thoả mãn điều kiện trigger

- Kết quả cuối cùng cần đặt được:

từ điện thoại, chát với telegram bot

nội dung chát được tự động gửi tới node Telegram trigger => Gửi tới Google Gemini Message a model (bản chất là gửi Prompt) : Nhận về json kết quả của Prompt => Gửi sang node Code in JavaScript để tách tiêu đề và nội dung => gửi đến node WordPress để Create a Post(đăng bài) với tiêu đề và nội dung từ node trước gửi sang.
f5 wordpress để thấy bài viết mới đã lên sóng.

Chụp ảnh quá trình thao tác/cấu hình/các kết quả trung gian đạt được

- Nhận xét thành quả đạt được!!!

## Bài làm

-Tạo file:
nano docker-compose.yml
<img width="990" height="953" alt="Ảnh chụp màn hình 2026-05-25 094415" src="https://github.com/user-attachments/assets/d9799c27-bedd-4685-ba8b-b166f9112952" />

<img width="1037" height="965" alt="Ảnh chụp màn hình 2026-05-25 094438" src="https://github.com/user-attachments/assets/d9dd11cc-4838-4b01-8096-0d06270785b4" />

<img width="1194" height="895" alt="Ảnh chụp màn hình 2026-05-25 101404" src="https://github.com/user-attachments/assets/2786b660-4b0b-4545-a9b6-c87eecb716aa" />

-TẠO THƯ MỤC

<img width="1067" height="526" alt="Ảnh chụp màn hình 2026-05-25 094501" src="https://github.com/user-attachments/assets/116f1919-74dd-48d5-871b-4f1f55ac7b50" />

CHẠY DOCKER
  chạy container
     docker compose up -d
  kiểm tra
     docker ps

     
    <img width="875" height="544" alt="Ảnh chụp màn hình 2026-05-25 104627" src="https://github.com/user-attachments/assets/f5ddcb41-c44a-44f8-bf08-6f0a46a9e3b3" />

<img width="853" height="299" alt="Ảnh chụp màn hình 2026-05-25 104647" src="https://github.com/user-attachments/assets/0ee420a2-8b3d-4879-9ded-c18ad8883bf4" />

<img width="1765" height="197" alt="Ảnh chụp màn hình 2026-05-25 105458" src="https://github.com/user-attachments/assets/97e930ee-5c83-4c44-a3fb-e6e0e91dfdcf" />

Cấu hình cloudflare tunnel add router để public wordpress lên sub-domain1 (dùng để truy cập wordpress)

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 203138" src="https://github.com/user-attachments/assets/ee239d67-461f-4472-8822-c47cecece20b" />

Cấu hình cloudflare tunnel add router để public Phpmyadmin lên sub-domain2 (dùng để truy cập phpmyadmin)

<img width="777" height="806" alt="Ảnh chụp màn hình 2026-05-25 203223" src="https://github.com/user-attachments/assets/74300b2f-a62b-4922-9cb2-820142733ec9" />

Cấu hình cloudflare tunnel add router để public n8n này lên sub-domain3 (dùng để truy cập và cấu hình n8n)

<img width="747" height="779" alt="Ảnh chụp màn hình 2026-05-25 203252" src="https://github.com/user-attachments/assets/24b15105-683a-4237-84e3-dedac2331bc1" />

<img width="1810" height="968" alt="Ảnh chụp màn hình 2026-05-25 204919" src="https://github.com/user-attachments/assets/41495ec5-366f-46f0-bb68-043579c9a3dd" />

Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu chưa có bảng nào

https://db.honhat.io.vn

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 211540" src="https://github.com/user-attachments/assets/eaa5f66f-bc5c-4054-9c91-25fbe3db90d7" />

Truy cập sub-domain1 để cài đặt wordpress (làm theo hướng dẫn của wordpress)

  - https://wp.honhat.io.vn

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 210539" src="https://github.com/user-attachments/assets/9c89e158-c092-4fd8-9143-0b0a3bc7e1b0" />

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 210555" src="https://github.com/user-attachments/assets/fdfa43c4-d68e-4aa2-b301-09e859961f37" />

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 210606" src="https://github.com/user-attachments/assets/fedf4f30-2951-4af8-b5f0-1be8ad9501d7" />

Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu có những bảng dữ liệu nào sau khi cài wp

https://db.honhat.io.vn

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 211553" src="https://github.com/user-attachments/assets/ec1fba72-3847-4054-8fed-7ab2c33aace1" />

Tạo 1 bài viết trong wordpress giới thiệu về bản thân sinh viên: thông tin cá nhân, sở thích, ... bài viết có thể chứa hình ảnh, âm thanh, video, ...

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 220652" src="https://github.com/user-attachments/assets/2667fe82-431e-46b4-84ba-12249a40f0cc" />
<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 220657" src="https://github.com/user-attachments/assets/c0b7bcb9-6299-4779-a161-6842d4625bcf" />

Tạo 1 bài viết trong wordpress giới thiệu về nhữn kiến thức mà em đã học được ở môn Phát triển ứng dụng với mã nguồn mở

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 220640" src="https://github.com/user-attachments/assets/5036f1ff-a123-459d-93b8-c1df5e77491d" />

Truy cập sub-domain3 để cấu hình n8n https://n8n.honhat.io.vn

tạo tài khoản admin : nhớ điền đúng email

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 211705" src="https://github.com/user-attachments/assets/69f3045d-154f-4346-bb3c-343a3089cf48" />

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 211828" src="https://github.com/user-attachments/assets/7daa88f0-8e76-4a14-a102-d8c0c54d4141" />

Send me a Licence key, bước này điền đủ thông tin, làm chậm sẽ thấy mục gửi License key về mail (n8n sẽ gửi email KEY cho dùng), check email để lấy KEY

<img width="945" height="2046" alt="image" src="https://github.com/user-attachments/assets/cc0e0f50-765b-4a28-9164-f404282a95cd" />

Activate License key: vào trang chủ => SETTING (góc dưới trái) => Usage and plan => Enter activation key: paste key từ email vào đây => Activate => sẽ nhận đc thông báo (góc dưới phải) Your Registered Community Edition has been successfully activated.

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 220240" src="https://github.com/user-attachments/assets/e3a65df1-e25f-4949-aa9f-ea50d289ca85" />

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 220252" src="https://github.com/user-attachments/assets/b195a903-ca3e-46fb-990a-669dc4bab6ee" />

Create workflow (home page => overview => Create workflow)

Add trigger node: tìm node: Telegram => OnMessage ; cấu hình Credential: Set up Credential => cần Nhập Access Token

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 220931" src="https://github.com/user-attachments/assets/6a1b1c71-c3d0-4421-bcb3-66e569cfe69f" />

Access Token thì lấy ở Telegram qua việc chát với @BotFather

<img width="1125" height="2436" alt="image" src="https://github.com/user-attachments/assets/6e6808de-09b9-43b3-ac7b-e64ca4fe00c5" />

Cần chát với bot @BotFather để đẻ ra bot mới của riêng mình. bot này sẽ là nơi nhận lệnh (promt) để AI sinh html => n8n sẽ dùng html này để đăng bài lên wp

<img width="1125" height="2436" alt="image" src="https://github.com/user-attachments/assets/0f14d27c-516b-4fc0-863c-2ad12bf3f76c" />

Sau khi tạo bot mới cần copy lấy Token, và chát lần đầu với bot mới này, nội dung bất kỳ (bước này quan trọng!)

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 221018" src="https://github.com/user-attachments/assets/8d32ad15-2d9c-4c91-9b91-827ec9177642" />

Add (nối tiếp vào sau node Telegram Trigger) node: AI Google Gemini => Message a model => Set up Credential => cần Nhập API KEY

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 221236" src="https://github.com/user-attachments/assets/7d825cac-293b-4158-a9b1-e44871d021d2" />

Lấy API KEY tại trang: https://aistudio.google.com => https://aistudio.google.com/api-keys

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d4bd5550-2ffa-4cc7-863f-ec9d4e3a7710" />

Nhập API Key lên giao diện n8n

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 221236" src="https://github.com/user-attachments/assets/b3d27027-99f2-48c5-94c3-7523be6ca8d9" />

kéo thả nội dung đã chát với bot của telegram (phía bên trái) vào nội dung phần PROMPT kết quả được {{ $json.message.text }}, cần gõ thêm vào sau {{ $json.message.text }} để promt dài hơn : vd ({{ $json.message.text }}. Kết quả sinh ra ở định dạng HTML+CSS để tôi dùng HTML+CSS này tạo bài viết cho wordpress.)

Turn on Output Content as JSON : để kết quả trả về dạng json

Có thể thử nghiệm các thành phần khác trong Options (add Options: System message, ...) => đưa ra cái nào đáng dùng?

Bấm nút Execute step và chờ Gemini xử lý. Khi thành công, ở cột Output bên phải sẽ thấy AI trả về một chuỗi text thô lộn xộn nhưng chứa cấu trúc post_title và post_content.

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 221320" src="https://github.com/user-attachments/assets/3a176292-43fe-4088-836f-46a3d13bc656" />

Add (nối tiếp vào sau node Message a model) node: Code in JavaScript

Code js ở dạng này, có thể phải thay đổi tuỳ theo json AI trả về.
```text
// 1. lấy dữ liệu gốc
const rawText = $input.first().json.content.parts[0].text;

// 2. Chuyển đổi chuỗi (đã được bọc JSON) thành Object trong JavaScript
const cleanData = JSON.parse(rawText);

// 3. Trả về kết quả định dạng lại gọn gàng cho n8n sử dụng
return {
 title: cleanData.post_title,
 content: cleanData.post_content
};  
```
<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 221407" src="https://github.com/user-attachments/assets/16998af6-6fb6-4060-9350-56e9ed1d473c" />

Add (nối tiếp vào sau node Code in JavaScript) node: WordPress => Create a Post

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 221627" src="https://github.com/user-attachments/assets/a6141717-6d7a-4704-8da3-1d53ee6991cf" />

Set up Credential: vào wp tại url: https://wp.honhat.io.vn/wp-admin => vào mục Tài Khoản => chọn user đã tạo lúc setup wordpress => Mật khẩu ứng dụng => Nhập n8n và bấm "Thêm mật khẩu ứng dụng" => copy chuỗi 24 kí tự : Đây là mật khẩu ứng dụng => paste vào mục Password của n8n Credential

<img width="1658" height="382" alt="image" src="https://github.com/user-attachments/assets/445e7ec4-a1c9-4a0d-9b73-8294c3372374" />

Wordpress URL: điền giá trị https://wp.honhat.io.vn/ (giá trị này cũng khai báo trong biến môi trường WEBHOOK_URL của n8n)

Ignore SSL Issues (Insecure): TURN ON

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 221551" src="https://github.com/user-attachments/assets/5d469093-cd97-4047-87ef-44cdf5dfe9f7" />

Cấu hình node Create a Post: bấm nút Execute previous nodes để thấy trường giá trị của node trước trả về, kéo nội dung phần title (bên trái) vào trường title, tương tự kéo nội dung content vào content

Add field (Thêm thuộc tính): Status == Publish (bài đăng sẽ ở trạng thái xuất bản ngay lập tức, mặc định nó ở giá trị Draft bản nháp)

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 221627" src="https://github.com/user-attachments/assets/1d7d99fe-4da7-46f1-a8d8-fc719bdb6138" />

PUBLISH flow (góc trên phải) Nút này thực hiện việc xuất bản flow <=> flow sẽ tự động thực thi khi thoả mãn điều kiện trigger

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ea54a982-36d8-43ee-8973-e13775d283ff" />

Kết quả cuối cùng cần đặt được:

từ điện thoại, chát với telegram bot

<img width="1125" height="2436" alt="z7865349726560_841b97bb7da6e903eb269d2c14a05f16" src="https://github.com/user-attachments/assets/6a6d61c0-d5ac-4f38-b9b4-ee1448aa7916" />

nội dung chát được tự động gửi tới node Telegram trigger => Gửi tới Google Gemini Message a model (bản chất là gửi Prompt) : Nhận về json kết quả của Prompt => Gửi sang node Code in JavaScript để tách tiêu đề và nội dung => gửi đến node WordPress để Create a Post(đăng bài) với tiêu đề và nội dung từ node trước gửi sang.

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 224837" src="https://github.com/user-attachments/assets/f46fa15b-5aa6-4b05-aaa2-9a3a1ce3f890" />

f5 wordpress để thấy bài viết mới đã lên sóng.

<img width="1920" height="1080" alt="Ảnh chụp màn hình 2026-05-25 222716" src="https://github.com/user-attachments/assets/a8b9548a-608b-49c3-b0c5-38282b21ced5" />

# Nhận xét thành quả đạt được!!!

Sau khi hoàn thành bài tập, em đã triển khai thành công hệ thống WordPress trên Ubuntu bằng Docker Compose với các service gồm MariaDB, PhpMyAdmin, WordPress, Cloudflared và n8n. Các container hoạt động ổn định và có thể truy cập từ Internet thông qua subdomain bằng Cloudflare Tunnel.

Trong quá trình thực hiện, em đã cài đặt và cấu hình WordPress thành công, kiểm tra cơ sở dữ liệu bằng PhpMyAdmin và tạo được các bài viết theo yêu cầu của đề bài. Đồng thời, em cũng cấu hình n8n kết hợp với Telegram Bot và Google Gemini AI để xây dựng hệ thống tự động tạo và đăng bài lên WordPress.

Tuy nhiên, trong quá trình làm bài em cũng gặp một số lỗi như lỗi cấu hình Cloudflare Tunnel, lỗi route DNS bị trùng, lỗi kết nối giữa n8n với Telegram/Gemini và một số lỗi trong quá trình cấu hình workflow. Ngoài ra, có lúc workflow không hoạt động đúng do chưa publish hoặc lỗi credential bị mất kết nối. Sau khi tìm hiểu tài liệu và kiểm tra lại cấu hình từng bước, em đã khắc phục được các lỗi trên và hệ thống hoạt động ổn định.

Kết quả cuối cùng đạt được là chỉ cần gửi nội dung qua Telegram, hệ thống sẽ tự động gửi yêu cầu đến Gemini AI để tạo nội dung bài viết, xử lý dữ liệu bằng JavaScript và đăng trực tiếp lên website WordPress. Qua bài tập này, em hiểu rõ hơn về Docker, Docker Compose, Cloudflare Tunnel, n8n automation cũng như cách tích hợp AI vào một bài toán thực tế.
