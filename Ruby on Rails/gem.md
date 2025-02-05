## gem ‘letter_opener_web’

送信されるメールをブラウザで確認できるgem
メールの送信処理が実行されると、メールの内容がブラウザ上で表示され、実際にメールが送信されることなく内容を確認でき、メールの内容やフォーマットを簡単に確認・修正が可能となる

▼リンク

https://github.com/fgrehm/letter_opener_web

## gem ‘config’

Railsアプリケーションで設定管理を簡単に行うためのgem
環境ごとに異なる設定を一元管理することが可能で、「開発環境、テスト環境、本番環境」などで異なる設定をしたい場合に利用

## gem ‘enum_help’

## gem ‘ransack’

- データベースを跨ぐ検索をする際は、下記のようにモデルで他のモデル名を宣言してデータベースを指定することで可能になる。

```ruby
#models/post.rb
  def self.ransackable_associations(auth_object = nil)
    %w[comments user]
  end
```

- 検索するデータを許可する際は、下記のように対象のカラムをそれぞれのモデルで許可することで検索可能になる。

```ruby
#models/post.rb
def self.ransackable_attributes(auth_object = nil)
    %w[title body]
  end
```
