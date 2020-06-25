---
title: Props
type: guide
order: 102
---

> Bài viết này giả định bạn đã đọc [Components Cơ bản](components.html). Đọc nó trước nếu bạn mới đến với components.

## Prop Casing (camelCase vs kebab-case)

HTML attribute names are case-insensitive, so browsers will interpret any uppercase characters as lowercase. That means when you're using in-DOM templates, camelCased prop names need to use their kebab-cased (hyphen-delimited) equivalents:
Tên thuộc tính của HTML không phân biệt chữ hoa hay thường, do đó trình duyệt sẽ dịch ký tự in hoa như in thường. Có nghĩa là khi bạn dùng DOM templates, 
``` js
Vue.component('blog-post', {
  // camelCase in JavaScript
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
})
```

``` html
<!-- kebab-case in HTML -->
<blog-post post-title="hello!"></blog-post>
```

Again, if you're using string templates, this limitation does not apply.

## Static and Dynamic Props
## Props Tĩnh và Động

So far, you've seen props passed a static value, like in:
Đến giờ, bạn đã thấy props truyền một giá trị tĩnh, như là:

```html
<blog-post title="Hành trình khám phá Vue"></blog-post>
```

You've also seen props assigned dynamically with `v-bind`, such as in:
Bạn cũng đã thấy props truyền giá trị động với `v-bind`, như sau:

```html
<blog-post v-bind:title="post.title"></blog-post>
```

Trong hai ví dụ trên, ta truyền giá trị l chuỗi, nhưng _bất kỳ_ loại giá trị nào cũng có thể được truyền cho prop. 

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

### Truyền một mảng

```html
<!-- Mặc dù mảng [234, 266, 273] là giá trị tĩnh, ta vẫn cần v-bind để nói với Vue rằng   -->
<!-- đây là một biểu thức JavaScript hơn là chuỗi ký tự.                                  -->
<blog-post v-bind:comment-ids="[234, 266, 273]"></blog-post>

<!-- Gán linh động giá trị của một biến -->
<blog-post v-bind:comment-ids="post.commentIds"></blog-post>
```

### Truyền một Đối tượng

```html
<!-- Mặc dù đối tượng sau có là giá trị tĩnh, ta vẫn cần v-bind để nói với Vue rằng   -->
<!-- đây là một biểu thức JavaScript hơn là chuỗi ký tự.                              -->
<blog-post v-bind:comments="{ id: 1, title: 'Hành trình khám phá Vue' }"></blog-post>

<!-- Gán linh động giá trị của một biến -->
<blog-post v-bind:post="post"></blog-post>
```

### Truyền thuộc tính của một đối tượng

If you want to pass all the properties of an object as props, you can use `v-bind` without an argument (`v-bind` instead of `v-bind:prop-name`). For example, given a `post` object:
Nếu bạn muốn truyền mọi thuộc tính của đối tượng như props, bạn có thể dùng `v-bind` mà không cần tham số (`v-bind` thay vì `v-bind:prop-name`). Ví dụ, cho một đối tượng `post`:

``` js
post: {
  id: 1,
  title: 'Hành trình khám phá Vue của tôi'
}ôi
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

All props form a **one-way-down binding** between the child property and the parent one: when the parent property updates, it will flow down to the child, but not the other way around. This prevents child components from accidentally mutating the parent's state, which can make your app's data flow harder to understand.
All props form a **ràng buộc xuống-một-chiều** giữa thuộc tính con và thuộc tính cha: khi một thuộc tính cha cập nhật, nó sẽ truyền xuống cho thuộc tính con, nhưng không là chiều còn lại. Điều này ngăn components con khỏi vô tình thay đổi trạng thái của components cha - điều có thể làm cho luồng dữ liệu trong app của bạn khó hiểu hơn.

In addition, every time the parent component is updated, all props in the child component will be refreshed with the latest value. This means you should **not** attempt to mutate a prop inside a child component. If you do, Vue will warn you in the console.
Thêm nữa, mỗi lần component bố được cập nhật, mọi thuộc tính trong component con sẽ được cập nhật theo giá trị mới nhất. Điều này có nghĩa bạn **không** nên gắng thay đổi một prop bên trong component con. Nếu bạn cố làm vậy, Vue sẽ cảnh báo trong console. 

There are usually two cases where it's tempting to mutate a prop:
Thường có 2 trường hợp lôi cuốn thay đổi một prop:

1. **The prop is used to pass in an initial value; the child component wants to use it as a local data property ính.** In this case, it's best to define a local data property that uses the prop as its initial value:
1. **Prop đã được truyền trong giá trị khởi tạo, component con sau đó muốn dùng nó như thuộc tính local** Trong trường hợp này, tốt nhất là định nghĩa một thuộc tính local và dùng prop làm giá trị khởi tạo của nó:

  ``` js
  props: ['initialCounter'],
  data: function () {
    return {
      counter: this.initialCounter
    }
  }
  ```

2. **The prop is passed in as a raw value that needs to be transformed.** In this case, it's best to define a computed property using the prop's value:
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
<p class="tip">Lưu ý rằng đối tượng và mảng trong JavaScript được truyền bằng tham chiếu, nên nếu prop là một mảng hay đối tượng, thay đổi nó trong component con **sẽ** ảnh hưởng đến trạng thái của component bố.</p>

## Hợp lệ hóa Prop

Components can specify requirements for its props. If a requirement isn't met, Vue will warn you in the browser's JavaScript console. This is especially useful when developing a component that's intended to be used by others.
Component có thể yêu cầu cụ thể cho prop của nó. Nếu yêu cầu không khớp, Vue sẽ cảnh báo trong console JavaScript của trình duyệt. Nó đặc biệt hữu ích khi xây dựng một component mà người khác cũng có thể dùng.

To specify prop validations, you case provide an object with validation requirements to the value of `props`, instead of an array of strings. For example:
Để chỉ định prop như nào là hợp lệ, bạn có thể cung cấp một đối tượng với các yêu cầu về tính hợp lệ cho giá trị của `props`, thay vì chỉ là một mảng các chuỗi. Ví dụ:

``` js
Vue.component('my-component', {
  props: {
    // Kiểm tra loại dữ liệu cơ bản (`null` khớp với bất kỳ loại dữ liệu nào)
    propA: Number,
    // Đa loại dữ liệu
    propB: [String, Number],
    // Yêu cầu là chuỗi
    propC: {
      type: String,
      required: true
    },
    //  Số với giá trị mặc định
    propD: {
      type: Number,
      default: 100
    },
    // Đối tượng với gía trị mặc định 
    propE: {
      type: Object,
      // Giá trị đối tượng và mảng mặc định phải được trả về từ 
      // một hàm factory (factory function)
      default: function () {
        return { message: 'hello' }
      }
    },
    // Hàm xác định tính hợp lệ tùy biến
    propF: {
      validator: function (value) {
        // Giá trị của value phải khớp với một trong các chuỗi sau
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }
})
```

When prop validation fails, Vue will produce a console warning (if using the development build).
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

to validate that the value of the `author` prop was created with `new Person`.
để xác thực rằng giá trị của prop `author` đã được tạo ra với `new Person`.

## Thuộc tính Non-Prop

A non-prop attribute is an attribute that is passed to a component, but does not have a corresponding prop defined.
Một thuộc tính non-prop là thuộc tính được truyền vào component, nhưng không có prop đã được định nghĩa tương ứng. 

While explicitly defined props are preferred for passing information to a child component, authors of component libraries can't always foresee the contexts in which their components might be used. That's why components can accept arbitrary attributes, which are added to the component's root element.
Mặc dù prop có định nghĩa rõ ràng được ưu ái trong việc truyền thông tin đến component con, người tạo ra thư viện component không thể nhìn thấy được bối cảnh mà các component có thể được dùng. Đó là lý do tại sao component cho phép thuộc tính tùy biến - cái được thêm vào thành phần component gốc. 

For example, imagine we're using a 3rd-party `bootstrap-date-input` component with a Bootstrap plugin that requires a `data-date-picker` attribute on the `input`. We can add this attribute to our component instance:
Ví dụ, hình dung chúng ta đang dùng một component `bootstrap-date-input` của bên thứ 3 với trình cắm Bootstrap yêu cầu một thuộc tính `data-date-picker` ở `input`. Ta có thể thêm thuộc tính vào đối tượng component của chúng ta:

``` html
<bootstrap-date-input data-date-picker="activated"></bootstrap-date-input>
```

Và thuộc tính `data-date-picker="activated"` sẽ tự động được thêm vào thành phần gốc của `bootstrap-date-input`.

### Replacing/Merging with Existing Attributes
### Thay thế/Sát nhập với Thuộc tính hiện có 

Hình dung đây là template cho `bootstrap-date-input`:

``` html
<input type="date" class="form-control">
```

To specify a theme for our date picker plugin, we might need to add a specific class, like this:
Để chỉ rõ một giao diện cho plugin picker ngày tháng, ta có thể cần thêm một số class, như:

``` html
<bootstrap-date-input
  data-date-picker="activated"
  class="date-picker-theme-dark"
></bootstrap-date-input>
```

In this case, two different values for `class` are defined:
Trong trường hợp này, hai `class` với giá trị khác nhau được định nghĩa:

- `form-control`, which is set by the component in its template
- `form-control` - component thiết lập trong template của nó
- `date-picker-theme-dark`, which is passed to the component by its parent
- `date-picker-theme-dark` - được component cha truyền vào. 

For most attributes, the value provided to the component will replace the value set by the component. So for example, passing `type="text"` will replace `type="date"` and probably break it! Fortunately, the `class` and `style` attributes are a little smarter, so both values are merged, making the final value: `form-control date-picker-theme-dark`.
Hầu hết mọi thuộc tính, giá trị được truyền đến một component sẽ thay thế giá trị được thiết lập bởi component đó. Ví dụ, `type="text"` được truyền sẽ thay thế `type="date"`! May mắn thay, các thuộc tính `class` và `style` thông minh hơn một chút, nên giá trị của cả hai sẽ bị sát nhập vào, tạo nên một giá trị cuối cùng kiểu: `form-control date-picker-theme-dark`.

### Disabling Attribute Inheritance
### Vô hiệu hóa sự thừa kế thuộc tính 

If you do **not** want the root element of a component to inherit attributes, you can set `inheritAttrs: false` in the component's options. For example:
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

With `inheritAttrs: false` and `$attrs`, you can manually decide which element you want to forward attributes to, which is often desirable for [base components](../style-guide/#Base-component-names-strongly-recommended):
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

This pattern allows you to use base components more like raw HTML elements, without having to care about which element is actually at its root:
Hình mẫu này cho phép bạn dùng component nền tảng giống như các thành phần HTML thô, không cần quan tâm thành phần nào thực sự ở gốc của nó:

```html
<base-input
  v-model="username"
  class="username-input"
  placeholder="Enter your username"
></base-input>
```
