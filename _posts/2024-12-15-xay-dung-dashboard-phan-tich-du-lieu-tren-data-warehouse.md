---
title: "Xây dựng Dashboard phân tích dữ liệu trên Data Warehouse."
layout: post
date: 2024-12-15 09:00:00 +0700
image: /assets/images/blog/bigdata/2024-12-15/demo-dashboard.png
headerImage: false
tag:
- bigdata
- blockchain
category: blog
author: Long Nguyen
description: Bài viết này sẽ giới thiệu cách sử dụng công cụ Metabase để xây dựng các dashboard phân tích dựa trên dữ liệu từ Data Warehouse để phục vụ cho hoạt động BI
---

Tiếp tục series các bài viết chủ đề Data Warehouse, hôm nay chúng ta sẽ đến với một nội dung khá là thú vị đó là xây dựng Dashboard phân tích dữ liệu trên Data warehouse. Xây dựng Dashboard phân tích dữ liệu là một trong những chức năng cơ bản nhất trong các hoạt động BI (Business Intelligence), giúp trực quan hóa dữ liệu dưới dạng các biểu đồ, từ đó giúp cho người dùng có thể dễ dàng hiểu và nhìn thấy các insight từ dữ liệu. \\
Để minh họa cho bài viết thì mình sẽ sử dụng [Chainslake](https://metabase.chainslake.io){:target="_blank"}, một blockchain data warehouse do mình phát triển, cho phép người dùng truy vấn dữ liệu blockchain, xây dựng và chia sẻ các dashboard phân tích dữ liệu blockchain hoàn toàn miễn phí.

## Nội dung
1. [Giới thiệu tổng quan về Metabase](#introduction) 
2. [Cài đặt và cấu hình Metabase kết nối vào Data warehouse](#install-and-config)
3. [Các bảng dữ liệu trong data warehouse](#tables)
4. [Truy vấn dữ liệu, xây dựng và chia sẻ dashboard](#query-build-share)
5. [Kết luận](#conclusion)

> :pray: *Hiện tại mình đang nhận tư vấn, thiết kế và triển khai hạ tầng phân tích dữ liệu, Data Warehouse, Lakehouse cho các cá nhân, đơn vị có nhu cầu. Bạn có thể xem và dùng thử một hệ thống mình đã build [tại đây](https://metabase.chainslake.io/public/dashboard/ac9dbee4-af29-4ba8-b494-eae69f4ee835){:target="_blank"}. Các bạn vui lòng liên hệ với mình qua email: <hoanglong180695@gmail.com>. Mình xin cảm ơn!*

## Giới thiệu tổng quan về Metabase <a name="introduction"></a>

[Metabase](https://www.metabase.com/) là một công cụ BI cho phép truy vấn dữ liệu bằng ngôn ngữ SQL trên nhiều cơ sở dữ liệu và các SQL engine khác nhau thông qua các [plugins](https://www.metabase.com/docs/latest/databases/connecting), trình diễn kết quả truy vấn thành các bảng, biểu đồ, số liệu trên các dashboard phân tích, bạn có thể xem một dashboard demo của mình [tại đây](https://metabase.chainslake.io/public/dashboard/ac9dbee4-af29-4ba8-b494-eae69f4ee835){:target="_blank"}.

Metabase có cả phiên bản Opensource với các chức năng cơ bản, có thể cài đặt trên hạ tầng có sẵn và bản Enterprise với các chức năng nâng cao có thể sử dụng trực tiếp trên cloud của Metabase hoặc tự host thông qua key bản quyền. Bạn có thể xem chi tiết [tại đây](https://www.metabase.com/pricing/){:target="_blank"}.

## Cài đặt và cấu hình Metabase kết nối vào Data warehouse <a name="install-and-config"></a>

Mình sẽ cài đặt phiên bản Opensource của Metabase qua docker và kết nối vào cụm DWH của chúng ta thông qua Trino (các bạn có thể xem lại bài viết về cách cài đặt Trino của mình [tại đây](/cai-dat-trino-truy-van-du-lieu-trong-data-warehouse))

Đầu tiên mình sẽ tạo một cơ sở dữ liệu cho Metabase trong postgres trên cụm [DWH](/cai-dat-data-warehouse-tren-hadoop-phan-1/#install_postgresql)


```sql
postgres=# CREATE DATABASE metabase;
```

Cài đặt metabase thông qua Docker

```sh
$ wget https://github.com/starburstdata/metabase-driver/releases/download/5.0.0/starburst-5.0.0.metabase-driver.jar
$ docker run -d -p 3000:3000 --name metabase \
  --network hadoop --add-host=node01:172.20.0.2 \
  -e "MB_DB_TYPE=postgres" \
  -e "MB_DB_DBNAME=metabase" \
  -e "MB_DB_PORT=5432" \
  -e "MB_DB_USER=postgres" \
  -e "MB_DB_PASS=password" \
  -e "MB_DB_HOST=node01" \
  -v starburst-5.0.0.metabase-driver.jar:/plugins/starburst-5.0.0.metabase-driver.jar \
   --name metabase metabase/metabase
```

> Lưu ý `172.20.0.2` là ip container của node01 trên máy của mình, bạn thay bằng ip máy của bạn.

Truy cập vào `http://localhost:3000` bạn sẽ thấy giao diện khởi động của Metabase, sau khi tạo tài khoản admin bạn sẽ được đưa tới giao diện làm việc chính của Metabase.

![Metabase intro](/assets/images/blog/bigdata/2024-12-15/metabase-intro.png)

Để kết nối với Trino bạn vào `Admin settings` chọn thẻ `Databases`, chọn `Add database` và thiết lập như hình dưới đây:

![Connect to Trino](/assets/images/blog/bigdata/2024-12-15/connect-trino.png)

Sau khi cấu hình xong bạn `Exit admin` và vào kiểm tra xem đã có dữ liệu trong mục `Browser/Databases` hay chưa. Bạn có thể xem các bảng dữ liệu của Chainslake [tại đây](https://metabase.chainslake.io/browse/databases/3-chainslake){:target="_blank"}

![Chainslake data](/assets/images/blog/bigdata/2024-12-15/chainslake-data.png)

## Các bảng dữ liệu trong data warehouse <a name="tables"></a>

Trong phần này mình sẽ mô tả sơ lược về các bảng dữ liệu trong data warehouse của [Chainslake](https://metabase.chainslake.io/browse/databases/3-chainslake){:target="_blank"} để có cơ sở cho việc viết truy vấn và build dashboard trong phần tiếp theo.

Data warehouse của Chainslake bao gồm nhiều thư mục, mỗi thư mục gồm nhiều bảng có cùng chủ đề với nhau, cụ thể mình sẽ giới thiệu một số bảng và thư mục sau:

- __ethereum__: Thư mục chứa dữ liệu raw của Ethereum (Bạn có thể tìm hiểu thêm về các loại dữ liệu của Ethreum blockchain trong bài viết trước của mình [tại đây](/he-thong-phan-tich-du-lieu-blockchain-phan-2/))
    - __transactions__: Chứa toàn bộ dữ liệu giao dịch trên Ethereum (từ khi bắt đầu đến hiện tại)
    - __logs__: Chứa toán bộ dữ liệu log (event) phát ra từ các contract khi chúng thực thi giao dịch.
    - __traces__: Chứa toàn bộ dữ liệu Internal Transactions (Là các đơn vị giao dịch nhỏ nhất trên Ethereum).
    - __address_type__: Chứa toàn bộ các địa chỉ từng xuất hiện trên Ethereum bao gồm 2 loại là `wallet` và `contract`
- __ethereum_decoded__: Thư mục chứa dữ liệu decode của một số protocol trên Ethereum. Hiện tại Chainslake đang có dữ liệu decode của một số protocol phổ biến sau:
    - __erc20_evt_transfer__: Dữ liệu chuyển token ERC20 (toàn bộ token từ trước đến nay)
    - __erc721_evt_transfer__: Dữ liệu chuyển NFT ERC721 
    - __erc1155_evt_transferbatch__, __erc1155_evt_transfer_single__: Dữ liệu chuyển NFT 1155
    - __uniswap_v2_evt_swap__, __uniswap_v3_evt_swap__: Dữ liệu swap token trên các pool thanh khoản.
- __ethereum_contract__: Chứa toàn bộ thông tin về contract của 1 số protocol phổ biến
    - __erc20_tokens__: Thông tin về các token ERC20.
    - __erc721_tokens__: Thông tin về các NFT ERC721.
    - __erc1155_tokens__: Thông tin về các NFT ERC1155.
    - __uniswap_v2_info__, __uniswap_v3_info__: Thông tin về các pool contract của Swap (Uniswap và các protocol khác cùng standard)
- __ethereum_dex__: Chứa dữ liệu giao dịch mua bán của các token ERC20 trên các sàn phi tập trung DEX.
    - __token_trades__: Chứa dữ liệu giao dịch mua bán của các token ERC20 trên các sàn phi tập trung DEX với WETH.
- __ethereum_prices__: Chứa dữ liệu về giá của token theo các giao dịch lấy từ dữ liệu DEX.
    - __erc20_usd_day__, __erc20_usd_hour__, __erc20_usd_minute__: Giá token ERC20 theo usd theo ngày, giờ, phút.
- __ethereum_balances__: Thư mục chứa thông tin về số dư, biến động số dư của các địa chỉ trên Ethereum
    - __erc20_native__: Bảng tổng hợp số dư cuối cùng và biến động số dư mỗi ngày của toàn bộ địa chỉ của toàn bộ token bao gồm cả Native token trên Ethereum.
- __binance_cex__: Bảng chứa dữ liệu giá giao dịch của các coin trên sàn giao dịch Binance
    - __coin_token_address__: Danh sách cá coin đang niêm yết trên sàn Binance mà có token tương ứng trên Ethereum
    - __trade_minute__: Dữ liệu giá của các cặp giao dịch theo phút trên Binance.

## Truy vấn dữ liệu, xây dựng và chia sẻ dashboard <a name="query-build-share"></a>

Việc viết truy vấn và build dashboard trên Chainslake là khá đơn giản với nếu như bạn đã có kỹ năng về SQL. Mình có tạo sẵn một dashboard trong collection [Demo](https://metabase.chainslake.io/collection/61-demo){:target="_blank"} để bạn có thể hiểu và dễ dàng bắt đầu tạo ra các dashboard phân tích của riêng mình. 

![Demo collection](/assets/images/blog/bigdata/2024-12-15/demo-collection.png)

![Demo dashboard](/assets/images/blog/bigdata/2024-12-15/demo-dashboard.png)

Một số lưu ý khi viết truy vấn:
- Các bảng dữ liệu đều có số lượng bản ghi cực kỳ lớn (do chứa toàn bộ dữ liệu lịch sử) do đó tất cả các bảng mình đều partition theo `block_date`, bạn nên sử dụng column này để lọc hoặc join giữa các bảng bất kỳ khi nào có thể sẽ giúp câu truy vấn của bạn chạy nhanh và hiệu quả hơn. 
- Sử dụng các kỹ thuật về tối ưu câu truy vấn, giảm lượng dữ liệu cần phải scan khi thực thi truy vấn luôn cần phải được xem xét cẩn thận để đảm bảo câu truy ván có thể thực thi được trong thời gian cho phép, đồng thời điều này cũng giúp bạn cải thiện khả năng viết và tối ưu truy vấn.
- Kết quả truy vấn sẽ được tự động cache trong 1 ngày, bạn có thể thay đổi trong cấu hình của mỗi truy vấn.
- Cuối cùng là đừng quên chia sẻ các dashboard của bạn cho mọi người để Chainslake có thêm nhiều user hơn các bạn nhé.

## Kết luận <a name="conclusion"></a>

Trên đây là những chia sẻ của mình về cách xây dựng dashboard phân tích dữ liệu trên data warehouse, rất mong nhận được sự ủng hộ của tất cả các bạn. Hẹn gặp lại các bạn trong các bài viết tiếp theo.