
[Source](https://mithril.js.org/simple-application.html "Permalink to Simple application - Mithril.js")

# Ứng dụng đơn giản - Mithril.js

Hãy phát triển một ứng dụng đơn giản bao gồm một số khía cạnh chính của Single Page Applications

Một ví dụ tương tác đang chạy có thể được xem ở đây [flems: Simple Application][1]

Đầu tiên hãy tạo một điểm bắt đầu cho ứng dụng. Tạo một file `index.html`:
    
    
    
    
        
            
            
            
        
        
            http://www.google-analytics.com/ga.js"> src="">https://mithril.js.org/bin/app.js">
        
    
    

`<!doctype html>` dòng cho biết đây là văn bản HTML 5. Thẻ meta `charset` đầu tiên cho biết loại mã hóa của tài liệu và thẻ meta` viewport` quy định cách các trình duyệt di động hiển thị trang. Thẻ `title` chứa văn bản được hiển thị trên tab trình duyệt cho ứng dụng này và thẻ` script` cho biết đường dẫn đến tệp Javascript điều khiển ứng dụng là gì.

Chúng ta có thể tạo toàn bộ ứng dụng trong duy nhất một file Javascript, nhưng làm như vậy sẽ khiến khó khăn trong việc điều hướng codebase. Thay vào đó, hãy chia code thành các _modules_ và gom nhóm các mô đun này thành _bundle_ `bin / app.js`.

Có nhiều cách để thiết lập một công cụ bundler, nhưng hầu hết được phân phối qua NPM. Trên thực tế, hầu hết các thư viện và công cụ Javascript hiện đại được phân phối theo cách đó, bao gồm cả Mithril. NPM là viết tắt của Node.js Package Manager. Để tải NPM, [cài đặt Node.js] [2]; NPM được cài đặt tự động cùng với nó. Khi bạn đã cài đặt Node.js và NPM, hãy mở command line  và chạy lệnh này: 
    
    
    npm init -y
    

Nếu NPM được cài đặt thành công, một ffile `package.json` sẽ được tạo ra. file này sẽ chứa tệp meta-description là bộ xương của project. Vui lòng chỉnh sửa thông tin project và tác giả trong file này.

* * *

Để cài đặt Mithril, làm theo hướng dẫn trong trang [installation][3]. Khi bạn đã cài đặt bộ xương project với Mithril, chúng ta sẵn sàng tạo ứng dụng.

Hãy bắt đầu bằng cách tạo một module để lưu trạng thái của chúng ta.Hãy tạo một file có tên là `src/models/User.js`
    
    
    // src/models/User.js
    var User = {
        list: []
    }
    
    module.exports = User
    

Bây giờ chúng ta hãy thêm code để tải một số dữ liệu từ server. Để giao tiếp với một server, chúng ta có thể sử dụng tiện ích XHR của Mithril, `m.request`. Đầu tiên, chúng ta include Mithril vào module:
    
    
    // src/models/User.js
    var m = require("mithril")
    
    var User = {
        list: []
    }
    
    module.exports = User
    

Tiếp theo chúng ta tạo một hàm sẽ kích hoạt lời gọi XHR. Hãy gọi nó là 'loadList`
    
    
    // src/models/User.js
    var m = require("mithril")
    
    var User = {
        list: [],
        loadList: function() {
            // TODO: make XHR call
        }
    }
    
    module.exports = User
    

Sau đó, chúng ta có thể thêm một lời gọi `m.request` để thực hiện một request XHR. Đối với hướng dẫn này, chúng ta sẽ tạo lời gọi XHR đến API [REM] [4], một API REST giả lập được thiết kế để tạo mẫu nhanh. API này trả về danh sách người dùng từ endpoint `GET https: // rem-rest-api.herokuapp.com / api / users`. Hãy sử dụng `m.request` để tạo một request XHR và lưu dữ liệu với response của endpoint đó.
    
    
    // src/models/User.js
    var m = require("mithril")
    
    var User = {
        list: [],
        loadList: function() {
            return m.request({
                method: "GET",
                url: "https://rem-rest-api.herokuapp.com/api/users",
                withCredentials: true,
            })
            .then(function(result) {
                User.list = result.data
            })
        },
    }
    
    module.exports = User
    

Tùy chọn `method` là [HTTP method] [5]. Để lấy dữ liệu từ server mà không gây ra các ảnh hưởng phụ trên máy chủ, chúng ta cần sử dụng phương thức `GET`.`url` là địa chỉ cho endpoint API. Dòng `withCredentials: true` chỉ ra rằng chúng ta đang sử dụng các cookie (là một yêu cầu cho API REM).

Lời gọi `m.request` trả về Promise xử lý dữ liệu từ endpoint. Theo mặc định, Mithril giả định body của một response  HTTP có định dạng JSON và tự động phân tích cú pháp nó thành đối tượng hoặc mảng Javascript. Lệnh gọi `.then` chạy khi yêu cầu XHR hoàn tất. Trong trường hợp này, callbacks gán mảng `result.data` cho ` User.list`.

Lưu ý rằng chúng ta cũng có câu lệnh `return` trong` loadList`. Đây là một thói quen tốt khi làm việc với Promises, cho phép chúng ta đăng ký nhiều callbacks để chạy sau khi hoàn thành request XHR.

Mô hình đơn giản này cho thấy hai thành phần: `User.list` (một mảng các đối tượng người dùng), và` User.loadList` (một phương thức lấy `User.list` với dữ liệu server).

* * *

Bây giờ, hãy tạo một mô-đun view để chúng ta có thể hiển thị dữ liệu từ User model module của chúng ta.

Tạo một file có tên là `src/views/UserList.js`. Trước tiên, hãy đưa Mithril và mô hình của chúng ta, vì chúng ta sẽ cần sử dụng cả hai:
    
    
    // src/views/UserList.js
    var m = require("mithril")
    var User = require("../models/User")
    

Tiếp theo, hãy tạo một thành phần Mithril. Một thành phần đơn giản là một đối tượng có phương thức `view`:
    
    
    // src/views/UserList.js
    var m = require("mithril")
    var User = require("../models/User")
    
    module.exports = {
        view: function() {
            // TODO add code here
        }
    }
    

Theo mặc định, Mithril views được mô tả bằng cách sử dụng [hyperscript][6]. Hyperscript ung cấp một cú pháp ngắn gọn có thể rút gọn tự nhiên hơn HTML cho các thẻ phức tạp, vì cú pháp của nó đơn giản là Javascript, có thể tận dụng rất nhiều hệ sinh thái công cụ Javascript: Ví dụ [Babel][7], [JSX][8] (phần mở rộng cú pháp  inline-HTML), [eslint][9] (linting), [uglifyjs][10] (minification), [istanbul][11] (code coverage), [flow][12] (static type analysis), etc.

Hãy sử dụng Mersril hyperscript để tạo danh sách các item. Hyperscript là cách tốt nhất để viết các views Mithril, nhưng [JSX là một lựa chọn phổ biến khác mà bạn có thể khám phá] [8] một khi bạn cảm thấy thoải mái hơn với những điều cơ bản:
    
    
    // src/views/UserList.js
    var m = require("mithril")
    var User = require("../models/User")
    
    module.exports = {
        view: function() {
            return m(".user-list")
        }
    }
    

Chuỗi `" .user-list "` là một bộ chọn CSS, và như bạn mong đợi, `.user-list` đại diện cho một lớp. Khi thẻ không được chỉ định, `div` là giá trị mặc định. Vì vậy, chế độ xem này tương đương với `<div class="user-list"></div>`
`.

Bây giờ, hãy tham khảo danh sách người dùng từ mô hình mà chúng ta đã tạo trước đó (`User.list`) để tự động lặp qua dữ liệu:
    
    
    // src/views/UserList.js
    var m = require("mithril")
    var User = require("../models/User")
    
    module.exports = {
        view: function() {
            return m(".user-list", User.list.map(function(user) {
                return m(".user-list-item", user.firstName + " " + user.lastName)
            }))
        }
    }
    

Vì `User.list` là một mảng Javascript, và vì các khung nhìn hyperscript chỉ là Javascript, chúng ta có thể lặp qua mảng bằng cách sử dụng phương thức` .map`. Điều này tạo ra một loạt các vnodes đại diện cho một danh sách các `div`, mỗi cái chứa tên của một người dùng

Tất nhiên, vấn đề là chúng ta không bao giờ gọi hàm `User.loadList`. Do đó, `User.list` vẫn là một mảng trống, và do đó view này sẽ hiển thị một trang trống. Vì chúng ta muốn hàm `User.loadList` được gọi khi chúng ta render thành phần này, chúng ta có thể tận dụng lợi thế của thành phần [lifecycle methods:] [13]:
    
    
    // src/views/UserList.js
    var m = require("mithril")
    var User = require("../models/User")
    
    module.exports = {
        oninit: User.loadList,
        view: function() {
            return m(".user-list", User.list.map(function(user) {
                return m(".user-list-item", user.firstName + " " + user.lastName)
            }))
        }
    }
    

Lưu ý rằng chúng ta đã thêm phương thức `oninit` vào thành phần, tham chiếu` User.loadList`. Điều này có nghĩa là khi thành phần khởi tạo, User.loadList  được gọi, sẽ kích hoạt một request XHR. Khi server trả về một response, `User.list` gán giá trị.

Cũng lưu ý rằng chúng ta **không** thực hiện `oninit: User.loadList ()` (với dấu ngoặc đơn ở cuối). Sự khác biệt là `oninit: User.loadList ()` gọi hàm một lần và ngay lập tức, nhưng `oninit: User.loadList` chỉ gọi hàm đó khi thành phần render. Đây là một sự khác biệt quan trọng và một cạm bẫy phổ biến cho các nhà phát triển mới đến javascript:  gọi hàm ngay lập tức có nghĩa là request XHR sẽ kích hoạt ngay sau khi source code  được thực thi, ngay cả khi thành phần không bao giờ được renders.. Ngoài ra, nếu thành phần được tạo lại (thông qua điều hướng qua lại trong ứng dụng), hàm sẽ không được gọi lại như mong đợi.

Hãy render view từ file đầu vào `src / index.js` mà chúng ta đã tạo trước đó:
    
    
    // src/index.js
    var m = require("mithril")
    
    var UserList = require("./views/UserList")
    
    m.mount(document.body, UserList)
    

Lệnh `m.mount` render thành phần được chỉ định (` UserList`) thành một phần tử DOM (`document.body`), xóa bất kỳ DOM nào đã có trước đó. Mở tệp HTML trong trình duyệt bây giờ sẽ hiển thị danh sách tên mọi người

* * *

Ngay bây giờ, danh sách trông khá đơn giản vì chúng tôi chưa xác định bất kỳ kiểu nào.

Có nhiều conventions và thư viện tương tự giúp tổ chức các style ứng dụng ngày nay. Một số, như [Bootstrap] [14] quyết định cho một bộ cấu trúc HTML cụ thể và tên lớp có ý nghĩa về ngữ nghĩa, có xu hướng cung cấp dissonance nhận thức thấp, nhưng nhược điểm của việc tùy biến trở nên khó khăn hơn. Những người khác, giống như [Tachyons] [15] cung cấp một số lượng lớn các tên lớp học tự mô tả, nguyên tử với chi phí tự đặt tên lớp là không có ngữ nghĩa. "CSS-in-JS" là một loại hệ thống CSS đang phát triển phổ biến, về cơ bản bao gồm CSS phạm vi thông qua công cụ biên dịch. Các thư viện CSS-in-JS đạt được khả năng bảo trì bằng cách giảm kích thước của không gian vấn đề, nhưng đến với chi phí có độ phức tạp cao.

Bất kể convention / thư viện CSS bạn chọn là gì, nguyên tắc chung là tránh các khía cạnh xếp tầng của CSS. Để giữ cho hướng dẫn này đơn giản, chúng tôi sẽ chỉ sử dụng CSS đơn giản với các tên class  rõ ràng, để các kiểu tự cung cấp nguyên tử của Tachyons và sự va chạm tên lớp không được thông qua độ dài của tên lớp. Plain  CSS có thể đủ cho các dự án có độ phức tạp thấp (ví dụ: 3 đến 6 tháng của thời gian triển khai ban đầu và một vài giai đoạn dự án).

Để thêm các style, trước tiên hãy tạo một tệp có tên là `styles.css` và include nó vào trong tệp` index.html`:
    
    
    
    
        
            
            
            
            https://mithril.js.org/styles.css" rel="stylesheet">
        
        
            ">https://mithril.js.org/bin/app.js">
        
    
    

Bây giờ chúng ta có thể style cho thành phần `UserList`:
    
    
    .user-list {list-style:none;margin:0 0 10px;padding:0;}
    .user-list-item {background:#fafafa;border:1px solid #ddd;color:#333;display:block;margin:0 0 1px;padding:8px 15px;text-decoration:none;}
    .user-list-item:hover {text-decoration:underline;}
    

CSS ở trên được viết bằng cách sử dụng quy ước giữ tất cả các kiểu cho một quy tắc trong một dòng, theo thứ tự bảng chữ cái. Quy ước này được thiết kế để tận dụng tối đa màn hình thực và dễ dàng quét các bộ chọn CSS (vì chúng luôn ở bên trái) và nhóm logic của chúng, và nó thực thi các quy tắc CSS có thể dự đoán và thống nhất cho mỗi bộ chọn .

Rõ ràng bạn có thể sử dụng bất kỳ quy ước khoảng cách / indentation nào bạn thích. Ví dụ trên chỉ là một minh họa của một quy ước không phổ biến rộng rãi có các lý do mạnh mẽ đằng sau nó, nhưng đi chệch khỏi các quy ước khoảng cách về cosmetic-oriented.

Tải lại cửa sổ trình duyệt ngay bây giờ sẽ hiển thị một số phần tử được style

* * *

Hãy thêm định tuyến vào ứng dụng của chúng ta.

Định tuyến có nghĩa là liên kết màn hình với một URL duy nhất, để tạo khả năng chuyển từ một trang "trang này sang trang khác". Mithril được thiết kế cho các ứng dụng đơn trang, vì vậy các "trang" này không nhất thiết phải là các tệp HTML khác nhau theo nghĩa truyền thống của từ đó. Thay vào đó, định tuyến trong các ứng dụng trang đơn giữ lại cùng một tệp HTML trong suốt vòng đời của nó, nhưng thay đổi trạng thái của ứng dụng thông qua Javascript. Định tuyến phía máy khách có lợi ích của việc tránh nhấp nháy màn hình trống giữa các lần chuyển trang và có thể giảm lượng dữ liệu được gửi xuống từ máy chủ khi được sử dụng kết hợp với kiến trúc hướng dịch vụ web (tức là ứng dụng tải xuống dữ liệu dưới dạng JSON thay vì tải xuống các đoạn kết xuất trước của HTML chi tiết).

Chúng ta có thể thêm định tuyến bằng cách thay đổi `m.mount` thành lệnh` m.route`:
    
    
    // src/index.js
    var m = require("mithril")
    
    var UserList = require("./views/UserList")
    
    m.route(document.body, "/list", {
        "/list": UserList
    })
    
lời gọi `m.route` chỉ định rằng ứng dụng sẽ được hiển thị thành` document.body`. Đối số `" / list "` là tuyến mặc định. Điều đó có nghĩa là người dùng sẽ được chuyển hướng đến tuyến đường đó nếu họ đến một tuyến đường không tồn tại. Đối tượng `{" / list ": UserList}` khai báo một bản đồ các tuyến hiện có và các thành phần mà mỗi tuyến đường xử lý.

Làm mới trang trong trình duyệt bây giờ sẽ thêm `#! / List` vào URL để chỉ ra rằng định tuyến đang hoạt động. Do tuyến đường đó hiển thị UserList, chúng ta vẫn sẽ thấy danh sách những người trên màn hình như trước đây.

Đoạn mã `#!` Được gọi là một hashbang, và nó là một chuỗi thường được sử dụng để thực hiện định tuyến phía client. Có thể cấu hình chuỗi này qua [`m.route.prefix`] [16]. Một số cấu hình yêu cầu hỗ trợ thay đổi phía server, vì vậy chúng ta sẽ tiếp tục sử dụng hashbang cho phần còn lại của hướng dẫn này.

* * *

Hãy thêm một tuyến đường khác vào ứng dụng của chúng ta để chỉnh sửa users. Trước tiên, hãy tạo một mô-đun gọi là `views / UserForm.js`
    
    
    // src/views/UserForm.js
    
    module.exports = {
        view: function() {
            // TODO implement view
        }
    }
    

Sau đó chúng ta có thể `require`  module mới này từ `src/index.js`
    
    
    // src/index.js
    var m = require("mithril")
    
    var UserList = require("./views/UserList")
    var UserForm = require("./views/UserForm")
    
    m.route(document.body, "/list", {
        "/list": UserList
    })
    

Và cuối cùng, chúng ta có thể tạo một tuyến đường tham chiếu đến nó :
    
    
    // src/index.js
    var m = require("mithril")
    
    var UserList = require("./views/UserList")
    var UserForm = require("./views/UserForm")
    
    m.route(document.body, "/list", {
        "/list": UserList,
        "/edit/:id": UserForm,
    })
    

Lưu ý rằng route mới có một `: id` trong nó. Đây là một tham số tuyến đường; bạn có thể nghĩ về nó như một thẻ hoang dã; route `/ edit / 1` sẽ phân giải thành` UserForm` bằng `id` của` "1" `. `/ edit / 2` cũng sẽ giải quyết thành` UserForm`, nhưng với `id` của` "2" `. Và cứ thế.

Hãy triển khai thành phần `UserForm` để nó có thể đáp ứng các tham số tuyến đường đó:
    
    
    // src/views/UserForm.js
    var m = require("mithril")
    
    module.exports = {
        view: function() {
            return m("form", [
                m("label.label", "First name"),
                m("input.input[type=text][placeholder=First name]"),
                m("label.label", "Last name"),
                m("input.input[placeholder=Last name]"),
                m("button.button[type=button]", "Save"),
            ])
        }
    }
    

Và hãy thêm một số styles vào `styles.css`:
    
    
    /* styles.css */
    body,.input,.button {font:normal 16px Verdana;margin:0;}
    
    .user-list {list-style:none;margin:0 0 10px;padding:0;}
    .user-list-item {background:#fafafa;border:1px solid #ddd;color:#333;display:block;margin:0 0 1px;padding:8px 15px;text-decoration:none;}
    .user-list-item:hover {text-decoration:underline;}
    
    .label {display:block;margin:0 0 5px;}
    .input {border:1px solid #ddd;border-radius:3px;box-sizing:border-box;display:block;margin:0 0 10px;padding:10px 15px;width:100%;}
    .button {background:#eee;border:1px solid #ddd;border-radius:3px;color:#333;display:inline-block;margin:0 0 10px;padding:10px 15px;text-decoration:none;}
    .button:hover {background:#e8e8e8;}
    

Ngay bây giờ, thành phần này không có gì để đáp lại các sự kiện của người dùng. Hãy thêm một số code vào mô hình `User` của chúng ta trong` src / models / User.js`. Đây là cách code ngay bây giờ:
    
    
    // src/models/User.js
    var m = require("mithril")
    
    var User = {
        list: [],
        loadList: function() {
            return m.request({
                method: "GET",
                url: "https://rem-rest-api.herokuapp.com/api/users",
                withCredentials: true,
            })
            .then(function(result) {
                User.list = result.data
            })
        },
    }
    
    module.exports = User
    

Hãy thêm code để cho phép chúng ta tải một người dùng
    
    
    // src/models/User.js
    var m = require("mithril")
    
    var User = {
        list: [],
        loadList: function() {
            return m.request({
                method: "GET",
                url: "https://rem-rest-api.herokuapp.com/api/users",
                withCredentials: true,
            })
            .then(function(result) {
                User.list = result.data
            })
        },
    
        current: {},
        load: function(id) {
            return m.request({
                method: "GET",
                url: "https://rem-rest-api.herokuapp.com/api/users/" + id,
                withCredentials: true,
            })
            .then(function(result) {
                User.current = result
            })
        }
    }
    
    module.exports = User
    

Lưu ý rằng chúng ta đã thêm thuộc tính `User.current` và phương thức` User.load (id) `để thêm thuộc tính đó. Bây giờ chúng ta có thể thêm vào view `UserForm` bằng cách sử dụng phương thức mới này:
    
    
    // src/views/UserForm.js
    var m = require("mithril")
    var User = require("../models/User")
    
    module.exports = {
        oninit: function(vnode) {User.load(vnode.attrs.id)},
        view: function() {
            return m("form", [
                m("label.label", "First name"),
                m("input.input[type=text][placeholder=First name]", {value: User.current.firstName}),
                m("label.label", "Last name"),
                m("input.input[placeholder=Last name]", {value: User.current.lastName}),
                m("button.button[type=button]", "Save"),
            ])
        }
    }
    

Tương tự như thành phần `UserList`,` oninit` gọi `User.load ()`. Hãy nhớ rằng chúng ta đã có một tham số tuyến đường được gọi là `: id` trên tuyến đường` "/ edit /: id": UserForm`? Tham số tuyến đường trở thành một thuộc tính của vnode của thành phần `UserForm`, vì vậy việc định tuyến tới` / edit / 1` sẽ làm cho `vnode.attrs.id` có giá trị` "1" `.

Bây giờ, hãy sửa đổi view `UserList` để chúng ta có thể điều hướng từ đó đến` UserForm`:
    
    
    // src/views/UserList.js
    var m = require("mithril")
    var User = require("../models/User")
    
    module.exports = {
        oninit: User.loadList,
        view: function() {
            return m(".user-list", User.list.map(function(user) {
                return m("a.user-list-item", {href: "/edit/" + user.id, oncreate: m.route.link}, user.firstName + " " + user.lastName)
            }))
        }
    }
    

Ở đây chúng ta đã thay đổi `.user-list-item` thành` a.user-list-item`. Chúng tôi đã thêm một `href` tham chiếu đến tuyến đường chúng ta muốn và cuối cùng chúng ta đã thêm` oncreate: m.route.link`. Điều này làm cho liên kết hoạt động giống như một liên kết định tuyến (trái ngược với việc chỉ hoạt động giống như một liên kết thông thường). Điều này có nghĩa là việc nhấp vào liên kết sẽ thay đổi một phần của URL xuất hiện sau hashbang `#!` (Do đó thay đổi tuyến đường mà không cần tải trang HTML hiện tại)

Nếu bạn làm mới trang trong trình duyệt, bây giờ bạn có thể nhấp vào một người và được đưa đến một biểu mẫu. Bạn cũng có thể nhấn nút quay lại trong trình duyệt để quay lại từ biểu mẫu tới danh sách mọi người.

* * *

Bản thân biểu mẫu vẫn không lưu khi bạn nhấn "Save". Hãy làm cho biểu mẫu này hoạt động:
    
    
    // src/views/UserForm.js
    var m = require("mithril")
    var User = require("../models/User")
    
    module.exports = {
        oninit: function(vnode) {User.load(vnode.attrs.id)},
        view: function() {
            return m("form", {
                    onsubmit: function(e) {
                        e.preventDefault()
                        User.save()
                    }
                }, [
                m("label.label", "First name"),
                m("input.input[type=text][placeholder=First name]", {
                    oninput: m.withAttr("value", function(value) {User.current.firstName = value}),
                    value: User.current.firstName
                }),
                m("label.label", "Last name"),
                m("input.input[placeholder=Last name]", {
                    oninput: m.withAttr("value", function(value) {User.current.lastName = value}),
                    value: User.current.lastName
                }),
                m("button.button[type=submit]", "Save"),
            ])
        }
    }
    

Chúng ta đã thêm các sự kiện `oninput` vào cả hai đầu vào, đặt thuộc tính` User.current.firstName` và `User.current.lastName` khi người dùng nhập.

Ngoài ra, chúng ta đã tuyên bố rằng phương thức `User.save` sẽ được gọi khi nhấn nút " Save ". Hãy thực hiện phương thức đó:
    
    
    // src/models/User.js
    var m = require("mithril")
    
    var User = {
        list: [],
        loadList: function() {
            return m.request({
                method: "GET",
                url: "https://rem-rest-api.herokuapp.com/api/users",
                withCredentials: true,
            })
            .then(function(result) {
                User.list = result.data
            })
        },
    
        current: {},
        load: function(id) {
            return m.request({
                method: "GET",
                url: "https://rem-rest-api.herokuapp.com/api/users/" + id,
                withCredentials: true,
            })
            .then(function(result) {
                User.current = result
            })
        },
    
        save: function() {
            return m.request({
                method: "PUT",
                url: "https://rem-rest-api.herokuapp.com/api/users/" + User.current.id,
                data: User.current,
                withCredentials: true,
            })
        }
    }
    
    module.exports = User
    

Trong phương thức `save` ở phía dưới, chúng ta đã sử dụng phương thức `PUT`  HTTP để chỉ ra rằng chúng ta đang upserting dữ liệu đến máy chủ.

Bây giờ hãy thử chỉnh sửa tên của người dùng trong ứng dụng. Khi bạn lưu thay đổi, bạn sẽ có thể thấy thay đổi được phản ánh trong danh sách người dùng.

* * *

Hiện tại, chúng ta chỉ có thể điều hướng trở lại danh sách người dùng thông qua nút quay lại trình duyệt. Lý tưởng nhất, chúng ta muốn có một trình đơn - hoặc tổng quát hơn, một bố cục nơi chúng ta có thể đặt các yếu tố giao diện người dùng toàn cầu

Hãy tạo một file `src/views/Layout.js`:
    
    
    // src/views/Layout.js
    var m = require("mithril")
    
    module.exports = {
        view: function(vnode) {
            return m("main.layout", [
                m("nav.menu", [
                    m("a[href='https://mithril.js.org/list']", {oncreate: m.route.link}, "Users")
                ]),
                m("section", vnode.children)
            ])
        }
    }
    

This component is fairly straightforward, it has a `
` with a link to the list of users. Similar to what we did to the `/edit` links, this link uses `m.route.link` to activate routing behavior in the link.

Chú ý cũng có một `
`phần tử có` vnode.children` là con. `vnode` là một tham chiếu đến vnode đại diện cho một cá thể của thành phần Layout (tức là vnode được trả về bởi một lời gọi` m (Layout) `). Do đó, `vnode.children` chỉ mọi trẻ em của vnode đó.

Hãy thêm một số styles:
    
    
    /* styles.css */
    body,.input,.button {font:normal 16px Verdana;margin:0;}
    
    .layout {margin:10px auto;max-width:1000px;}
    .menu {margin:0 0 30px;}
    
    .user-list {list-style:none;margin:0 0 10px;padding:0;}
    .user-list-item {background:#fafafa;border:1px solid #ddd;color:#333;display:block;margin:0 0 1px;padding:8px 15px;text-decoration:none;}
    .user-list-item:hover {text-decoration:underline;}
    
    .label {display:block;margin:0 0 5px;}
    .input {border:1px solid #ddd;border-radius:3px;box-sizing:border-box;display:block;margin:0 0 10px;padding:10px 15px;width:100%;}
    .button {background:#eee;border:1px solid #ddd;border-radius:3px;color:#333;display:inline-block;margin:0 0 10px;padding:10px 15px;text-decoration:none;}
    .button:hover {background:#e8e8e8;}
    

Hãy thay đổi bộ định tuyến trong `src / index.js` để thêm layout  của chúng ta vào mix:
    
    
    // src/index.js
    var m = require("mithril")
    
    var UserList = require("./views/UserList")
    var UserForm = require("./views/UserForm")
    var Layout = require("./views/Layout")
    
    m.route(document.body, "/list", {
        "/list": {
            render: function() {
                return m(Layout, m(UserList))
            }
        },
        "/edit/:id": {
            render: function(vnode) {
                return m(Layout, m(UserForm, vnode.attrs))
            }
        },
    })
    

Chúng ta đã thay thế mỗi thành phần bằng một [RouteResolver] [17] (về cơ bản, một đối tượng có phương thức `render`). Các phương thức `render` có thể được viết theo cách tương tự như các thành phần view thông thường, bằng cách lồng các lệnh` m () `.

Điều thú vị cần lưu ý là cách các thành phần có thể được sử dụng thay vì một chuỗi chọn trong một lời gọi `m ()`. Ở đây, trong route `/ list`, chúng ta có` m (Layout, m (UserList)) `. Điều này có nghĩa là có một vnode gốc đại diện cho một thể hiện của `Layout`, trong đó có một vnode` UserList` là con duy nhất của nó.

Trong route `/ edit /: id`, cũng có một đối số` vnode` mang các tham số route vào thành phần `UserForm`. Vì vậy, nếu URL là `/ edit / 1`, thì` vnode.attrs` trong trường hợp này là `{id: 1}`, và `m (UserForm, vnode.attrs)` này tương đương với `m (UserForm, {id: 1}) `. Mã JSX tương đương sẽ là `<UserForm id={vnode.attrs.id} />`.


License: MIT. © Leo Horie.

[1]: https://flems.io/#0=N4IgzgpgNhDGAuEAmIBcICGAHLA6AVmCADQgBmAljEagNqgB2GAthGiAKqQBOAsgPZJoBIqVj8GiSewBuGbgAIuERQF4FwADoMFuhVAph4qBbQC6xbXv38MSADKHjCsgFcGCChIAUASg1W1nrcEPCu3DrMuCEAjq4QRt5aOkGprPAAFoImmiAA4gCiACq5limp1uFQOSAZ8PBYYKgA9M0hzAC0IUYd2BS4GSr8ANau2HjizM19za48YKWBFXoA7hSZAMIhQpIUGFBNCvDc8WXLAL6+S0G4mRAM3m4e8F4P3a5Q8P7Jy9bK3LgDEYFOp3p9cEgMPAMNdrJdrucytdYOEQpITMBEdcoLYkCYnp4fBQkN9YcFQuFItEIHEEvAkmS0qEsniFLlCiUSIyglUanUGk1Wu0unTelh+oNuCMxjhcJNpuLZvNmrkFABqBTEs6-XRrTbbe4vfaHY6nbnw8o3O4PAkvHxgr4BS3Lf5y1GGkEKB3mq6WrEMa5gDAyCD49yEh6k53ksIRBRRWLxRI-HXx5nZNkgAAKHE52p1vMz-MaLTaEE63XgYolQ1G4zl-CmMzmKjAKpA6qUPDd3DR8FwWu51kh0JMrpRvcN+d+eoyW2Qhr2BxMpog07hvrh2nOIERjBYbHQ-0cRhEJBA4kkhtk8i7KhP8E9Kd0EgoDHWY+7OLsD+nMgoEArGGzyvH4TrLCEsaRN4uS4C23AdEC8ClHeAJIbgzDYI84Z2g88FRqmXoUnGzAwZgcE8IhTgdOs5YocAGQhGQNTNMg6ztp28EDkgxAKBIsAhFCobxtE-CuIggJvsMiIKFxlDcEYAByB6dqqqoalxUAYEpB6bhUlx6bo5zbruxD7qw7D-AAYvw3BRIQ56XlI8A3oo1m2cwT7XK+77OLaoEyAwggQN8rrfkg3iBcFuBQscYDcb4-rWP+gHARGYHPkEkGUvGZFkB59FDhUEhgK4ABGzAfi4OGgSF4GERUEC4FgIQhpIAAiEBkBgHz0oZDV-N2QYhn4RWpMZ0bjbxtBjbluRaWVwgLdAKG5FZFAKY+TCsLkvjrhUpG5G+WDiQODAnfAtDwAAnlgECqIgAAe8BmLQWBabAEBZFAQjcKo62bQo20QGYhWTb8PkXSYUSzgAgvU3BkXIUDxCh-k+Mj8Shd2E59rg8k6awnqYxAlz7TqJOfioPZ4wT8DKTt4MbuTQSHSAy1QICGCLVAq0gPY2lbQeu0s9YbPHadEuXe9GCfd9v2qALwLA6DJD1QNkPidDuBwwjSP7Kjavow8JPY9TuOGlzhMQMTBuk3ts3JXbVMAhbkhW-TwtM3oZOzWzZXifAEi4AH9QSFdt33aVFXrKrvG5AAysGEAi9yZj9RNO57iAwPsAL11if2DliBIzmuQo+eF15lopUB1UgRjQVCARFTZSRZGYW+XMF+JKEzd7uhs0wMgYfcrh947ehszCauZQNjFdSYADkzRIUvosQx4gmINrUriU1BgMMMk9GfHnDzLts3pxvg9kZAEYoVFQhyhkVBIGi-XWOnCImdnufoPWYuF5S7XnQAmQuTUWpdQoI9bwS8ADES9fTgP3t4JA-AUSsHdmVQQ10z6rycGDawuQCFGFyBibkaJfppVwhlWabdoKV3ErxUix4nC+E-j7BE04SFsXgM0VAxJyHqyyvcah9d0pPzqnPVuxFGEYB7vAFh3h3J2V4lImKCMwAcPNNw7cvhTLmUPCAOUYBRDAKvNIdAOCkB4LOhdYgIdA4SA0PldEQU7L7AUAARgAGxYEegoAAaioSETAADcmFuAAHM3wmAAEwAAYAkTW0N3KuwAomxIYKgbxyTAk9SDpEjAj0OhrCQJkXJiTqkBPCRNUeDBXAaCyXExJCg2kAGZ8l1O0Gk+CVFgTACQh0Iw10YCoCCgwCAxSYmtPaT47pWA7BIDfNE1AiSekMAoioAZVZaKeWAGVWWwxol7wYHieB3UrkYHCTg7g1DvEBIUGAfgBgkAKHgUgL54TxA4m4KgeBHSgXhJWWAGW11UBlRxLAYYMzsnrPmY8x64SllfNWagAAHE87xABWWpT0qxCHENwKErwJkSGmfU-pwz9moCyCGRQwACUdCJbZUlEhUDuF+ofSlvStkcw0KC8FkLoWwpaTktpbS8XIvqVLDQdyHlPJeW8j5XykC3Nsr9LodgKBzFQB02pODSlgAoAAL3RQqnZRqQWGGFVCjBYr5DwslQs2pqKVkMDWXk7F0rwnlMqXkxJABSTZTiw46EOcc05YlzkAogPGjV9yVC5KVa84kqrvmWoQiSlZeqDXIt+bZAFQKOk2rBVpCFb4eUdHtTCuFcy2neuRe69FTafG+uZaykluFyVTNDaHIOOT6UqHlVGs5FyIAYsnZOupu4LDsykjQegOcDzsEqpkbgVBzxVHYMWQUsxzonIbFMddjEqAAAFvG4Cvb45op7N2cyATdO67AHLnDMOcIAA
[2]: https://nodejs.org/en/
[3]: https://mithril.js.org/installation.html
[4]: http://rem-rest-api.herokuapp.com/
[5]: https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods
[6]: https://mithril.js.org/hyperscript.html
[7]: https://mithril.js.org/es6.html
[8]: https://mithril.js.org/jsx.html
[9]: http://eslint.org/
[10]: https://github.com/mishoo/UglifyJS2
[11]: https://github.com/gotwarlost/istanbul
[12]: https://flowtype.org/
[13]: https://mithril.js.org/lifecycle-methods.html
[14]: http://getbootstrap.com/
[15]: http://tachyons.io/
[16]: https://mithril.js.org/route.html#mrouteprefix
[17]: https://mithril.js.org/route.html#routeresolver
[18]: https://mithril.js.org/examples.html
[19]: https://gitter.im/MithrilJS/mithril.js

  
