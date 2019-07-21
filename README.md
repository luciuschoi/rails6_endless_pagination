# Endless Pagination Using Stimulus.js

**레일스 (~> 5.x)** 에서 **jquery.pageless plugin**을 사용하면 **Endless(Pageless or Infinite)** 페이지 스크롤 기능을 쉽게 구현할 수 있었다.

더우기, **pageless-rails** 젬을 사용하면 **jquery.pagelss.js** 파일을 **asset pipeline**으로 쉽게 importing할 수 있다.

> Gem homepage: https://github.com/luciuschoi/pageless-rails
>
> Plugin homepage: https://github.com/jney/jquery.pageless

이 글에서는 **레일스 6**에서 위의 젬 도움 없이 **Stimulus.js** 를 사용하여 구현하는 방법을 알아 볼 것이다.

Endless pagination을 구현하기 위해서는, **jquery.pageless.js** 를 사용할 때처럼, 먼저 **will_paginate** 젬을 이용하여 **pagination**을 구현해 놓아야 한다.

> **Note** : **Bootstrap** 설치하는 방법은 [Rails 6를 영접하자](https://github.com/luciuschoi/welcome_rails6/tree/master) 참고하기 바란다.

이 글에서 예시한 샘플 프로젝트의 소스는 https://github.com/luciuschoi/rails6_endless_pagination 으로 접속하면 볼 수 있다.

## 0. 준비작업

먼저 샘플 프로젝트를 생성한다. 이 때 **--webpack=stimulus** 옵션을 주목하기 바란다.

```bash
$ rails -v
Rails 6.0.0.rc1
$ rails new blog6 --webpack=stimulus
$ cd blog6
```

그리고 **posts** 리소스를 다음과 같이 **scaffold** 제너레이터를 이용하여 생성하고,

```bash
$ bin/rails g scaffold Post content:text
```

데이터베이스와 테이블을 생성한다.

```bash
$ bin/rails db:create
$ bin/rails db:migrate
```

**config/routes.rb** 파일을 열고 다음과 같이 루트 라우트를 업데이트한다.

```ruby
Rails.application.routes.draw do
  root "posts#index"
  resources :posts
end
```

##1. **Gemfile**

위에서 언급한 바와 같이 레일스 프로젝트에서 데이터베이스 테이블 레코드를 페이지 단위로 보여 주기 위해서 그 동안 pagination 젬들을 사용하여 쉽게 구현할 수 있었다. 그 중에서도 이 글에서는 will_paginate 젬을 설치하여 사용할 것이다.

이를 위해서 Gemfile을 열고 아래와 같이 젬을 추가하고,

```ruby
gem 'will_paginate'
```

번들 인스톨 명령을 실행한다.

```bash
$ bundle install
```

##2. index 뷰파일의 refactoring

**app/views/posts/index.html.erb**

```erb
<p id="notice"><%= notice %></p>

<h1>Posts</h1>

<div data-controller='endless'>
  <table>
    <thead>
      <tr>
        <th>Content</th>
        <th colspan="3"></th>
      </tr>
    </thead>

    <tbody id="posts">
      <%= render @posts %>
    </tbody>
  </table>

  <button id='gotop' data-action='click->endless#gotop'>go to Top</button>

  <%= will_paginate @posts %>
</div>
<br>

<%= link_to 'New Post', new_post_path %>
```

뷰 파셜 템플릿 파일의 생성 (**\_post.html.erb**)

```erb
<tr>
  <td><%= post.content %></td>
  <td><%= link_to 'Show', post %></td>
  <td><%= link_to 'Edit', edit_post_path(post) %></td>
  <td><%= link_to 'Destroy', post, method: :delete, data: { confirm: 'Are you sure?' } %></td>
</tr>
```

##3. Pagination 적용하기

**app/controllers/posts_controller.rb**

```ruby
class PostsController < ApplicationController

  ···

  def index
    @posts = Post.all.paginate(page: params[:page], per_page: 10)
  end

  ···

end
```

##4. Ajax 콜

posts#index 액션을 remote 호출시마다 새로 렌더링된 다음 페이지 분량의 posts 들을 추가하고 pagination 부분의 갱신한다.

**app/views/posts/index.js.erb**

```erb
$('#posts').append('<div>[<%= j @posts.current_page %> 페이지] -------------> </div>');
$('#posts').append('<%= j render(@posts) %>');
console.log("rendered page <%= @posts.current_page %>");

<% if @posts.next_page %>
  $('.pagination').replaceWith('<%= j will_paginate(@posts) %>');
  $(".pagination").hide();
<% else %>
  $("#gotop").show();
  $('.pagination').remove();
<% end %>
```

##5. Endless 컨트롤러 작성

**stimulus.js** 동작을 위해서 **data-controller**로 지정한 **endless** 컨트롤러를 작성한다.

**app/javascript/controllers/endless_controller.js**

```js
import { Controller } from 'stimulus';

export default class extends Controller {
  static targets = [];

  connect() {
    $('.pagination').hide();
    $('#gotop').hide();
    if ($('.pagination').length && $(this).length) {
      $(window).scroll(function() {
        let url = $('.pagination .next_page').attr('href');
        if (
          url &&
          $(window).scrollTop() > $(document).height() - $(window).height() - 50
        ) {
          $('.pagination').text('Loading more posts...');
          return $.cachedScript(url);
        }
      });
      return $(window).scroll();
    }
  }

  gotop() {
    $('html, body').animate(
      {
        scrollTop: 0
      },
      200
    );
  }
}
```

gotop() 함수는 페이지 스크롤이 마지막 페이지에 도달했을 때 하단에 보이는 "go to Top" 버튼에 연결되는 클릭이벤트 핸들러이다.
