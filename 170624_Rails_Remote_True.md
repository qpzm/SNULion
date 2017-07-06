# 170624 RAILS 내장 AJAX 기능 사용하기



## 들어가기

Ajax 요청은 서버로 부터 응답이 오기까지 기다리지 않아도 된다는 장점(a.k.a 비동기 방식)이 있습니다. 다만 긴 javascript code를 작성해야 하는 불편함이 있었는데요, 사실, Rails에서는 `remote:true` 옵션을 통해 ajax요청을 쉽게 보내고 응답할 수 있는 방법이 있습니다. 

[실습링크](https://github.com/sdu6342/partialview_seminar)



## AJAX 요청 보내기

rails에선  `link_to`, `form_for` helper등에 `remote: true`옵션을 이용하여 ajax요청을 보낼 수 있습니다.

> ##### app/views/home/index.html.erb

```html
<!-- link_to 를 이용한 요청 예시-->
<%= link_to '요청!', ajax_call_path, remote: true %>

<!-- form_for 를 이용한 요청 예시-->
<%= form_for @model, remote: true do |f| %>
...
<% end %>

<!-- form_tag 를 이용한 요청  이번 실습-->
<%= form_tag search_path, remote: true do %>
	<%= text_field_tag :word %>
<% end %>
```

위와 같이 `remote: true`옵션을 추가하면 해당 태그들이 ajax를 보내게 됩니다.



## AJAX 응답 step1 Controller

ajax요청 역시 remote:true가 없는 일반 HTTP request와 동일하게 컨트롤러의 액션으로 연결됩니다. 따라서 기존에 액션을 만들던 대로 **똑같이** 만들면 됩니다.  

> app/controllers/home_controller.rb

```ruby
class HomeController < ApplicationController
  def search
    @user = User.search(params[:word])
  end
end
```



## AJAX 응답 step2 js.erb

remote:true 옵션이 없는 링크를 통해 일반 HTTP request를 보낼경우 `search.html.erb`파일이 실행되며 페이지가 새로고침됩니다.

반면, Ajax 요청일 경우 `search.js.erb`파일로 연결됩니다.

위 `search`액션에 Ajax 요청이 들어오므로 rails는 기본적으로 `app/views/home/search.js.erb`파일을 실행합니다.

> app/views/home/search.js.erb

```javascript
$("#footer").text('총 <%= @users.count %>명이 검색되었습니다');
```

1. html뷰파일에서 써왔듯이  똑같이 js뷰파일에서도 `<% %>, <%= %>`를 이용하여 ruby코드를 끼워넣을 수 있습니다.  

2. 컨트롤러에서 만든 인스턴스 변수(@붙은 변수)를 사용할 수 있는 것도 동일합니다.  

3. **ruby 코드가 text형태로 삽입되어 뷰파일이 만들어지고 실행됨을 명심합시다.**  

   위 `search.js.erb`의 코드는 `$('#footer').text("총 3명이 검색되었습니다.");`으로 변환되어 실행될 것입니다.
   ​

## AJAX with partial view

1. javascript로 작성한 ajax요청

```javascript
$('#target').append("<table><thead><tr><th>Name</th><th>Age</th><th>City</th></tr></thead><tbody><tr class="user" id="user_1"><td>구자운</td>
(중략)
<tr class="user" id="user_49"><td>조현</td></tr></tbody></table>");
```

2. `remote: true`를 이용한 ajax 요청

언더바(_)로 시작하는 partial view 파일을 사용해서 코드를 분리할 수 있다.

- 요청에 따라 바뀌길 원하는 부분을 partial view로 만듭니다. 이때 partial view안에 변수가 들어가는 부분이 있다면, `render partial`의 `locals`옵션을 사용하여 변수를 넘겨주도록 합니다.  
- `locals: {key: value}` 와 같이 쓰면 partial view 파일에서는 key값으로 value에 접근 가능하다.

> app/views/home/_list.html.erb

```html
<%= table_for users do -%>
  <% columns :name, :age, :city %>
<% end %>
```

> app/views/home/index.html.erb

```html
<div id="list">
  <%= render partial: "list", locals: {users: @users}%>
</div>
```

- `div#user_info`안의 내용은 모두 partial view로 분리되어 있으므로 javascript의 `.html()`함수를 이용해서 partial view를 갈아끼워주면 끝. 

- `.html(" ")` 내부에 따옴표를 빼면 명확한 에러 없이 동작하지 않으니 주의!

- 아래의 코드는 정상적으로 동작할까?

  ```javascript
  // Without escaping, you get a broken javascript string at href
  $("#reviews").append("<a href="/mycontroller/myaction">Action!</a>");
  ```

  - ruby코드에서 출력한 값이 js파일에서 syntax error를 내는 것을 방지하기 위해 줄바꿈을 포함한 많은 내용을 출력할 때에는 `escape_javascript`함수( 줄여서 `j`라고씀)를 써야한다. [참조링크](https://stackoverflow.com/questions/1620113/why-escape-javascript-before-rendering-a-partial)

> app/views/home/search.js.erb

```javascript
$("#list").html('<%= escape_javascript(render partial: 'list', locals: {users: @users}) %>');

//위와 동일
$("#list").html('<%=j render partial: 'list', locals: {users: @users} %>');
```



### 실습 동전줍기 게임 만들기

[GitHub 링크](https://github.com/ucheol2/snulion_beggar)





---

### Contributer

정유철, 이현민