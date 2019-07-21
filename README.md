# Endless Pagination Using Stimulus.js

**레일스 (~> 5.x)** 에서 **jquery.pageless plugin**을 사용하면 **Endless(Pageless or Infinite)** 페이지 스크롤 기능을 쉽게 구현할 수 있었다.

더우기, **pageless-rails** 젬을 사용하면 **jquery.pagelss.js** 파일을 **asset pipeline**으로 쉽게 importing할 수 있다.

> Gem homepage: https://github.com/luciuschoi/pageless-rails
>
> Plugin homepage: https://github.com/jney/jquery.pageless

이 글에서는 **레일스 6**에서 위의 젬 도움 없이 **Stimulus.js** 를 사용하여 구현하는 방법을 알아 볼 것이다.

**jquery.pageless.js** 를 사용할 때처럼 먼저 **will_paginate** 젬을 이용하여 먼저 **pagination**을 구현해 놓아야 한다.

> **Note** : Bootstrap 설치하는 방법은 [Rails 6를 영접하자](https://github.com/luciuschoi/welcome_rails6/tree/master) 참고하기 바란다.

**Gemfile**

```ruby
gem 'will_paginate'
```

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

**app/views/posts/index.html.erb**

```erb
<div id='posts' data-controller='endless'>
  <%= render @posts %>
</div>

<button id='gotop' data-action='click->endless#gotop'>go to Top</button>
<%= will_paginate @posts %>
```

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
