# Devise 세미나 - 서동욱(2017-05-?)

### 1. 회원가입을 만들기 위해서는 어떤 기능이 필요할까?

Model : User( email, password )

View : 로그인, 로그아웃, 개인정보 수정, 이메일 인증, 확인 메시지..etc

Controller : 로그인, 로그아웃, 개인정보 수정, 이메일 인증, 확인 메시지...etc

- 이 모든걸 Devise가 만들어 준다. 

( 혼자 만들어 보고 싶다면 [https://www.codecademy.com/learn/rails-auth](https://www.codecademy.com/learn/rails-auth) 를 참고하자. )

### 2. 그래서 Devise gem을 어떻게 설치할까?

[](https://github.com/plataformatec/devise)

Gemfile.rb [ gem 추가하기 ]

```ruby
gem "devise"
```

in bash [ devise 설치 후 db 만들기 ]

	bundle
	rails generate devise:install
	rails generate devise USER
	rake db:migrate 

### 3. 로그인 / 로그아웃 / 회원가입 링크 만들기

in bash

```con
rails g controller home index
```

in config/routes.rb

```ruby
root "home#index"
```

in app/views/home/index.html.erb 

```ruby
<% if user_signed_in? %>
 <span><%= current_user.email %>님 안녕하세요</span>
 <a href="/users/sign_out" data-method="delete">로그아웃</a>
<% else %>
 <a href="/users/sign_in">로그인</a>
 <span>|</span>
 <a href="/users/sign_up">회원가입</a>
<% end %>
```

in app/views/home/index.html.erb (루비온레일즈의 뷰헬퍼를 이용해 더 이쁘게 써보기)

```ruby
<% if user_signed_in? %>
	<span><%= current_user.email %>님 안녕하세요</span>
 <%= link_to "로그아웃", destory_user_session_path, method: "delete" %>
<% else %>
	<%= link_to "로그인", new_user_session_path %>
	<span>|</span>	
	<%= link_to "회원가입", new_user_registartion_path %> 
<% end %>
```

in app/views/layout.html.erb에 body로 옮기기(어떤 액션을 가더라도 최상위에 보여주고싶음)

### 4. 특정 Action 권한 먹이기

뷰헬퍼 조금 사용하자 이미지 태그같은 친구들( 회원가입만 해야 갤러리에 들어 갈 수 있게하자 ! )

사진을 하나 보여주는 액션/뷰를 만들고 라우트를 설정하고, 메뉴에 추가하자

```ruby
def treasure
	if !user_signed_in?
 	redirect_to "users/sign_in"
	end	
end
```

코드의 활용성을 높이기 위해서!


```ruby
before_action :sign_in, only: :treasure
def treasure	
end

def sign_in
	if !user_signed_in?
 	redirect_to "users/sign_in"
	end	
end
```

devise가 제공하는 것을 쓰자!!


```ruby
before_action :authenticate_user!, only: :treasure
def treasure	
end
```

### 5. Customize view file

in bash

```ruby
rails g devise:views
```

view/devise에 여러개의 폴더가 생성된 것을 확인 할 수 있다. 몇개 수정해보자!

### 6. 이름, 주소를 회원정보에 추가로 받게 하기.

user model에 name, adress추가하기 -> 가입페이지에서 추가 input만들기 -> 컨트롤러 추가해주기

in db/migrations/xxx_devise_create_users.rb 

```ruby
t.string :name
t.string :address
```

in app/views/devise/registration/new.html.erb에 다음 코드 추가하기. 

```ruby
<div class="field"> 
 <%= f.label :name %><br /> 
 <%= f.text_field :name, autofocus: true %> 
</div> 
<div class="field"> 
 <%= f.label :address %><br /> 
 <%= f.text_field :address, autofocus: true %> 
</div> 
```

in app/controllers/application.rb (기존에 devise가 사용하는 params에 :name, :adress를 추가해준것)

```ruby
before_action :configure_permitted_parameters, if: :devise_controller?
```

```ruby
def configure_permitted_parameters
 devise_parameter_sanitizer.permit(:sign_up) do |user_params|
 user_params.permit(:email, :name, :address, :passwodr, :password_confirmation)
 end
end
```

### 7. E-mail 인증 만들기 with mailgune_rails gem

해야할것 = 레일즈로 메일을 보낼 수 있게 한다. && devise User confirmable 

[](https://github.com/jorgemanrubia/mailgun_rails)

in Gemfile.rb

```ruby
gem "mailgun_rails"
```

in config/environments/development.rb

```ruby
 config.action_mailer.delivery_method = :mailgun
 config.action_mailer.mailgun_settings = {
 api_key: "key-1bf0b913ff7f2fbb3ad7704037f240b8",
 domain: "sandbox551728c9dc5442899deb86c5dacf4f07.mailgun.org"
 }
 config.action_mailer.default_url_options = { host: '사이트주소'}
```


in db/migrations/xxx_devise_create_users.rb 주석해제

```ruby
t.string :confirmation_token
t.datetime :confirmed_at
t.datetime :confirmation_sent_at
t.string :unconfirmed_email # Only if using reconfirmable
```

in model/user.rb, confirmable 추가하기

```ruby
devise :confirmable
```
### 8. 각종 상태 메시지 띄우기

```ruby
<% flash.each do |key, value| %>
	<%= content_tag :div, value, class: "flash #{key}" %>
<% end %>
```

### 9. Sign up with facebook (아래 페이지를 따라가면 완성)

[](https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview)

### Reference

2. 4기 devise 세미나 by 동민
