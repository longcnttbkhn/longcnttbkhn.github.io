---
title: "Hướng dẫn cơ bản về Bitcoin"
layout: post
date: 2022-11-01 23:13:28 +0700
image: /assets/images/blog/blockchain/2022-11-01/bitcoin-logo-sm.png
headerImage: false
tag:
- blockchain
category: blog
author: Long Nguyen
description: Bitcoin là đồng tiền điện tử đầu tiên và cũng là quan trọng nhất trong thế giới Crypto. Trong bài viết này mình sẽ giới thiệu những kiến thức cơ bản nhất về Bitcoin và một số hướng dẫn ban đầu khi sử dụng nó.
---

## Nội dung
1. [Giới thiệu tổng quan](#introduction)
2. [Cài đặt và thử nghiệm với Bitcoin](#experience)
3. [Một số điều cần lưu ý khi sử dụng Bitcoin](#note)
4. [Kết luận](#conclusion)

## Giới thiệu tổng quan <a name="introduction"></a>

Bitcoin được giới thiệu lần đầu trong bài báo cùng tên của tác giả ẩn danh Satoshi Nakamoto, bạn có thể xem đầy đủ [tại đây][bitcoin-paper]. Mình sẽ tóm tắt lại các nội dung chính như sau:

*Bitcoin hoạt động theo cơ chế ngang hàng (peer-to-peer)*. Trong hệ thống tiền tệ truyền thống, để thực hiện được một giao dịch ta luôn cần một bên thứ 3 như ngân hàng, nhà nước hoặc một tổ chức mà cả 2 bên tham gia đều tin tưởng đứng ra làm trung gian ghi lại lịch sử giao dịch và cập nhật số dư cho cả 2 bên. Trong khi đó Bitcoin vẫn có thể thực hiện giao dịch mà không cần một bên thứ 3 nào khác, 2 bên tham gia cũng không cần tin tưởng nhau, đây được gọi là cơ chế ngang hàng.

*Sổ cái phi tập trung*. Tất cả lịch sử giao dịch được lưu trong 1 cuấn sổ cái, trong hệ thống tập trung thì chỉ có bên thứ 3 mới được giữ và có quyền thêm giao dịch vào sổ cái. Còn với Bitcoin, mỗi bên tham gia sẽ giữ một bản sao đầy đủ của sổ cái, điều này đảm bảo việc không ai có thể thay đổi thông tin trong sổ cái nếu không có sự đồng thuận của tất cả các bên.

*Cơ chế đồng thuận*. Để thêm một giao dịch vào sổ cái phi tập trung thì giao dịch đó phải hợp lệ tức là không thể chi tiêu số tiền lớn hơn số dư, nếu điều này xảy ra các bên khác sẽ từ chối ghi lại giao dịch vào sổ cái của họ và giao dịch sẽ thất bại. Vấn đề ở đây là để biết một giao dịch có hợp lệ hay không ta cần biết tất cả các giao dịch đã xảy ra trước nó, tức là các giao dịch phải có thứ tự. Trong hệ thống tập trung ta có thể đảm bảo thứ tự của giao dịch bằng cách sử dụng hàng đợi, tuy nhiên trong hệ thống phi tập trung, nơi mà có rất nhiều bên đều muốn thêm giao dịch vào sổ cái, chúng ta cần một cơ chế đồng thuận.

*Bằng chứng công việc (Proof-of-Work)*. Bitcoin sử dụng cơ chế đồng thuận bằng chứng công việc, theo đó tất cả các bên sẽ cùng giải một bài toán khó, các dữ liệu đầu vào của bài toán chỉ có khi giao dịch cuối cùng đã được thêm vào sổ cái, điều này đảm bảo không ai có thể bắt đầu tìm lời giải cho bài toán trước những người khác. Bên nào tìm ra lời giải trước sẽ được quyền thêm giao dịch vào sổ cái.

*Blockchain:* Thực tế trong mạng lưới Bitcoin sẽ có những bên sở hữu khả năng tính toán mạnh hơn những người khác nên có ưu thế lớn trong việc tìm ra lời giải cho bài toán. Thay vì tự mình thêm giao dịch vào sổ cái những người khác có thể gửi giao dịch đến cho các node tính toán mạnh, những node này sẽ tập hợp các giao dịch thành một khối và thêm vào sổ cái lúc này là một chuỗi các khối (blockchain).

*Thợ đào:* Các node làm nhiệm vụ thêm block mới vào blockchain được gọi là thợ đào. Mỗi khi thêm được một block mới, thợ đào sẽ nhận được phần thưởng là một số lượng bitcoin nhất định, họ cũng sẽ nhận được phí giao dịch từ mỗi giao dịch được gửi đến.

*Cơ chế xác thực:* Để gửi và nhận bitcoin, người dùng cần có một cặp <địa chỉ, khoá bí mật>. Để nhận bitcoin thì chỉ cần đưa địa chỉ cho người gửi còn nếu muốn gửi bitcoin cho người khác thì người dùng cần tạo một giao dịch với địa chỉ của người nhận, số bitcoin muốn chuyển đi và một chữ ký số được tạo ra bằng cách sử dụng khoá bí mật ký lên giao dịch. Thợ đào khi nhận được giao dịch sẽ kiểm tra giao dịch và chữ ký số để xác thực thông tin trước khi giao dịch được thêm vào blockchain.

## Cài đặt và thử nghiệm với Bitcoin <a name="experience"></a>

Lý thuyết vậy thôi giờ chúng ta bắt đầu vào thực hành nhé! Sử dụng Bitcoin thì có nhiều cách nhưng ở đây mình sẽ sử dụng [Bitcoin Core][bitcoin-core], đây là phần mềm được tạo ra bởi chính cha đẻ của Bitcoin nên có thể xem như là chính thống. Bitcoin core có đầy đủ tất cả chức năng phục vụ cho mạng lưới Bitcoin cũng như cho mỗi người dùng, do đó rất thích hợp cho việc nghiên cứu, thử nghiệm với Bitcoin.

Bạn download về và cài đặt như các phần mềm bình thường khác, lưu ý chọn đúng hệ điều hành đang sử dụng. Mặc định khi khởi chạy Bitcoin core sẽ kết nối đến các nút dữ liệu khác trong mạng lưới và download toàn bộ lịch sử giao dịch, quá trình này sẽ tốn khoảng 150GB ổ cứng và nhiều ngày chờ đợi. Do chỉ phục vụ mục đích tìm hiểu, nghiên cứu nên mình sẽ thay đổi sang chế độ test bằng cách thêm cấu hình `chain=regtest` vào file `bitcoin.conf` trong thư mục:

* Trên Windows: `%APPDATA%\Bitcoin\`
* Trên Macos: `$HOME/Library/Application Support/Bitcoin/`
* Trên Linux: `$HOME/.bitcoin/`

### Tạo wallet

Sau khi cấu hình xong bạn khởi chạy Bitcoin core lên sẽ có giao diện như sau:
![Create wallet](/assets/images/blog/blockchain/2022-11-01/create-wallet.png)

Sau khi tạo ví
![Overview](/assets/images/blog/blockchain/2022-11-01/overview.png)

> Ví là nơi lưu trữ thông tin địa chỉ và khoá bí mật tương ứng, ví thực hiện công việc tạo, ký và xác thực các giao dịch, tổng hợp số dư của tất cả địa chỉ trong ví từ đó giúp cho việc gửi và nhận của người dùng trở nên dễ dàng hơn.

Bạn mở giao diện *console* bằng cách vào Window chọn Console:

![Console](/assets/images/blog/blockchain/2022-11-01/console.png)

Trên console chúng ta có thể sử dụng một số lệnh hay dùng sau:

* help: Hiển thị danh sách tất cả các lệnh
* getbalance: Hiển thị số dư của ví hiện tại
* getnewaddress: Tạo một địa chỉ mới trong ví. Có thể tạo rất nhiều địa chỉ trong ví, thông thường mỗi lần nhận tiền sẽ đều tạo một địa chỉ mới.
* listaddressgroupings: Hiển thị danh sách các địa chỉ có tiền trong ví

### Mine block

Bây giờ ta sẽ mine block để lấy tiền thưởng. Ở chế độ `regtest` từ block 101 thì mỗi block mine ra sẽ được nhận 50 BTC tiền thưởng, trên console ta sử dụng lệnh sau:

```sh
generatetoaddress 101 bcrt1q0z5ldgufpvqzmk7qmcq2p75zjdrn8u45yxa0an
```

> Lưu ý: `bcrt1q0z5ldgufpvqzmk7qmcq2p75zjdrn8u45yxa0an` là địa chỉ mình tạo bằng lệnh `getnewaddress`, bạn tự thay bằng địa chỉ mà bạn tạo ra.

Sau khi mine xong thì trên ví sẽ có 50 BTC (tất nhiên đây chỉ là BTC test thôi)

![Console](/assets/images/blog/blockchain/2022-11-01/balance.png)

> Trên console bạn sử dụng các lệnh `getbalance` và `listaddressgroupings` để kiểm tra số dư và địa chỉ trong ví. 

### Gửi và nhận tiền

Trước hết bạn cần tạo một wallet mới bằng cách vào File chọn Create wallet... Tiếp theo vào tab Receive để tạo một đỉa chỉ mới cho việc nhận tiền:

![Receive](/assets/images/blog/blockchain/2022-11-01/receive.png)

Chọn Copy address sau đó switch sang account ban đầu, chọn tab Sent sau đó điền địa chỉ vào ô Pay To, số tiền vào Amount, chọn Transaction Fee là Custom sau đó click `send` để gửi.

![Send](/assets/images/blog/blockchain/2022-11-01/send.png)

Sau khi Send bạn sẽ thấy giao dịch đã được tạo nhưng chưa được confirm. Bây giờ ta cần mine block mới chứa transaction này, bạn vào console và gõ lệnh sau:

```sh
generateblock bcrt1q0z5ldgufpvqzmk7qmcq2p75zjdrn8u45yxa0an '["3204b38460db25d788577ef4b5051cb7d4284ff0bf56bcca084888fce6aa0adf"]'
```

> Trong đó: `bcrt1q0z5ldgufpvqzmk7qmcq2p75zjdrn8u45yxa0an` là địa chỉ nhận thưởng khi mine block, `3204b38460db25d788577ef4b5051cb7d4284ff0bf56bcca084888fce6aa0adf` là Transaction ID của giao dịch. 

Sau khi mine block ta kiểm tra lại trên ví nhận thì đã thấy tiền được chuyển sang:

![Send success](/assets/images/blog/blockchain/2022-11-01/send_success.png)

## Một số điều cần lưu ý khi sử dụng Bitcoin <a name="note"></a>

### Thường xuyên backup ví

Khi sử dụng Bitcoin, việc backup ví là **cực kỳ quan trọng**, nếu vì một lý do nào đó (hỏng, mất ổ cứng) làm mất dữ liệu của ví thì bạn sẽ **không thể** chuyển tiền ra khỏi ví được, đồng nghĩa với mất toàn bộ số tiền trong ví. Nên thực hiện việc backup ví sau mỗi lần giao dịch, file backup nên được lưu trên các thiết bị tin cậy, có thể sử dụng ví cứng [Ledger][ledger].

Để backup ví trên Bitcoin core bạn vào File chọn Backup Wallet...

### Không sử dụng lại địa chỉ

Do toàn bộ giao dịch của Bitcoin được công khai trên blockchain nên để tránh bị theo dõi hành vi bạn nên tạo địa chỉ mới cho mỗi lần nhận tiền (Khi gửi tiền, ví sẽ tự động tạo một địa chỉ mới và gửi số tiền còn lại vào đó).

### Đào bitcoin

Đào bitcoin là quá trình các thợ đào cạnh tranh nhau để giành quyền thêm block mới vào blockchain. Thợ đào sẽ tìm một số gọi là `nonce` thoả mãn điều kiện bài toán bằng cách thử rất nhiều số khác nhau cho đến khi tìm được `nonce`. Thợ đào có năng lực tính toán mạnh hơn hoặc là may mắn hơn tìm được số `nonce` trước sẽ được quyền thêm block mới vào blockchain và nhận được phần thưởng, bạn có thể xem thêm về cách hoạt động của Bitcoin trong demo [này][demo-blockchain]. Hiện nay việc đào Bitcoin thược được thực hiện bằng những phần mềm và phần cứng chuyên dụng như máy ASIC cho phép việc đào bitcoin hiệu quả hơn.

Thuật toán bitcoin sẽ tự động điều chỉnh để tăng độ khó của bài toán khi năng lực tính toán của các thợ đào tăng lên, đảm bảo rằng mỗi 10' sẽ có 1 block mới được thêm vào blockchain. Phần thưởng cho mỗi lần thêm block mới của thợ đào sẽ giảm đi 1 nửa sau mỗi 210 ngàn block (khoảng 4 năm). Lúc đầu phần thưởng là 50 BTC cho mỗi block, hiện nay là 12.5 BTC, dự kiến đến năm 2140 sẽ đào hết 21 triệu đồng BTC.

### Ví Bitcoin

Đa số người dùng Bitcoin chỉ cần sử dụng các chức năng gửi và nhận tiền do đó họ không nhất thiết phải download toàn bộ lịch sử giao dịch của Bitcoin, thay ví thế chỉ cần sử dụng ví để lưu trữ thông tin địa chỉ và khoá bí mật. Hiện nay có rất nhiều loại ví có thể sử dụng với Bitcoin, có thể được phân thành 2 loại như sau:
* Ví nóng: là loại ví có kết nối Internet bao gồm 2 loại:
    * Ví Online: địa chỉ, khoá bí mật được quản lý bởi nhà cung cấp dịch vụ ví, người dùng cần đăng ký tài khoản để sử dụng, ví dụ như ví của các sàn giao dịch: Binance, Coinbase...
    * Ví Offline: là phần mềm cài trên máy tính hoặc điện thoại, thông tin địa chỉ và khoá bí mật cũng được lưu trữ trên thiết bị của người dùng, ví dụ: Trust Wallet...
    
    Ưu điểm của ví nóng là đơn giản, dễ sử dụng, chi phí thấp tuy nhiên nhược điểm là bảo mật không cao, nếu sử dụng ví Online còn có nguy cơ không rút được tiền nếu bị nhà cung cấp khoá tài khoản.
* Ví lạnh: Là loại ví không kết nối Internet, có thể là một chiếc USB lưu trữ thông tin hoặc thậm chí người dùng có thể in thông tin ví ra giấy và cất trữ ở một nơi an toàn. Ưu điểm của ví lạnh là bảo mật tốt do không có kết nối Internet, tuy nhiên nhược điểm là khó sử dụng hơn.

### Bitcoin Explorer

Bitcoin Explorer là trang web cho phép tìm kiếm tra cứu thông tin các giao dịch, kiểm tra số dư của bất kỳ địa chỉ nào trên mạng bitcoin. Thông thường mọi người sẽ sử dụng ví kết hợp với Bitcoin Explorer thay vì phải lưu trữ toàn bộ lịch sử bitcoin trên máy. VD: [btc.com](https://explorer.btc.com/btc), [blockchair](https://blockchair.com/bitcoin), [Blockcypher](https://live.blockcypher.com/)

## Kết luận  <a name="conclusion"></a>

Qua bài viết này, mình đã giới thiệu các kiến thức cơ bản nhất về Bitcoin và một số hướng dẫn ban đầu với nó, hi vọng sẽ giúp ích ít nhiều cho bạn trong quá trình sử dụng. Hẹn gặp lại trong các bài viết sau. 

[bitcoin-paper]: https://bitcoin.org/bitcoin.pdf
[bitcoin-core]: https://bitcoin.org/en/download
[ledger]: https://www.ledger.com/
[trust-wallet]: https://trustwallet.com/vi/bitcoin-wallet/
[demo-blockchain]: https://andersbrownworth.com/blockchain/
