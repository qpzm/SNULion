# Partial View Seminar - 서동욱 (2017-06-24)

### 1. Partial view의 장점

* 용이한view 관리 = **div**의늪에서 벗어나기
* view재활용성증가
* 쉬운 ajax 사용

### 2. Render Partial 사용하기

다음과 같이 index_html.erb를 매우 매우 매우 보기 좋게 정리할 수 있다. **div**의 늪에서 벗어날 수 있다!

partial view는 이름에 항상 _를 항상 붙여주어야 한다.(레일즈의 규칙, COC를 찾아볼것 )

"index.html.erb"

```ruby
<div class="container">
  <%= render partial: "search_bar" %>
  <%= render partial: "list" %>
  <%= render partial: "footer" %>
</div>
```

> index를 여러개의 partial view로 간단하게 표현 할 수 있다! 

"_search_bar.html.erb"

```html
<div> I'm so Strong search bar </div>
</br> 
```

"_list.html.erb"

```html
<div> 내가 가장 강력한 리스트지 </div>
</br>
```

"_footer.html.erb"

```html
<div> 난 footer라고 해 </div>
</br>
```

### 2. Search bar 만들기 ( form_tag 미리 연습하기 )

"_search_bar.html.erb"

~~~ruby
<%= form_tag "/home/search", method: "get" do %>
  <%= text_field_tag "word" %>
  <%= submit_tag "Strong Serach" %>
<% end %>
~~~



### 3.  Render Partial Locals 사용하기 (_list가 @user를 받게 하기)

1에서 만든 Render partial과의 차이점은 데이터를 로컬로 던질 수 있다는 점이다. 즉 Partial view안에서 컨트롤러가 던진 @로 시작하는 변수를 받을 수 있다. Locals를 사용하면 된다!

"controller/home_controller.erb"

```ruby
def index
   @user = @user.all
end
```

"home/index.html"

```ruby
<div class="container">
  <%= render partial: "search_bar" %>
  <%= render partial: "list", locals: {users: @users}%>
  <%= render partial: "footer" %>
</div>
```

> controller에서 던진 @users를 list에 users로 보내준다.

"home/_list.html.erb"

```ruby
<% users.each do |u| %>
  <%= u.name %> | <%= u.age %> | <%= u.city %>
  </br>
<% end %>
```

> index에서 users로 던진 데이터를 받아서 처리한다.



### 4. Bonus! table_for_collection gem 사용하기

https://github.com/lunich/table_for

"Gemfile"

```ruby
gem 'table_for_collection'
```

In Bash

```
bundle
```

"home/_list.html.erb"

```ruby
<%= table_for users do -%>
  <% columns :name, :age, :city %>
<% end %>
```