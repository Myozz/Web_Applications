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
