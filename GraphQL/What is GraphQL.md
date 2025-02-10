# What is GraphQL?

GraphQL là ngôn ngữ truy vấn API được thiết kế để tạo điều kiện giao tiếp hiệu quả giữa máy khách và máy chủ. Nó cho phép người dùng chỉ định chính xác dữ liệu họ muốn trong phản hồi, giúp tránh các đối tượng phản hồi lớn và nhiều lệnh gọi đôi khi có thể thấy với REST API.

GraphQL service định nghĩa một quy tắc mà qua đó máy khách có thể giao tiếp với máy chủ. Máy khách không cần biết dữ liệu nằm ở đâu. Thay vào đó, khách hàng gửi truy vấn đến máy chủ GraphQL để lấy dữ liệu từ những nơi có liên quan. Vì GraphQL không phụ thuộc vào nền tảng nên nó có thể được triển khai bằng nhiều ngôn ngữ lập trình khác nhau và có thể được sử dụng để giao tiếp với hầu như mọi kho lưu trữ dữ liệu.

# How GraphQL works

Các GraphQL schema xác định cấu trúc dữ liệu của dịch vụ, liệt kê các đối tượng có sẵn (được gọi là kiểu), trường và mối quan hệ.

Dữ liệu được mô tả bởi GraphQL schema có thể được xử lý bằng ba loại thao tác:
- Truy vấn lấy dữ liệu.
- Mutations thêm, thay đổi hoặc xóa dữ liệu.
- Subscription tương tự như truy vấn, nhưng thiết lập kết nối cố định để máy chủ có thể chủ động đẩy dữ liệu đến máy khách theo định dạng đã chỉ định.

Tất cả các hoạt động GraphQL đều sử dụng cùng một endpoint và thường được gửi dưới dạng yêu cầu POST. Điều này khác biệt hơn so với REST API, sử dụng các endpoint cụ thể cho từng hoạt động trên nhiều phương thức HTTP. Với GraphQL, kiểu và tên của hoạt động sẽ xác định cách xử lý truy vấn, thay vì endpoint mà truy vấn được gửi tới hoặc phương thức HTTP được sử dụng.

Các dịch vụ GraphQL thường phản hồi các hoạt động có đối tượng JSON trong cấu trúc được yêu cầu.

# What is a GraphQL schema?

Trong GraphQL, schema biểu diễn một quy tắc giữa frontend và backend của dịch vụ. Nó định nghĩa dữ liệu có sẵn dưới dạng một loạt các kiểu, sử dụng ngôn ngữ định nghĩa schema có thể đọc được bằng con người. Các kiểu này sau đó có thể được triển khai bởi một dịch vụ.

Hầu hết các kiểu được định nghĩa là kiểu đối tượng, định nghĩa các đối tượng khả dụng và các trường và đối số mà chúng có. Mỗi trường có kiểu riêng, có thể là một đối tượng khác hoặc một scalar, enum, union, interface hoặc kiểu tùy chỉnh.

Ví dụ bên dưới cho thấy định nghĩa schema đơn giản cho loại `Product`. Toán tử `!` chỉ ra rằng trường không thể có giá trị `null` khi được gọi (tức là bắt buộc).

```graphql
    #Example schema definition

    type Product {
        id: ID!
        name: String!
        description: String!
        price: Int
    }
```

Các schema cũng phải bao gồm ít nhất một truy vấn khả dụng. Thông thường, chúng cũng chứa thông tin chi tiết về các mutation khả dụng.

## What are GraphQL queries?

Truy vấn GraphQL lấy dữ liệu từ kho dữ liệu. Chúng tương đương với các yêu cầu GET trong REST API.

Các truy vấn thường có các thành phần chính sau:
- Kiểu operation query. Về mặt kỹ thuật, đây là tùy chọn nhưng được khuyến khích vì nó cho máy chủ biết rõ ràng rằng yêu cầu đến là truy vấn.
- Tên truy vấn. Tên này có thể là bất kỳ tên nào bạn muốn. Tên truy vấn là tùy chọn, nhưng được khuyến khích vì nó có thể giúp gỡ lỗi.
- Cấu trúc dữ liệu. Đây là dữ liệu mà truy vấn sẽ trả về.
- Tùy chọn, một hoặc nhiều đối số. Chúng được sử dụng để tạo các truy vấn trả về thông tin chi tiết của một đối tượng cụ thể (ví dụ: "cho tôi biết tên và mô tả của sản phẩm có ID 123").

Ví dụ bên dưới hiển thị truy vấn có tên myGetProductQuery yêu cầu các trường tên và mô tả của sản phẩm có id là 123.

```graphql
    #Example query

    query myGetProductQuery {
        getProduct(id: 123) {
            name
            description
        }
    }
```

Lưu ý rằng loại sản phẩm có thể chứa nhiều trường trong lược đồ hơn những trường được yêu cầu ở đây. Khả năng chỉ yêu cầu dữ liệu bạn cần là một phần quan trọng của tính linh hoạt của GraphQL.

## What are GraphQL mutations?

Mutations thay đổi dữ liệu theo một cách nào đó, có thể là thêm, xóa hoặc chỉnh sửa. Chúng tương đương với các phương thức POST, PUT và DELETE của REST API.

Giống như truy vấn, mutation có loại hoạt động, tên và cấu trúc cho dữ liệu trả về. Tuy nhiên, mutation luôn lấy đầu vào của một số loại. Đây có thể là giá trị nội tuyến, nhưng trên thực tế thường được cung cấp dưới dạng biến.

Ví dụ bên dưới cho thấy một mutation để tạo ra một sản phẩm mới và phản hồi liên quan của nó. Trong trường hợp này, dịch vụ được cấu hình để tự động gán ID cho các sản phẩm mới, đã được trả về theo yêu cầu.

```graphql
    #Example mutation request

    mutation {
        createProduct(name: "Flamin' Cocktail Glasses", listed: "yes") {
            id
            name
            listed
        }
    }
```

```graphql
    #Example mutation response

    {
        "data": {
            "createProduct": {
                "id": 123,
                "name": "Flamin' Cocktail Glasses",
                "listed": "yes"
            }
        }
    }
```

## Cách thành phần của queries and mutations

Cú pháp GraphQL bao gồm một số thành phần chung cho query và mutation.

### Fields

Tất cả các loại GraphQL đều chứa các mục dữ liệu có thể truy vấn được gọi là fields. Khi bạn gửi query hoặc mutation, bạn chỉ định field nào bạn muốn API trả về. Phản hồi sẽ phản ánh nội dung được chỉ định trong yêu cầu.

Ví dụ bên dưới hiển thị truy vấn để lấy thông tin chi tiết về ID và tên của tất cả nhân viên và phản hồi liên quan. Trong trường hợp này, `id`, `name.firstname` và `name.lastname` là các field được yêu cầu.

```graphql
    #Request

    query myGetEmployeeQuery {
        getEmployees {
            id
            name {
                firstname
                lastname
            }
        }
    }
```

```graphql
    #Response

    {
        "data": {
            "getEmployees": [
                {
                    "id": 1,
                    "name" {
                        "firstname": "Carlos",
                        "lastname": "Montoya"
                    }
                },
                {
                    "id": 2,
                    "name" {
                        "firstname": "Peter",
                        "lastname": "Wiener"
                    }
                }
            ]
        }
    }
```

### Arguments

Đối số (Argument) là các giá trị được cung cấp cho các trường cụ thể. Các đối số có thể được chấp nhận cho một loại được xác định trong lược đồ.

Khi bạn gửi truy vấn hoặc đột biến có chứa đối số, máy chủ GraphQL sẽ xác định cách phản hồi dựa trên cấu hình của nó. Ví dụ, nó có thể trả về một đối tượng cụ thể thay vì chi tiết của tất cả các đối tượng.

Ví dụ bên dưới cho thấy yêu cầu getEmployee lấy ID nhân viên làm đối số. Trong trường hợp này, máy chủ chỉ phản hồi với thông tin chi tiết của nhân viên khớp với ID đó.

```graphql
    #Example query with arguments

    query myGetEmployeeQuery {
        getEmployees(id:1) {
            name {
                firstname
                lastname
            }
        }
    }
```

```graphql
    #Response to query

    {
        "data": {
            "getEmployees": [
            {
                "name" {
                    "firstname": Carlos,
                    "lastname": Montoya
                    }
                }
            ]
        }
    }
```

Note: Nếu các đối số do người dùng cung cấp được sử dụng để truy cập trực tiếp vào các đối tượng thì GraphQL API có thể dễ bị tấn công bởi các lỗ hổng kiểm soát truy cập như tham chiếu đối tượng trực tiếp không an toàn (IDOR).

### Variables

Biến cho phép bạn truyền các đối số động, thay vì truyền các đối số trực tiếp trong chính truy vấn.

Truy vấn dựa trên biến sử dụng cùng cấu trúc với truy vấn sử dụng đối số nội tuyến, nhưng một số khía cạnh nhất định của truy vấn được lấy từ từ điển biến dựa trên JSON riêng biệt. Chúng cho phép bạn tái sử dụng một cấu trúc chung giữa nhiều truy vấn, chỉ có giá trị của biến là thay đổi.

Khi xây dựng truy vấn hoặc đột biến sử dụng biến, bạn cần:
- Khai báo biến và kiểu dữ liệu
- Thêm tên biến vào vị trí thích hợp trong truy vấn.
- Truyền khóa và giá trị biến từ biến.

Ví dụ bên dưới hiển thị cùng một truy vấn như trong ví dụ trước, nhưng ID được truyền dưới dạng biến thay vì là một phần trực tiếp của chuỗi truy vấn.

```graphql
    #Example query with variable

    query getEmployeeWithVariable($id: ID!) {
        getEmployees(id:$id) {
            name {
                firstname
                lastname
            }
         }
    }

    Variables:
    {
        "id": 1
    }
```

Trong ví dụ này, biến được khai báo ở dòng đầu tiên với ($id: ID!). Dấu ! cho biết đây là trường bắt buộc cho truy vấn này. Sau đó, nó được sử dụng làm đối số ở dòng thứ hai với (id:$id). Cuối cùng, giá trị của chính biến được đặt trong từ điển JSON của biến. Để biết thông tin về cách kiểm tra các lỗ hổng này, hãy xem Lỗ hổng API GraphQL.

### Aliases

Đối tượng GraphQL không thể chứa nhiều thuộc tính có cùng tên. Ví dụ: truy vấn sau không hợp lệ vì nó cố gắng trả về loại sản phẩm hai lần.

```graphql
    #Invalid query

    query getProductDetails {
        getProduct(id: 1) {
            id
            name
        }
        getProduct(id: 2) {
            id
            name
        }
    }
```

Bí danh cho phép bạn bỏ qua hạn chế này bằng cách đặt tên rõ ràng cho các thuộc tính mà bạn muốn API trả về. Bạn có thể sử dụng bí danh để trả về nhiều phiên bản của cùng một loại đối tượng trong một yêu cầu. Điều này giúp giảm số lượng lệnh gọi API cần thiết.

Trong ví dụ bên dưới, truy vấn sử dụng bí danh để chỉ định tên duy nhất cho cả hai sản phẩm. Truy vấn này hiện đã vượt qua xác thực và các chi tiết được trả về.

```graphql
    #Valid query using aliases

    query getProductDetails {
        product1: getProduct(id: "1") {
            id
            name
        }
        product2: getProduct(id: "2") {
            id
            name
        }
    }
```

```graphql
    #Response to query

    {
        "data": {
            "product1": {
                "id": 1,
                "name": "Juice Extractor"
             },
            "product2": {
                "id": 2,
                "name": "Fruit Overlays"
            }
        }
    }
```

Note: Sử dụng các bí danh với các đột biến cho phép bạn gửi nhiều thông điệp GraphQL trong một yêu cầu HTTP một cách hiệu quả.

### Fragments

Fragment là các phần có thể tái sử dụng của truy vấn hoặc đột biến. Chúng chứa một tập hợp con các trường thuộc về loại liên quan.

Sau khi được xác định, chúng có thể được đưa vào các truy vấn hoặc đột biến. Nếu sau đó chúng được thay đổi, thay đổi sẽ được đưa vào mọi truy vấn hoặc đột biến gọi đoạn mã.

Ví dụ bên dưới hiển thị truy vấn `getProduct` trong đó thông tin chi tiết về sản phẩm được chứa trong đoạn `productInfo`.

```graphql
    #Example fragment

    fragment productInfo on Product {
        id
        name
        listed
    }
```

```graphql
    #Query calling the fragment

    query {
        getProduct(id: 1) {
            ...productInfo
            stock
        }
    }
```

```graphql
    #Response including fragment fields

    {
        "data": {
            "getProduct": {
                "id": 1,
                "name": "Juice Extractor",
                "listed": "no",
                "stock": 5
            }
        }
    }
```

## Subscriptions

`Subcription` là một loại truy vấn đặc biệt. Chúng cho phép khách hàng thiết lập kết nối lâu dài với máy chủ để máy chủ có thể đẩy các bản cập nhật theo thời gian thực đến khách hàng mà không cần phải liên tục thăm dò dữ liệu. Chúng chủ yếu hữu ích cho những thay đổi nhỏ đối với các đối tượng lớn và cho chức năng yêu cầu cập nhật nhỏ theo thời gian thực (như hệ thống trò chuyện hoặc chỉnh sửa cộng tác).

Giống như các truy vấn và mutation thông thường, subcripttion request xác định hình dạng của dữ liệu được trả về. 

Subcription thường được triển khai bằng WebSockets.

## Introspection

Introspection là một hàm GraphQL tích hợp cho phép bạn truy vấn máy chủ để biết thông tin về lược đồ. Nó thường được sử dụng bởi các ứng dụng như GraphQL IDE và các công cụ tạo tài liệu.

Giống như các truy vấn thông thường, bạn có thể chỉ định các trường và cấu trúc của phản hồi mà bạn muốn trả về. Ví dụ, bạn có thể muốn phản hồi chỉ chứa tên của các đột biến khả dụng.

Nội quan có thể gây ra rủi ro tiết lộ thông tin nghiêm trọng vì nó có thể được sử dụng để truy cập thông tin nhạy cảm (như mô tả trường) và giúp kẻ tấn công tìm hiểu cách chúng có thể tương tác với API. Tốt nhất là nên tắt tính năng nội quan trong môi trường sản xuất.




















