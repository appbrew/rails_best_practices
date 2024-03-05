# AppBrew Rails Best Practices

持続可能なRailsアプリを目指して。

## 一般

### Rails Guidesに従う

基本的に、Ruby on Rails Guides ([En](https://guides.rubyonrails.org/) / [Jp](https://railsguides.jp/)) に従ってください。

### Rubyの書き方に従う

基本的に、普通のRubyのライブラリやアプリケーションを書くときの作法に従ってください。

### RuboCopに従う

基本的に、RuboCopのルールに従ってください。

RuboCopで表現可能なルールについては、このガイドラインでは説明しておらず、RuboCopの設定やカスタムCopとして表現されています。

### 誰が書いてもそうなるコードを目指す

あるべきところにあるべきコードが記述されているような、驚きの少ないコードを目指してください。

### 開発環境構築手順を保守する

READMEを見るだけで開発環境を構築できるようにしてください。

### ログを確認する

リクエストの処理中に不要なSQL等が発行されていないか、常に確認してください。

### YARDで型を書く

メソッドが引数または返り値を持つ場合、期待する型をYARDで記述してください。

悪い例:

```ruby
def call(foo:)
  foo.bar(baz)
end
```

良い例:

```ruby
# @param [Foo] foo
# @return [String]
def call(foo:)
  foo.bar(baz)
end
```

## モデル

### app/modelsにドメインモデルを置く

ファイルパスまたは命名規則がRailsに規定されていない限り、ドメインモデルはすべて `app/models` に置いてください。

### 適切にValidationを使う

ユーザー体験を向上させるためにも、ActiveModelやActiveRecordのモデルでは適切にValidationを利用してください。

## コントローラー

### RESTfulな名前を使う

RESTfulなエンドポイント、コントローラー名、アクション名を使ってください。

悪い例:

```ruby
get '/user_status/:user_id', to: 'users#status'
```

良い例:

```ruby
get '/users/:user_id/status', to: 'user_statuses#show'
```

### 不要なルーティングを定義しない

不要なルーティングパターンや不要なURLヘルパーを定義しないでください。

悪い例:

```ruby
resources :articles

get '/foos/:foo_id/bar'
```

良い例:

```ruby
resources :articles, only: %i[index show]

get '/foos/:foo_id/bar', as: nil
```

### 異常系にはbefore_actionを使う

アクションの主な処理より前にレスポンスを返すときは、before_actionを使ってください。

悪い例:

```ruby
def index
  if signed_in?
    redirect_to sign_in_path
    return
  end

  # ...
end
```

良い例:

```ruby
before_action :require_signed_in

def index
  # ...
end

private

def require_signed_in
  if signed_in?
    redirect_to sign_in_path
  end
end
```

### 適切なステータスコードを返す

利用するステータスコードの種類は、アプリや機能単位で統一してください。

悪い例:

```ruby
if @article.update(attributes)
  redirect_to @article
else
  render 'update_error'
end
```

良い例:

```ruby
if @article.update(attributes)
  redirect_to @article
else
  render 'update_error', status: 422
end
```

### paramsを引数に渡さない

`ActionController::Parameters` のインスタンスをメソッドの引数に渡さないでください。

悪い例:

```ruby
Foo.call(foo_params)
```

良い例:

```ruby
Foo.call({ a: foo_params[:a], b: foo_params[:b] })
```

良い例:

```ruby
Foo.call(foo_params.to_h)
```

## RSpec

### Example Groupのネストを避ける

`describe` や `context` のネストを浅く保ってください。

悪い例:

```ruby
context 'Aのとき' do
  # ...
end

context 'Aでないとき' do
  context 'Bのとき' do
    # ...
  end
end
```

良い例:

```ruby
context 'Aのとき' do
  # ...
end

context 'Bのとき' do
  # ...
end
```

### 文法規則に従ったテストメッセージを書く

その自然言語の文法規則に従い、適切な形式でテストメッセージを記述してください。

悪い例:

```ruby
it 'AがBになること' do
  # ...
end
```

良い例:

```ruby
it 'AをBにする' do
  # ...
end
```

### テストメッセージとコードを対応させる

`describe` とその直下の `subject`、また `context` とその直下の `before` とを対応させてください。

悪い例:

```ruby
context 'when article is unpublished' do
  it '...' do
    article.published_at = nil
    # ...
  end
end
```

良い例:

```ruby
context 'when article is unpublished' do
  before do
    article.published_at = nil
  end

  # ...
end
```

### アクション単位でrequest-specのファイルを分ける

1つのアクションに対して、1つのrequest-specのテストファイルを用意してください。

悪い例:

```ruby
# spec/requests/articles_spec.rb
RSpec.describe 'Articles' do
  describe 'GET /articles' do
    # ...
  end

  describe 'GET /articles/:id' do
    # ...
  end
end
```

良い例:

```ruby
# spec/requests/articles_index_spec.rb
RSpec.describe 'GET /articles' do
  # ...
end

# spec/requests/articles_show_spec.rb
RSpec.describe 'GET /articles/:id' do
  # ...
end
```

### match matcherを使う

JSON形式のレスポンスボディ等を検査する場合、[match matcher](https://relishapp.com/rspec/rspec-expectations/docs/built-in-matchers/match-matcher)を利用してください。

悪い例:

```ruby
expect(response.parsed_body['comments']['id']).to eq(comment.id)
```

良い例:

```ruby
expect(response.parsed_body).to match(
  'comment' => hash_including(
    'id' => comment.id
  )
)
```

### 複数件のレコードを用意する

複数件のレコードが利用されるテストを記述するとき、事前条件として2件以上のレコードをつくってください。

悪い例:

```ruby
before do
  FactoryBot.create(:article)
end
```

良い例:

```ruby
before do
  FactoryBot.create_list(:article, 2)
end
```

### controller-specを避ける

基本的に、controller-specを避け、request-specを使ってください。

悪い例:

```ruby
# spec/controllers/articles_controller_spec.rb
RSpec.describe ArticlesController do
  describe '#index' do
    # ...
  end
end
```

良い例:

```ruby
# spec/requests/articles_index_spec.rb
RSpec.describe 'GET /articles' do
  # ...
end
```

### system-specを書きすぎない

高価なsystem-specで保証する必要がある箇所にのみ、system-specを利用してください。

### 宣言の有無をテストしない

何かを宣言するコードについて、それを宣言しているかどうかを検査するだけのテストを記述しないでください。

悪い例:

```ruby
class Article < ApplicationRecord
  validates :title, presence: true
end

RSpec.describe Article do
  it do
    is_expected.to validate_presence_of(:title)
  end
end
```
