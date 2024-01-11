# 关于`inverse_of`选项的用法

## 当使用了`:foreign_key`选项时, 需要使用`:inverse_of`

```ruby
class Author < ApplicationRecord
  has_many :books, inverse_of: 'writer'
end

class Book < ApplicationRecord
  belongs_to :writer, class_name: 'Author', foreign_key: 'author_id'
end

```
参见 `Rails Guide` 的 `Active Record Associations` 一章中的 `3.5 Bi-directional Associations`

## 使用了`:through`时，需要使用`:inverse_of`

```ruby
class Tagging < ActiveRecord::Base
  belongs_to :post
  belongs_to :tag, inverse_of: :taggings
end

class Post < ActiveRecord::Base
  has_many :taggings
  has_many :tags, through: :taggings
end

class Tag < ActiveRecord::Base
  has_many :taggings
end

@post = Post.first
@tag = @post.tags.build name: "ruby"
@tag.save
```

只有在`Tagging`中使用了`inverse_of`选项时，当`@tag.save`保存后，`Tagging`才会保存一条相应的`through record`.  
具体参见 `ActiveRecord::Associations` 的 `Setting Inverses`的文档.