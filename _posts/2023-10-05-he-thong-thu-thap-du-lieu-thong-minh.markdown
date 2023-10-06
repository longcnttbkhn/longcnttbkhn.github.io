---
title: "Hệ thống thu thập dữ liệu thông minh - AI Power Crawler"
layout: post
date: 2023-10-05 22:00:00 +0700
image: /assets/images/blog/bigdata/2023-10-05/ai-crawler.jpeg
headerImage: false
tag:
- bigdata
- machine_learning
category: blog
author: Long Nguyen
description: Bài viết giới thiệu, trình bày các ý tưởng và trải niệm của tác giả trong quá trình xây dựng AI Power Crawler - Hệ thống thu thập dữ liệu thông minh.
---

Chào mọi người, mình đã trở lại rồi đây! Mình rất cảm ơn mọi người thời gian qua đã ủng hộ blog của mình, những góp ý và thắc mắc của các bạn chính là nguồn động lực to lớn đối với mình. Bài viết hôm nay khá là đặc biệt, mình sẽ giới thiệu cho mọi người về một dự án mà mình rất tâm huyết: AI Power Crawler (APC) - Hệ thống thu thập dữ liệu thông minh, hi vọng mọi người sẽ cảm thấy thích nó. APC được thiết kế để crawl dữ liệu từ nhiều website trong một lĩnh vực cụ thể, có khả năng tự động nhận diện các trang có dữ liệu mục tiêu, tự động trích xuất thông tin cần thiết trên trang. Trong giới hạn bài viết này mình sẽ trình bày tóm tắt những ý tưởng chính, thiết kế và kinh nghiệm khi triển khai dự án, nếu bạn quan tâm và muốn tìm hiểu sâu hơn thì có thể tham khảo luận văn của mình [tại đây](/assets/docs/master_thesis.pdf)

## Nội dung
1. [Giới thiệu tổng quan](#introduction)
2. [Phân tích giải pháp](#solution_analyst)
3. [Kiến trúc thiết kế](#system_architecture)
4. [Một số kinh nghiệm rút ra](#experience)
5. [Kết luận](#conclusion)

## Giới thiệu tổng quan <a name="introduction"></a>

Ý tưởng cho dự án này xuất hiện khi mình cần maintain một con Crawler dựa trên [Scrapy](https://scrapy.org/), thu thập dữ liệu từ các trang tin rao bất động sản, sử dụng Css selector và xpath parser để trích rút các thông tin từ tin rao như địa chỉ, giá, diện tích, số phòng... Hệ thống đang có gặp một số vấn đề như sau:

- Khó mở rộng trên nhiều website do phải làm parser cho từng website, trong nhiều trường hợp việc này khá khó khăn.
- Bộ parser gặp lỗi khi website thay đổi giao diện
- Khi cần trích xuất thêm thông tin sẽ phải cập nhật parser cho tất cả các website

APC được xây dựng dựa trên hệ thống đang có và tích hợp thêm các mô hình học máy để có thêm các chức năng thông minh như:
- Phân loại để nhận biết các trang có dữ liệu mục tiêu trước cả khi load trang (phân loại theo URL)
- Bộ Parser trích rút thông tin không cần sử dụng selector hay xpath, không phụ thuộc vào cấu trúc của trang.

Dự án đã được mình triển khai trong 2 sản phẩm ở 2 công ty khác nhau, một ở Cenhomes.vn với bài toán thu thập dữ liệu từ nhiều website bất động sản, một ở Darenft với bài toán tổng hợp so sánh giá listing của 1 NFT trên nhiều chợ. Mỗi bài toán có những yêu cầu cụ thể khác nhau tuy nhiên vẫn sử dụng chung về ý tưởng, giải pháp, thiết kế cùng một số công nghệ opensource như [Fasttext](https://fasttext.cc/), [Web2text](https://github.com/dalab/web2text), [Fonduer](https://github.com/HazyResearch/fonduer).

## Phân tích giải pháp <a name="solution_analyst"></a>

### Nhận biết trang mục tiêu

Trong hệ thống crawler bằng Scrapy, crawler sẽ sử dụng các luật được định nghĩa bởi lập trình viên để xác định một URL có có chứa dữ liệu mục tiêu hay không, các luật này sẽ được fix cứng trong code cho từng website (gọi là Spider). APC sử dụng 2 mô hình học máy để xác định trang mục tiêu:

- Content Classification Model: Mô hình phân loại trang dựa trên nội dung, có thể thực hiện việc phân loại trên nhiều website khác nhau.
    - Input: Title, Description, Main text trong trang
    - Ouput: Kết quả phân loại trang
    - Cách train mô hình: Dữ liệu dùng để train được thu thập từ nhiều website khác nhau, mỗi trang chỉ lấy title, description và main text. Dữ liệu được gán nhãn bằng cách sử dụng các luật phân loại có thể áp dụng trên từng website, kết quả sẽ được một bộ dữ liệu có gán nhãn để train mô hình. Mình sử dụng thư viện [Fasttext](https://fasttext.cc/) để train mô hình này, độ chính xác F1 score cần đạt trên 90%.
    - Trường hợp sử dụng: Mô hình phân loại trang dựa trên nội dung tuy có thể thực hiện việc phân loại trên nhiều website khác nhau nhưng có nhược điểm là phải load trang về mới có thể phân loại được, do đó mô hình này được sử dụng để tạo dữ liệu huấn luyện cho mô hình phân loại URL.
- URL Classification Model: Mô hình phân loại trang dựa trên URL, được train riêng cho từng website
    - Input: URL của trang
    - Output: Kết quả phân loại trang
    - Cách train mô hình: Dự liệu train lấy từ các trang 1 website, sử dụng mô hình phân loại dựa trên nội dung để phân loại và gán nhãn cho các trang sau đó sử dụng kết quả gán nhãn để train mô hình phân loại URL. Đối với mô hình này thì tùy thuộc vào từng trường hợp cụ thể để lựa chọn giải pháp phù hợp. Đối với bài toán ở Cenhomes.vn mình tiếp tục sử dụng Fasttext để train mô hình, còn với bài toán DareNFT mình đã viết một thuật toán để sinh ra pattern url cho website.
    - Trường hợp sử dụng: Mô hình phân loại trang dựa trên URL được sử dụng riêng để phân loại URL cho từng website, APC sử dụng nó để biết trước URL nào chứa dữ liệu mục tiêu từ đó định hướng việc thu thập.

### Trích rút thông tin từ trang

Bộ parser truyền thống trong Scrapy yêu cầu lập trình viên phải định nghĩa luật bóc tách dữ liệu cho từng website, điều này khiến việc mở rộng thu thập dữ liệu trên nhiều website trở nên khó khăn. Để trích xuất thông tin, APC sử dụng 2 mô hình học máy:

- Main content Detection Model: Mô hình phát hiện nội dung chính trong trang. Trong 1 trang thường có nhiều thành phần như header, footer, slidebar và main, trong đó chỉ có phần main là chứa nội dung có ích cho việc trích xuất thông tin, thêm vào đó việc trích rút thông tin từ phần main thay vì toàn bộ trang sẽ giúp giảm thiểu nhầm lẫn thông qua việc giảm bớt dữ liệu nhiễu. Mô hình xác định nội dung chính có thể hoạt động trên nhiều website khác nhau.
    - Input: HTML của toàn bộ trang
    - Output: Main html và main text của trang
    - Cách train mô hình: Dữ liệu train lấy từ nhiều website khác nhau, với mỗi website sử dụng css selector hoặc xpath để đánh dấu phần main (gán nhãn dữ liệu) sau đó mình sử dụng [Web2text](https://github.com/dalab/web2text) để train mô hình với độ chính xác từ 75-80%
    - Trường hợp sử dụng: Mô hình xác đinh main content có thể hoạt động trên nhiều website khác nhau tuy nhiên nó có một vấn đề là chạy rất chậm do đó mình ko sử dụng trực tiếp mô hình này khi crawl dữ liệu, thay vào đó mình sử dụng nó để phát hiện main content cho từng website dưới dạng xpath hoặc css selector rồi sử dụng nó lấy main khi crawl dữ liệu.

- Information Extraction Model: Mô hình này cho phép trích rút các thông tin theo yêu cầu bài toán. Ví dụ: địa chỉ, giá, diện tích... trong bài toán thu thập dữ liệu bất động sản
    - Input: Main html của trang
    - Ouput: các thông tin trích xuất được theo format Json
    - Cách train mô hình: Dữ liệu huấn luyện lấy từ kết quả trích xuất main content của các trang mục tiêu. Mình sử dụng framework [Fonduer](https://github.com/HazyResearch/fonduer) và phương pháp Weak supervise learning để gán nhãn và train mô hình, nói một cách đơn giản thì đây là một vòng lặp bao gồm nhiều bước từ gán nhãn -> train mô hình -> đánh giá -> điều chỉnh để dần dần nâng cao độ chính xác của mô hình và tiết kiệm chi phí cho việc gán nhãn dữ liệu. Chi tiết về cách làm bạn có thể đọc trong luận văn của mình nhé.
    - Trường hợp sử dụng: Mô hình trích rút thông tin không được tích hợp trực tiếp trong Crawler, thay vào đó mình đặt nó trong bước tiền xử lý dữ liệu, điều này cho phép Crawler lưu trữ trực tiếp phần main html, main text trong Datawarehouse và công đoạn trích rút thông tin được xem như 1 bước transform dữ liệu trong DWH.


## Kiến trúc thiết kế <a name="system_architecture"></a>
