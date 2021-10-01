**Packet flow**

Những gói tin có đích đến là server 

|**Step**|**Table**|**Chain**||
| :-: | :-: | :-: | :-: |
|1|||Trên đường mạng (Internet)|
|2|||Tới interface|
|3|raw|PREROUTING|Chain này được dùng để kiểm soát gói tin trước khi thiết lập giám sát đường truyền (connection tracking).|
|4|||Thiết lập giám sát đường truyền|
|5|mangle|PREROUTING|Dùng để mangle gói tin vd như thay đổi TOS...|
|6|nat|PREROUTING|Sử dụng chủ yếu cho DNAT, không dùng filter ở chain này vì một số gói tin có thể bypassed|
|7|||Các routing decision được thiết lập để xác định đích đến gói tin|
|8|mangle|INPUT|mangle gói tin sau khi route nhưng vẫn chưa được gửi tới process trên máy|
|9|filter|INPUT|Đây là nơi ta filter với mọi gói tin được gửi đến server. Lưu ý rằng mọi packets có đích đến là server đều phải đi qua chain này|
|10|||Quá trình xử lí trên máy (Local process or application)|


Các gói tin bắt đầu từ server 

|**Step**|**Table**|**Chain**||
| :-: | :-: | :-: | :-: |
|1|||Local process/application|
|2|||Routing decision được đưa ra. Source address, interface nào sẽ được sử dụng...|
|3|raw|OUTPUT|đây là nơi bạn có thể đưa ra một số quyết định trước khi gói tin được thiết lập trạng thái giám sát|
|4|||Thiết lập trạng thái giám sát|
|5|mangle|OUTPUT|Nơi ta có thể mangle packets|
|6|nat|OUTPUT|Sử dụng để nat các gói tin đi từ phía firewall ra ngoài|
|7|||Thêm routing decision bởi có thể quá trình mangle và nat làm thay đổi đích đến của gói tin|
|8|filter|OUTPUT|Nơi ta filter các gói tin đi từ phía Local|
|9|mangle|POSTROUTING|Được sử dụng chủ yếu nếu ta muốn mangle gói tin sau khi nó được route nhưng chưa rời khỏi host|
|10|nat|POSTROUTING|Nơi ta SNAT|
|11|||Đi ra một interface|
|12|||Ra đường truyền|


Các gói tin được forward

|**Step**|**Table**|**Chain**||
| :-: | :-: | :-: | :-: |
|1|||Trên đường mạng (Internet)|
|2|||Tới interface|
|3|raw|PREROUTING|Chain này được dùng để kiểm soát gói tin trước khi thiết lập giám sát đường truyền (connection tracking).|
|4|||Thiết lập giám sát đường truyền|
|5|mangle|PREROUTING|Dùng để mangle gói tin vd như thay đổi TOS...|
|6|nat|PREROUTING|Sử dụng chủ yếu cho DNAT, không dùng filter ở chain này vì một số gói tin có thể bypassed|
|7|||Các routing decision được thiết lập để xác định đích đến gói tin|
|8|mangle|FORWARD|dùng để mangle các packet sau khi routing decision được đưa ra nhưng trước routing decision cuối cùng|
|9|filter|FORWARD|sau khi đã được route thì chỉ những forwarded packets mới có thể tới chain này, đây là nơi ta filter|
|10|mangle|POSTROUTING|dùng để mangle các gói tin sau khi tất cả routing decision được thiết lập nhưng vẫn chưa ra khỏi host|
|11|nat|POSTROUTING|dùng cho SNAT|
|12|||Đi ra một interface|
|13|||Ra đường truyền|
Dưới đây là mô hình miêu tả quá trình gói tin traverse qua iptables

<img src="http://i.imgur.com/2nQQzrh.jpg">

Lưu ý mọi gói tin sẽ đều phải đi qua một hoặc nhiều path trong mô hình trên. Nếu bạn có DNAT cho nó quay về network ban đầu thì nó cũng phải đi hết các chain.

