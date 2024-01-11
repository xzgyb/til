# 关于使用`through`选项的记录保存

## 对于使用`belongs_to`的`through`

```ruby
class Appointment < ApplicationRecord
  belongs_to :physician
  belongs_to :patient
end

class Patient < ApplicationRecord
  has_many :appointments
  has_many :physicians, through: :appointments
end

class Physician < ApplicationRecord
  has_many :appointments
  has_many :patients, through: :appointments
end
```
这里如果使用
```ruby
patient = Patient.create(name: "hello")
patient.physicians.create(name: "world")
```
将创建相应的`Appointment through record`
```ruby
Patient.first.physicians

[#<Physician:0x00007fc694812a88
  id: 10,
  name: "world",
  created_at: Thu, 11 Jan 2024 03:07:22.695956000 UTC +00:00,
  updated_at: Thu, 11 Jan 2024 03:07:22.695956000 UTC +00:00>]

```

如果使用
```ruby
patient = Patient.create(name: "hello")
physician = patient.physicians.build(name: "world")
physician.save
```
调用`physician.save`将不创建相应的`Appointment through record`, 只创建了一条`Physician record`．

```ruby
Patient.first.physicians
[]
```
解决方式为, 给`Appointment`的`belongs_to :physician`添加`inverse_of`选项
```ruby
class Appointment < ApplicationRecord
  belongs_to :physician, inverse_of: :appointments
  belongs_to :patient
end
```

这时候在用
```ruby
patient = Patient.create(name: "hello")
physician = patient.physicians.build(name: "world")
physician.save
```
调用`physician.save`将创建相应的`Appointment through record`

```ruby
Patient.first.physicians
[#<Physician:0x00007f31b9b55120
  id: 12,
  name: "world",
  created_at: Thu, 11 Jan 2024 03:16:49.867230000 UTC +00:00,
  updated_at: Thu, 11 Jan 2024 03:16:49.867230000 UTC +00:00>]

```

## 对于使用`has_many`的嵌套`through`

```ruby
class Author < ActiveRecord::Base
  has_many :posts
  has_many :comments, through: :posts
  has_many :commenters, through: :comments
end

class Post < ActiveRecord::Base
  has_many :comments
  belongs_to :author
end

class Commenter < ActiveRecord::Base
end

class Comment < ActiveRecord::Base
  belongs_to :commenter
  belongs_to :post
end

author = Author.create(name: "hello")
author.commenters.create(title: "world")
```

对于嵌套的`through`, 不能修改相应的`association`,上面的代码执行时会出现异常

```ruby
.gem/ruby/3.3.0/gems/activerecord-7.1.2/lib/active_record/associations/through_association.rb:111:in `ensure_not_nested': Cannot modify association 'Author#commenters' because it goes through more than one other association. (ActiveRecord::HasManyThroughNestedAssociationsAreReadonly)
```

具体参见`ActiveRecord::Associations`的`Nested Associations`．




