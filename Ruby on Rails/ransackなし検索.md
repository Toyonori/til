●概要
Ransackを利用せず、scopeとformobjectで「記事タイトル」、「カテゴリ検索」に追加して「著者」「タグ」のセレクトボックスでの検索機能、および「記事本文」の検索機能の実装（本文検索は、記事の下書き、公開状態に関わらず検索可能とする）

●詳細

- article_controllerにて
search_paramsにauthor_id(著者),tag_id(タグ),body(本文)の検索許可を追加

```
def search_params
    params[:q]&.permit(:title, :category_id, :author_id, :tag_id, :body)
  end
```

- formsフォルダにて
著者、タグ、本文の検索条件を追加。

```ruby
attribute :category_id, :integer #integerは属性を整数型として定義
attribute :title, :string #stringは属性を文字列として定義
attribute :author_id, :integer
attribute :tag_id, :integer
attribute :body, :string

  def search
    articles = Article.all.with_drafts #with_draftでmodelで定義したscopeを呼び出し、すべての記事を呼び出す役割。

    articles = articles.by_category(category_id) if category_id.present?
    title_words.each do |word|
      articles = articles.title_contain(word)
    end
    articles = articles.by_author(author_id) if author_id.present?　#by_authorは著者IDによる完全一致検索
    articles = articles.by_tag(tag_id) if tag_id.present?
    articles = articles.body_contain(body) if body.present?　#body_containはタイトルに含まれるモンゴの部分一致検索

    articles.presence&.distinct || Article.none　#下記に詳細記載
  end

  private

#titleは'Ruby on Rails'などを'Ruby','on','Rails'などと空白で区切って検索性を向上させる。本文は長いのでこれをしない
  def title_words
    title.present? ? title.split(nil) : []
  end
```

articles.presence&.distinct || Article.none:
articles.presence: articles が空でない場合は articles を返し、空の場合は nil を返します。これは、articles が空の場合に distinct を呼び出すとエラーになるのを防ぐためです。
&.distinct: articles が nil でない場合（つまり、記事が存在する場合）、重複する記事を削除します。
|| Article.none: articles が nil の場合（つまり、検索結果が0件の場合）、空のActiveRecord::Relationを返します。これにより、ビューで articles を扱う際にエラーが発生するのを防ぎます。

- model article.rbにて
下記の著者、タグ、本文に関するscopeを追記。またwith_draftsで全体検索のための条件も定義する。
ここでscopeの後の記載が、他の場所で呼び出すためのメソッド定義のようなものになります。

```ruby
scope :by_author, ->(author_id) { where(author_id: author_id) }
scope :by_tag, ->(tag_id) { joins(:tags).where(tags: { id: tag_id }) }
scope :body_contain, ->(body) { joins(:sentences).where('sentences.body LIKE ?', "%#{body}%").distinct }
scope :with_drafts, -> { unscoped }

#scope :[メソッド名], ->([引数]) {実行するクエリ}
```

- Viewフォルダにて
下記のような検索条件を追記

```
= form_with model: @search_articles_form, scope: :q, url: admin_articles_path, method: :get, html: { class: 'form-inline' } do |f|
            => f.select :category_id, Category.pluck(:name, :id) , { include_blank: 'カテゴリ' }, class: 'form-control'
            => f.select :author_id, Author.pluck(:name, :id) , { include_blank: '著者' }, class: 'form-control'
            => f.select :tag_id, Tag.pluck(:name, :id), { include_blank: 'タグ' }, class: 'form-control'
            => f.search_field :body, placeholder: '記事内容', class: 'form-control'
            .input-group
              = f.search_field :title, placeholder: 'タイトル', class: 'form-control'
              span.input-group-btn
                = f.submit '検索', class: %w[btn btn-default btn-flat]
```

Tag.pluck(:name, :id): これが今回の質問の中心となる部分です。pluck メソッドは、Active Recordのメソッドで、指定されたカラムの値を配列として取得します。

Tag: これは Tag モデルを表しています。

pluck(:name, :id): Tag モデルの name カラムと id カラムの値をデータベースから取得します。結果として、[["タグ名1", ID1], ["タグ名2", ID2], ...] のような二次元配列が返されます。この配列は、selectタグのオプションとして使用されます。

例：Tag テーブルに以下のデータがあるとします。

| id | name |
| --- | --- |
| 1 | Ruby |
| 2 | Rails |
| 3 | WebDev |

この場合、Tag.pluck(:name, :id) は [["Ruby", 1], ["Rails", 2], ["WebDev", 3]] を返します。

`pluck` メソッドを使用する主な利点は、データベースへのクエリを最適化できることです。通常の `Tag.all.map { |tag| [tag.name, tag.id] }` のように記述する場合、すべての `Tag` オブジェクトがメモリにロードされます。一方、`pluck` を使用すると、必要なカラムの値だけが直接データベースから取得されるため、メモリの使用量を抑え、パフォーマンスを向上させることができます。特に、データ量が多い場合に効果を発揮します。
