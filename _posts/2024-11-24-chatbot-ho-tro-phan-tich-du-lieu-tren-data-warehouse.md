---
title: "Chatbot hỗ trợ phân tích dữ liệu trên Data Warehouse."
layout: post
date: 2024-11-24 09:00:00 +0700
image: /assets/images/blog/bigdata/2024-11-24/intro.png
headerImage: false
tag:
- bigdata
- blockchain
category: blog
author: Long Nguyen
description: Chat bot hỗ trợ phân tích dữ liệu trên Data Warehouse.
---

Trong các bài viết trước, mình đã hướng dẫn các bạn cách xây dựng Data warehouse trên nền tảng Hadoop và một số công nghệ khác. Sử dụng query engine Trino kết hợp với các công cụ trình diễn dữ liệu như Superset hoặc Metabase sẽ giúp chúng ta có thể dễ dàng truy vấn dữ liệu trong Data Warehouse để xây dựng các chart, dashboard phân tích, phục vụ cho hoạt động BI. \\
Tuy nhiên, để mà có thể sử dụng được hệ thống theo cách này, người dùng vẫn cần có kiến thức và kỹ năng nhất định về SQL. Để giải quyết vấn đề này đồng thời hỗ trợ thêm cho các bạn DA trong quá trình truy vấn dữ liệu, mình đã huấn luyện một Chatbot từ ChatGPT và tích hợp vào Data Warehouse. Các bạn có thể dùng thử Chatbot *Sonar* của mình [tại đây](https://sonar.chainslake.io/){:target="_blank"}.

## Nội dung

1. [Hướng dẫn sử dụng](#user_instructions)
2. [Một số lưu ý](#text_note)
3. [Kết luận](#conclusion)

> :pray: *Hiện tại mình đang nhận tư vấn, thiết kế và triển khai hạ tầng phân tích dữ liệu, Data Warehouse, Lakehouse cho các cá nhân, đơn vị có nhu cầu. Bạn có thể xem và dùng thử một hệ thống mình đã build [tại đây](https://metabase.chainslake.io/public/dashboard/ac9dbee4-af29-4ba8-b494-eae69f4ee835){:target="_blank"}. Các bạn vui lòng liên hệ với mình qua email: <hoanglong180695@gmail.com>. Mình xin cảm ơn!*

## Hướng dẫn sử dụng <a name="user_instructions"></a>

*Sonar* là Chat bot trong lĩnh vực crypto, được sử dụng để trả lời các câu hỏi về phân tích dữ liệu blockchain, với một số dạng dữ liệu được cung câp như hình dưới đây:

<p style="
    text-align: center;
"><img src="/assets/images/blog/bigdata/2024-11-24/intro.png" alt="Sonar - Who are you?" width="550"></p>

*Sonar* có thể trả lời được một số câu hỏi từ đơn giản như tra cứu, tổng hợp thông tin trên 1 bảng dữ liệu đến các câu truy vấn phức tạp đòi hỏi cần kết hợp nhiều bảng. Sau đây là một số ví dụ từ đơn giản đến phức tạp để bạn có thể tham khảo:
### Câu hỏi đơn giản

**1. Câu hỏi về giá**

:man: *How much is btc?* \\
:man: *What is the price of eth?*\\
:man: *Price of eth in btc*\\
:man: *Top 10 coins with the highest percentage increase yesterday*

<p style="
    text-align: center;
"><img src="/assets/images/blog/bigdata/2024-11-24/query_1.png" alt="Query price" width="550"></p>

**2. Câu hỏi về khối lượng giao dịch**

:man: *What is the btc trading volume today?*\\
:man: *Top 10 coins with the largest total trading volume today*\\
:man: *Max price and total volume of bnb by hour in the last 48 hours*

<p style="
    text-align: center;
"><img src="/assets/images/blog/bigdata/2024-11-24/query_2.png" alt="Query volume" width="550"></p>

**3. Câu hỏi về số dư trong ví**

:man: *link balance of wallet 0x8652fb672253607c0061677bdcafb77a324de081*\\
:man: *top 10 biggest holders of link*

<p style="
    text-align: center;
"><img src="/assets/images/blog/bigdata/2024-11-24/query_3.png" alt="Query volume" width="550"></p>   

**4. Câu hỏi về hoạt động transfer token**

:man: *Top 10 wallets that received the most btc in the last 48 hours*

<p style="
    text-align: center;
"><img src="/assets/images/blog/bigdata/2024-11-24/query_4.png" alt="Query volume" width="550"></p>

**5. Câu hỏi về các sàn giao dịch tập trung (CEX)**

:man: *List of 10 wallets of okx exchange*\\
:man: *List of 10 deposit wallets of binance exchange*\\
:man: *Top 10 exchanges with the most wallets*\\
:man: *Top 10 exchanges with the most deposit wallets*

<p style="
    text-align: center;
"><img src="/assets/images/blog/bigdata/2024-11-24/query_6.png" alt="Query volume" width="550"></p>

**6. Câu hỏi về các giao dịch phi tập trung (DEX)**

:man: *Top 10 biggest btc trades on dex today*

<p style="
    text-align: center;
"><img src="/assets/images/blog/bigdata/2024-11-24/query_7.png" alt="Query volume" width="550"></p>

### Câu hỏi phức tạp

Câu hỏi phức tạp là câu hỏi cần kết hợp nhiều dữ liệu với nhau, do chưa được huấn luyện nên *Sonar* sẽ không trả lời được hoặc trả lời sai, lúc này bạn sẽ cần bổ sung thêm các định nghĩa (*definition*), hướng dẫn (*hint*) để giúp *Sonar* hiểu được những gì cần phải làm. Ví dụ:

:man: *Top 10 wallets that bought the most btc on dex today.*

Trong những dữ liệu mà *Sonar* đã biết không có dữ liệu nào cho biết thông tin về ví đã mua token trên DEX, vì vậy để *Sonar* trả lời được câu này, ta sẽ cần bổ sung thêm định nghĩa và hướng dẫn như sau:

:man: *Definition: Wallets that buy tokens on DEX are wallets that receive tokens in token purchases on DEX. Request: Get the top 10 wallets that bought the most btc on DEX today. Hint: Get the btc purchases on DEX today and join it with the BTC transfer table today according to transaction hash. Then group by the wallet that receives btc, then calculate the total btc received by each wallet and return the list of receiving wallets in descending order of btc*

<p style="
    text-align: center;
"><img src="/assets/images/blog/bigdata/2024-11-24/query_8.png" alt="Query volume" width="550"></p>

>Bạn có thể tiếp tục hỏi về ví mua token trên DEX của các token khác trong cuộc hội thoại này để tận dụng ngữ cảnh đã thiết lập.

Một số câu hỏi phức tạp khác mà mình đã huấn luyện *Sonar* có thể trả lời trực tiếp mà không cần có hướng dẫn thêm:\\
:man: *Top 10 wallets selling the most btc on dex today*\\
:man: *Top 10 wallets with the most BTC deposits on binance today*\\
:man: *Top 10 wallets withdraw the most eth from coinbase today.*

<p style="
    text-align: center;
"><img src="/assets/images/blog/bigdata/2024-11-24/query_9.png" alt="Query volume" width="550"></p>

## 

## Một số lưu ý khi sử dụng <a name="text_note"></a>

- Trong câu hỏi cần có giới hạn về thời gian (VD: today, yesterday, in the last 48 hours...) để giảm bớt lượng data cần truy vấn
- Cần có giới hạn về số lượng kết quả trả về (VD: Top 10 wallets, top 10 coins)

## Kết luận <a name="conclusion"></a>

Trong bài viết này mình đã giới thiệu với các bạn một ví dụ về tích hợp Chatbot hỗ trợ cho hoạt động BI trên Data warehouse. Hẹn gặp lại các bạn trong các bài viết tiếp theo.


