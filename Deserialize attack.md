# Deserialize attack
### Overview

![image](https://user-images.githubusercontent.com/63194321/132947745-3802292f-d483-4787-aaaf-4d1da932c7ec.png)

•	Serialization là quá trình chuyển đổi các cấu trúc dữ liệu phức tạp (các đối tượng, các trường) thành các cấu trúc có định dạng “flatter” hơn có thể gửi và nhận dưới dạng 1 dòng byte tuần tự. Serializing làm cho dữ liệu đơn giản hơn nhiều để:
    
   o Ghi dữ liệu phức tạp vào file, database, inter-process memory, …
    
   o Gửi dữ liệu phức tạp qua mạng, giữa các thành phần khác nhau của ứng dụng, …
    
•	Deserialization là quá trình khôi phục các dòng byte thành một bản sao đầy đủ chức năng của đối tượng ban đầu với trạng thái chính xác như khi nó serialization. Tất cả các thuộc tính của đối tượng gốc được lưu trữ trong luồng dữ liệu nối tiếp, bao gồm bất kỳ trường riêng nào. Để ngăn chặn một trường được nối tiếp, nó phải được đánh dấu rõ ràng là `transient` trong tuyên bố lớp.

•	Deserialization attack là một lỗ hổng cho phép kẻ tấn công thao tác các đối tượng serialization để truyền dữ liệu có hại vào trong source của đối tượng do quá trình gửi dữ liệu người dùng bị phá huỷ bởi 1 trang web. Không những chỉ các đối tượng serialization mà nó còn có thể là các đối tượng thuộc bất kỳ lớp nào trong ứng dụng web đều có thể bị huỷ bỏ ngay lập tức vì thế nó còn gọi là lỗ hổng `object injection`. Có nhiều deserialization attack được hoàn thành trước khi mà quá trình desizialization kết thúc. Điều này có nghĩa là quá trình khử hướng chính nó có thể bắt đầu 1 cuộc tấn công ngay cả khi chức năng của trang web không tương tác với đối tương độc hại.

### Impact

Tác động của cuôc tấn công này thường rất nghiêm trọng bởi vì nó cung cấp một điểm vào cho một bề mặt tấn công được gia tăng mạnh mẽ. Nó cho phép kẻ tấn công sử dụng lại mã ứng dụng hiện có theo những cách gây hại, thường thì là remote code execution, privilege escalation, arbitrary file access, and denial-of-service attacks.

### How the vulnerability arises?

Lỗ hổng này thường xảy ra do thiếu hiểu biết về mức độ nguy hiểm của dữ liệu do người dùng kiểm soát có thể giải mã không một cách không an toàn. Một cách tốt nhất thông tin đầu vào của người dùng không bao giờ được giải mã.

Tuy nhiên một số chủ sở hữu trang web nghĩ họ an toàn vì họ thực hiện hình thức kiểm tra bổ sung trên sữ liệu giải mã. Cách không hiệu quả vì không thể thực hiện vệ sinh đầu vào hay xác nhận trong mọi tình huống.

Lỗ hổng cũng xảy ra vì các đối tượng deserialized thường được cho là đáng tin cậy. Đặc biệt khi người dùng sử dụng các ngôn ngữ thao tác nhị phân vì nhà phát triển nghĩ người dùng không thể đọc hay thao tác dữ liệu một casch hiệu quả. 

Các cuộc tấn công này cũng được thực hiện do số lượng phụ thuộc tồn tại trong các trang web. Một trang web điển hình có thể thực hiện nhiều thư viện khác nhau, mỗi thư viện cũng có sự phụ thuộc riêng. Điều này tạo ra một nhóm lớn các lớp học và phương pháp khó quản lý an toàn.

### How to identify insecure deserialization?

Xác định lỗ hổng deserialization là tương đối đơn giản bất kể là kiểm tra trên whitebox hay trên blackbox.

Trong quá trình kiểm tra thì bạn nên xem xét tất cả các dữ liệu được truyền vào trang web và hãy xác định xem bất cứ thứ gì trong giống dữ kiệu serialize hay là không. Dữ liệu serialize có thể xác định một cách rõ ràng nếu như biết định dạng mà các ngôn ngữ khác nhau sử dụng.

•	PHP serialization format

PHP sử dụng định dạng chuỗi chủ yếu và có thể đọc được bởi còn người, với các chữ cái đại diện cho loại dữ liệu và số đại diện cho độ dài của mỗi mục nhập.

VD:

![image](https://user-images.githubusercontent.com/63194321/132947856-4853fe7a-9427-4403-b982-fc07d510fdf0.png)
![image](https://user-images.githubusercontent.com/63194321/132947860-1f0c1fef-392a-4576-abad-2de2eeb476ad.png)

Các phương pháp gốc để nối tiếp PHP là `and`. Nếu bạn có quyền truy cập mã nguồn, bạn nên bắt đầu bằng cách tìm kiếm bất cứ nơi nào trong mã và điều tra thêm. ```serialize(), unserialize(), unserialize()```

•	Java serialization format

Đây là ngôn ngữ sủ dụng định dạng serialize dạng nhị phân. Điều này khiến dữ liệu trở nên khó bị đọc hơn nhưng vẫn có thể xác định được dữ liệu serialize khi nhận thấy 1 vài dấu hiệu. Ví dụ, các đối tượng Java serialize luôn bắt đầu bằng các byte giống nhau, được mã hóa như trong thập lục phân và trong Base64. `ac ed rO0`

Bất kỳ lớp nào thực hiện giao diện đều có thể được serialize hay deserialize. Nếu bạn có quyền truy cập mã nguồn, hãy lưu ý bất kỳ mã nào sử dụng phương pháp này, được sử dụng để đọc và khử dữ liệu từ một mã. `java.io.SerializablereadObject()InputStream`

### How to exploit vulnerabilities?

#### •	Manipulating serialized objects

Khi khai thác 1 lỗ hổng deserialization có thể dễ dàng thay đổi 1 thuộc tính trong 1 đối tượng serialize. Khi trạng thái của đối tượng vẫn còn tồn tại thì có thể nghiên cứu dữ liệu serialize để xác định và chỉnh sửa các giá trị thuộc tính. Sau đó có thể chuyển các đối tượng độc hại vào trong trang web thông qua quá trình deserialization của nó.

Nói chung có 2 cách để tiếp cận các đối tượng serialize. Cách đầu là chỉnh sửa trực tiếp ở luồng byte. Cách 2 là viết 1 kịch bản ngắn bằng ngôn ngữ tương ứng để tự tạo và nối tiếp đối tượng mới.

#### •	Modifying object attributes

Khi giả mạo giữ liệu kẻ tấn công bảo quản một đối tượng serialize hợp lệ thì quá trình deserialization sẽ tạo ra 1 đối tượng phía máy chủ với giá trị thuộc tính đã sửa đổi.

VD: Nếu kẻ tấn công phát hiện đối tượng serialize này trong yêu cầu HTTP, họ có thể giải mã nó để tìm luồng byte sau: User

![image](https://user-images.githubusercontent.com/63194321/132947992-57846050-2633-4ddc-9a2e-ea1eeb96e685.png)

Kẻ tấn công chỉ cần thay đổi giá trị boolean của thuộc tính thành true (1), mã hóa lại đối tượng và ghi đè cookie hiện tại của chúng với giá trị đã sửa đổi này.

![image](https://user-images.githubusercontent.com/63194321/132948081-b1bd3ce2-c1ff-459c-897e-ea29babcc4d0.png)

Chỉnh sửa một giá trị thuộc tính theo cách này cho thấy bước đầu tiên hướng tới việc truy cập vào lượng lớn bề mặt tấn công bị phơi bày bởi sự khử quan tâm không an toàn.

#### •	Modifying data types

Logic dựa trên PHP đặc biệt dễ bị tổn thương bởi loại cung cấp dữ liệu do hành vi của nhà điều hành so sánh lỏng lẻo khi so sánh các loại dữ liệu khác nhau. Nếu bạn thực hiện so sánh lỏng lẻo giữa số nguyên và chuỗi, PHP sẽ cố gắng chuyển đổi chuỗi thành số nguyên. `5 == "5 of something"`. Điều này còn tồi tệ hơn khi so sánh `0` với 1 chuỗi. `0 == "Example string"` cả 2 đều sẽ được trả về là `true`

Nếu kẻ tấn công sủ dụng điều này để thực hiện 1 cuộc tấn công.

![image](https://user-images.githubusercontent.com/63194321/132948132-928b0d8f-b3f0-47ba-9f74-877b8d0a1086.png)

Giả sử kẻ tấn công đã sửa đổi thuộc tính mật khẩu để nó chứa số nguyên thay vì chuỗi dự kiến. Nếu mã lấy mật khẩu trực tiếp từ yêu cầu, mã sẽ được chuyển đổi thành chuỗi và điều kiện sẽ đánh giá thành `0true0false`
Lưu ý rằng khi sửa đổi các loại dữ liệu ở bất kỳ định dạng đối tượng nối tiếp nào, điều quan trọng cần nhớ là cập nhật bất kỳ nhãn loại và chỉ báo độ dài nào trong dữ liệu nối tiếp. Nếu không, đối tượng nối tiếp sẽ bị hỏng và sẽ không bị hủy hoại.

#### •	Using application functionality

Cũng như chỉ cần kiểm tra các giá trị thuộc tính, chức năng của trang web cũng có thể thực hiện các hoạt động nguy hiểm trên dữ liệu từ một đối tượng deserialize. Trong trường hợp này, bạn có thể sử dụng deserialization không an toàn để truyền dữ liệu bất ngờ và tận dụng các chức năng liên quan để gây thiệt hại.

Ví dụ: là một phần của chức năng `delete user` của trang web, ảnh đại diện của người dùng sẽ bị xóa bằng cách truy cập đường dẫn tệp trong thuộc tính. Nếu điều này được tạo ra từ một đối tượng serialize, kẻ tấn công có thể khai thác điều này bằng cách chuyển trong một đối tượng đã sửa đổi với tập hợp đến đường dẫn tệp tùy ý. Xóa tài khoản người dùng của chính họ sau đó cũng sẽ xóa tệp tùy ý này. `$user->image_location$userimage_location`

#### •	Magic methods

Magic methods là một tập hợp con đặc biệt của các phương pháp mà bạn không cần phải gọi một cách rõ ràng. Thay vào đó, chúng được gọi tự động bất cứ khi nào một sự kiện hoặc kịch bản cụ thể xảy ra. Các phương pháp này là một tính năng phổ biến của lập trình hướng đối tượng bằng các ngôn ngữ khác nhau. Đôi khi chúng được chỉ định bằng tiền tố hoặc xung quanh tên phương pháp với dấu nhấn kép.

Vd . Một số hàm magic methods.

•	PHP: 
![image](https://user-images.githubusercontent.com/63194321/132948194-cdc28604-8fc2-4aa0-8cad-3e2766566d0d.png)

•	Java: 
![image](https://user-images.githubusercontent.com/63194321/132948205-0ca893ec-197b-44ca-9d58-8c2dcfe4836f.png)

#### •	Injecting arbitrary objects

Trong lập trình hướng đối tượng, các phương pháp có sẵn cho một đối tượng được xác định bởi lớp của nó. Do đó, nếu kẻ tấn công có thể thao túng lớp đối tượng nào đang được truyền dưới dạng dữ liệu nối tiếp, chúng có thể ảnh hưởng đến mã nào được thực thi sau và thậm chí trong quá trình deserialization. 

Các phương pháp khử hướng thường không kiểm tra những gì chúng đang deserializing. Điều này có nghĩa là bạn có thể vượt qua các đối tượng thuộc bất kỳ lớp nối tiếp nào có sẵn cho trang web và đối tượng sẽ bị hủy bỏ.
Nếu kẻ tấn công có quyền truy cập vào mã nguồn, họ có thể nghiên cứu chi tiết tất cả các lớp có sẵn. Để xây dựng một khai thác đơn giản, họ sẽ tìm kiếm các lớp có chứa các phương pháp ma thuật deserialization, sau đó kiểm tra xem có ai trong số họ thực hiện các hoạt động nguy hiểm trên dữ liệu có thể kiểm soát được hay không. Kẻ tấn công sau đó có thể vượt qua trong một đối tượng serialize của lớp này để sử dụng magic methods của nó để khai thác.

#### •	Gadget chains

Một `gadget` là một đoạn mã tồn tại trong ứng dụng có thể giúp kẻ tấn công đạt được một mục tiêu cụ thể. Một tiện ích cá nhân có thể không trực tiếp làm bất cứ điều gì có hại với đầu vào của người dùng. Tuy nhiên, mục tiêu của kẻ tấn công có thể chỉ đơn giản là gọi một phương pháp sẽ chuyển đầu vào của họ vào một tiện ích khác. Bằng cách xích nhiều tiện ích lại với nhau theo cách này, kẻ tấn công có khả năng chuyển đầu vào của chúng vào một `sink gadget` nguy hiểm, nơi nó có thể gây ra thiệt hại tối đa.

Không giống như một số loại khai thác khác, Gadget chains không phải là một tải trọng của các phương pháp xích được xây dựng bởi kẻ tấn công. Tất cả các mã đã tồn tại trên trang web. Điều duy nhất kẻ tấn công kiểm soát là dữ liệu được truyền vào chuỗi tiện ích. Điều này thường được thực hiện bằng cách sử dụng một magic methods được gọi trong quá trình khử hơi, đôi khi được gọi là `kick-off gadget`.

#### •	PHAR deserialization

![image](https://user-images.githubusercontent.com/63194321/132948270-7a91f650-194c-41cf-a7ff-004ed5a656e1.png)

Cho đến nay, chúng tôi đã xem xét chủ yếu việc khai thác các lỗ hổng deserialization, nơi trang web rõ ràng deserializes đầu vào của người dùng. Tuy nhiên, trong PHP đôi khi có thể khai thác deserialization ngay cả khi không có sử dụng rõ ràng của phương pháp `unserialize()`

PHP cung cấp một số gói kiểu URL mà bạn có thể sử dụng để xử lý các giao thức khác nhau khi truy cập đường dẫn tệp. Một trong số đó là wrapper, cung cấp giao diện luồng để truy cập các tệp PHP Archive(): `phar://`, `.phar`.

Tài liệu PHP tiết lộ rằng các tệp biểu hiện chứa siêu dữ liệu serialize. Điều quan trọng, nếu bạn thực hiện bất kỳ thao tác hệ thống tệp nào trên luồng, siêu dữ liệu này sẽ bị bỏ hoang ngầm. Điều này có nghĩa là một luồng có khả năng là một vector để khai thác deserialization không an toàn, miễn là bạn có thể chuyển luồng này vào một phương pháp hệ thống tệp. `phar://`

Kỹ thuật này cũng yêu cầu bạn tải lên máy chủ bằng cách nào đó. Một cách tiếp cận là sử dụng chức năng tải lên hình ảnh.

VD: Nếu bạn có thể tạo một tệp đa ngôn ngữ, với PHAR giả dạng là JPG đơn giản, đôi khi bạn có thể bỏ qua kiểm tra xác thực của trang web. Nếu sau đó bạn có thể buộc trang web tải "JPG" đa ngôn ngữ này từ luồng phar://, bất kỳ dữ liệu có hại nào bạn tiêm qua siêu dữ liệu PHAR sẽ bị hủy bỏ. Vì phần mở rộng tệp không được kiểm tra khi PHP đọc luồng, không quan trọng tệp sử dụng phần mở rộng hình ảnh.

Miễn là lớp của đối tượng được trang web hỗ trợ, cả hai magic methods __wakeup () và __destruct () có thể được gọi theo cách này, cho phép bạn có khả năng khởi động một chuỗi tiện ích bằng cách sử dụng kỹ thuật này.

### How to prevent?

Chúng ta có thể bảo vệ khỏi các cuộc deserialize attack bằng các cách sau:

•	Xác thực đầu vào của người dùng.

•	Không bao giờ deserialize dữ liệu từ một nguồn không đáng tin cậy.

•	Chạy mã deserialization quan hệ với các đặc quyền truy cập hạn chế.

•	Khi truyền dữ liệu giữa hai hệ thống, hãy kiểm tra xem đối tượng có bị giả mạo hay không. Người ta có thể sử dụng checkums cho việc này.

•	Nếu có, hãy sử dụng các phương pháp khử trùng an toàn. Ví dụ, trong python sử dụng ` yaml.safe_load()` thay vì  `yaml.load()`

### Reference

Blog Medium: https://medium.com/gdg-vit/deserialization-attacks-d312fbe58e7d

Blog: https://blog.sonarsource.com/new-php-exploitation-technique

Port Swigger: https://portswigger.net/web-security/deserialization/exploiting

