---
title: Props
type: guide
order: 102
---

> Bài viết này giả định bạn đã đọc [Components Cơ bản](components.html). Đọc nó trước nếu bạn mới đến với components.

## Prop Casing (camelCase vs kebab-case)

Tên thuộc tính của HTML không phân biệt ký tự hoa hay thường, do đó trình duyệt sẽ dịch ký tự in hoa như in thường. Có nghĩa là khi bạn dùng DOM templates, prop có tên theo kiểu camelCased (lạc đà) cần sử dụng dạng kebab-cased (có dấu gạch ngang nối giữa) tương đương của nó:
``` js
Vue.component('blog-post', {
  // camelCase trong JavaScript
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
})
```

``` html
<!-- kebab-case trong HTML -->
<blog-post post-title="Xin chào!"></blog-post>
```

Nếu bạn sử dụng template cho string (chuỗi ký tự), thì điều này không áp dụng.

## Props Tĩnh và Động

Đến giờ, bạn đã thấy một giá trị tĩnh truyền vào props, như sau:

```html
<blog-post title="Hành trình khám phá Vue"></blog-post>
```

Bạn cũng đã thấy props được gán giá trị động với `v-bind`, như sau:

```html
<blog-post v-bind:title="post.title"></blog-post>
```

Trong hai ví dụ trên, ta truyền giá trị là string, nhưng _bất kỳ_ loại giá trị nào cũng có thể được truyền cho prop. 

### Truyền một số.

```html
<!-- Mặc dù `42` là giá trị tĩnh, ta vẫn cần v-bind để nói với Vue rằng    -->
<!-- đây là một biểu thức JavaScript hơn là một chuỗi ký tự.               -->
<blog-post v-bind:likes="42"></blog-post>

<!-- Gán linh động giá trị của một biến -->
<blog-post v-bind:likes="post.likes"></blog-post>
```

### Truyền một Boolean

```html
<!-- Prop không có giá trị sẽ tự ngầm định là `true`. -->
<blog-post favorited></blog-post>

<!-- Mặc dù `false` là giá trị tĩnh, ta vẫn cần v-bind để nói với Vue rằng  -->
<!-- đây là một biểu thức JavaScript hơn là chuỗi ký tự.                    -->
<base-input v-bind:favorited="false">

<!-- Gán linh động giá trị của một biến -->
<base-input v-bind:favorited="post.currentUserFavorited">
```

### Truyền một mảng (Array)

```html
<!-- Mặc dù mảng [234, 266, 273] là giá trị tĩnh, ta vẫn cần v-bind để nói với Vue rằng   -->
<!-- đây là một biểu thức JavaScript hơn là chuỗi ký tự.                                  -->
<blog-post v-bind:comment-ids="[234, 266, 273]"></blog-post>

<!-- Gán linh động giá trị của một biến -->
<blog-post v-bind:comment-ids="post.commentIds"></blog-post>
```

### Truyền một Đối tượng (Object)

```html
<!-- Mặc dù đối tượng sau mang giá trị tĩnh, ta vẫn cần v-bind để nói với Vue rằng   -->
<!-- đây là một biểu thức JavaScript hơn là chuỗi ký tự.                              -->
<blog-post v-bind:comments="{ id: 1, title: 'Hành trình khám phá Vue' }"></blog-post>

<!-- Gán linh động giá trị của một biến -->
<blog-post v-bind:post="post"></blog-post>
```

### Truyền thuộc tính của một Object

Nếu bạn muốn truyền mọi thuộc tính của đối tượng làm props, bạn có thể dùng `v-bind` mà không cần tham số (`v-bind` thay vì `v-bind:prop-name`). Ví dụ, cho một đối tượng `post`:

``` js
post: {
  id: 1,
  title: 'Hành trình khám phá Vue của tôi'
}
```

Template như sau:

``` html
<blog-post v-bind="post"></blog-post>
```

Sẽ tương đương với:

``` html
<blog-post
  v-bind:id="post.id"
  v-bind:title="post.title"
></blog-post>
```

## Luồng Data một chiều

Mọi props tạo thành một **ràng buộc một-chiều-đi-xuống** (**one-way-down binding**) giữa thuộc tính con và thuộc tính cha: khi một thuộc tính cha cập nhật, nó sẽ truyền xuống cho thuộc tính con, nhưng không có chiều còn lại. Điều này ngăn components con khỏi vô tình thay đổi trạng thái của components cha - điều có thể làm cho luồng dữ liệu (data flow) trong app của bạn khó hiểu hơn.

Thêm nữa, mỗi lần component bố được cập nhật, mọi thuộc tính trong component con sẽ được cập nhật theo giá trị mới nhất. Điều này có nghĩa bạn **không** nên gắng thay đổi một prop bên trong component con. Nếu bạn cố làm vậy, Vue sẽ cảnh báo trong console. 

Thường có 2 trường hợp lôi cuốn thay đổi một prop:

1. **Prop được dùng để truyền vào giá trị khởi tạo, component con sau đó muốn dùng nó như thuộc tính local** Trong trường hợp này, tốt nhất là định nghĩa một thuộc tính local và dùng prop làm giá trị khởi tạo của nó:

  ``` js
  props: ['initialCounter'],
  data: function () {
    return {
      counter: this.initialCounter
    }
  }
  ```

2. **Prop được truyền đến với giá trị thô và cần biến đổi** Trong trường hợp này, tốt nhất là định nghĩa một thuộc tính computed sử dụng giá trị của prop:

  ``` js
  props: ['size'],
  computed: {
    normalizedSize: function () {
      return this.size.trim().toLowerCase()
    }
  }
  ```

<p class="tip">Note that objects and arrays in JavaScript are passed by reference, so if the prop is an array or object, mutating the object or array itself inside the child component **will** affect parent state.</p>
<p class="tip">Lưu ý rằng Object và Array trong JavaScript được truyền bằng tham chiếu (pass by reference), nên nếu thay đổi prop là Array hay Object trong component con **sẽ** ảnh hưởng đến trạng thái của component bố.</p>

## Hợp lệ hóa Prop

Component có thể yêu cầu cụ thể cho prop của nó. Nếu yêu cầu không khớp, Vue sẽ cảnh báo trong console JavaScript của trình duyệt. Điều này đặc biệt có lợi khi xây dựng một component mà người khác có thể cũng dùng.

Để xác định prop như nào là hợp lệ, bạn có thể cung cấp cho Object các yêu cầu về tính hợp lệ cho giá trị của `props`, thay vì chỉ là một Array các chuỗi. Ví dụ:

``` js
Vue.component('my-component', {
  props: {
    // Kiểm tra loại dữ liệu cơ bản (`null` khớp với mọi loại dữ liệu)
    propA: Number,
    // Đa loại dữ liệu
    propB: [String, Number],
    // Yêu cầu là String
    propC: {
      type: String,
      required: true
    },
    //  Số với giá trị mặc định
    propD: {
      type: Number,
      default: 100
    },
    // Object với giá trị mặc định 
    propE: {
      type: Object,
      // Giá trị đối tượng và mảng mặc định phải được trả về từ 
      // một hàm factory (factory function)
      default: function () {
        return { message: 'hello' }
      }
    },
    // Tùy biến hàm xác định tính hợp lệ
    propF: {
      validator: function (value) {
        // Giá trị của value phải khớp với một trong các chuỗi sau
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }
})
```

Khi xác thực tính hợp lệ của prop không đúng yêu cầu, Vue sẽ báo trong console (nếu sử dụng development build)

<p class="tip">Nhớ rằng prop đã được xác thực tính hợp lệ **trước khi** một đối tượng của component được tạo, nên thuộc tính của đối tượng (vd: `data`, `computed`, v.v...) sẽ không có sẵn bên trong các hàm `default` hoặc `validator`.</p>

### Xác thực loại dữ liệu

`type` (loại dữ liệu) can be one of the following native constructors:

- String
- Number
- Boolean
- Function
- Object
- Array
- Symbol

Thêm nữa, `type` cũng có thể là một hàm khởi tạo tùy biến và càng chắc chắn nếu kiểm tra với `instanceof`. Ví dụ, cho một hàm khởi tạo như bên dưới:

```js
function Person (firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
}
```

Bạn sẽ dùng như sau:

```js
Vue.component('blog-post', {
  props: {
    author: Person
  }
})
```

để xác thực rằng giá trị của prop `author` đã được tạo ra với `new Person`.

## Thuộc tính Non-Prop

Một thuộc tính non-prop là thuộc tính được truyền vào component, nhưng không có prop đã được định nghĩa tương ứng. 

Mặc dù prop có định nghĩa rõ ràng được ưu ái trong việc truyền thông tin đến component con, người tạo ra thư viện component không thể nhìn thấy được bối cảnh mà các component có thể được dùng. Đó là lý do tại sao component cho phép thuộc tính tùy biến - được thêm vào thành phần component gốc. 

Ví dụ, hình dung chúng ta đang dùng một component `bootstrap-date-input` của bên thứ 3 với trình cắm Bootstrap yêu cầu một thuộc tính `data-date-picker` ở `input`. Ta có thể thêm thuộc tính vào instance của component chúng ta:

``` html
<bootstrap-date-input data-date-picker="activated"></bootstrap-date-input>
```

Và thuộc tính `data-date-picker="activated"` sẽ tự động được thêm vào thành phần gốc của `bootstrap-date-input`.

### Thay thế/Sát nhập với Thuộc tính hiện có 

Hình dung đây là template cho `bootstrap-date-input`:

``` html
<input type="date" class="form-control">
```

Để chỉ rõ một giao diện cho plugin hiển thị picker ngày tháng, ta có thể cần thêm một số class, như:

``` html
<bootstrap-date-input
  data-date-picker="activated"
  class="date-picker-theme-dark"
></bootstrap-date-input>
```

Trong trường hợp này, hai `class` với giá trị khác nhau được định nghĩa:

- `date-picker-theme-dark`, which is passed to the component by its parent
- `date-picker-theme-dark` - được component cha truyền vào. 

Hầu hết mọi thuộc tính, giá trị được truyền đến một component sẽ thay thế giá trị được thiết lập bởi component đó. Ví dụ, `type="text"` được truyền sẽ thay thế `type="date"`! May mắn thay, các thuộc tính `class` và `style` thông minh hơn một chút, nên giá trị của cả hai sẽ bị sát nhập vào, tạo nên một giá trị cuối cùng kiểu: `form-control date-picker-theme-dark`.

### Vô hiệu hóa sự thừa kế thuộc tính 

Nếu bạn **không** muốn thành phần gốc của component thừa kế thuộc tính, bạn có thể thiết lập `inheritAttrs: false` trong tùy chọn component. Ví dụ:

```js
Vue.component('my-component', {
  inheritAttrs: false,
  // ...
})
```

This can be especially useful in combination with the `$attrs` instance property, which contains the attribute names and values passed to a component, such as:
Cái này có thể đặc biệt có ích trong việc kết hợp các thuộc tính `$attrs`, bao gồm cả tên và giá trị của thuộc tính được truyền vào một component, như sau:

```js
{
  class: 'username-input',
  placeholder: 'Enter your username'
}
```

Với `inheritAttrs: false` và `$attrs`, bạn có thể tự tay quyết định thành phần nào bạn muốn chuyển tiếp thuộc tính đến, cái nào thường dành cho [Components nền t](../style-guide/#Base-component-names-strongly-recommended):

```js
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on:input="$emit('input', $event.target.value)"
      >
    </label>
  `
})
```

Hình mẫu này cho phép bạn dùng component nền tảng giống như các thành phần HTML thô, không cần quan tâm thành phần nào thực sự ở gốc của nó:

```html
<base-input
  v-model="username"
  class="username-input"
  placeholder="Enter your username"
></base-input>
```
