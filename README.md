# has_and_belongs_to_many: A has_and_belongs_to_many association creates a direct many-to-many connection with another model.

* Generate Model as well as migration
```
rails generate model Book title:string
rails generate model Author name:string
rails generate migration CreateJoinTableAuthorsBooks author book

rails db:migrate
```

* Table Structure
```
class CreateBooks < ActiveRecord::Migration[7.1]
  def change
    create_table :books do |t|
      t.string :title

      t.timestamps
    end
  end
end

class CreateAuthors < ActiveRecord::Migration[7.1]
  def change
    create_table :authors do |t|
      t.string :name

      t.timestamps
    end
  end
end

class CreateJoinTableAuthorsBooks < ActiveRecord::Migration[7.1]
  def change
    create_join_table :authors, :books do |t|
      t.index [:author_id, :book_id]
      t.index [:book_id, :author_id]
    end
  end
end
```

* Association in between model
```
class Book < ApplicationRecord
  has_and_belongs_to_many :authors
end

class Author < ApplicationRecord
  has_and_belongs_to_many :books
end
```

* ORM Query for create and fetch object
```
author1 = Author.create(name: "Author 1")
author2 = Author.create(name: "Author 2")

book1 = Book.create(title: "Book 1")
book2 = Book.create(title: "Book 2")
book3 = Book.create(title: "Book 3")
```

* Update join table
```
book1.authors.push(author1, author2)
book1.authors.push(author1)
author1.books.push(book3)
```

* Fatch the Object
```
author1.books
author2.books

book1.authors
book2.authors
book3.authors
```


# has_many :through: A has_many :through association creates a indirect many-to-many connection with another model

* Generate Model as well as migration
```
rails generate model User name:string
rails generate model Role title:string
rails generate model Assignment user:references role:references start_date:date

rails db:migrate
```

* Table structure
```
class CreateUsers < ActiveRecord::Migration[7.1]
  def change
    create_table :users do |t|
      t.string :name

      t.timestamps
    end
  end
end

class CreateRoles < ActiveRecord::Migration[7.1]
  def change
    create_table :roles do |t|
      t.string :title

      t.timestamps
    end
  end
end

class CreateAssignments < ActiveRecord::Migration[7.1]
  def change
    create_table :assignments do |t|
      t.references :user, null: false, foreign_key: true
      t.references :role, null: false, foreign_key: true
      t.date :start_date

      t.timestamps
    end
  end
end

```

* Associations in between model
```
class User < ApplicationRecord
  has_many :assignments
  has_many :roles, through: :assignments
end

class Role < ApplicationRecord
  has_many :assignments
  has_many :users, through: :assignments
end

class Assignment < ApplicationRecord
  belongs_to :user
  belongs_to :role
end
```

* ORM Query for create and fetch object
```
user1 = User.create(name: "Alice")
user2 = User.create(name: "Bob")

role1 = Role.create(title: "Admin")
role2 = Role.create(title: "Manager")
```

* Update relationship table
```
Assignment.create(user: user1, role: role1, start_date: Date.today)
Assignment.create(user: user1, role: role2, start_date: Date.today)
Assignment.create(user: user2, role: role2, start_date: Date.today)
```

* Fatch the Object
```
# Fetch all roles of a user
user1.roles

# Fetch all users of a role
role2.users
```

* if you create new users as well as roles need to update Assignments for the same
```
role3 = Role.create(title: "Student")
user3 = User.create(name: "John")

user3.roles  # return []
role3.users  # return []

Assignment.create(user: user3, role: role3, start_date: Date.today)
# Fetch all roles of a user3
user3.roles
```

* Fetch object with Assignment
```
assignment = Assignment.first
assignment.user
assignment.role
```

* we can directly fetch user's assignment and roles assignment
```
user3.assignments
role3.assignments
```



# Polymorphic Associations: A model can belong to more than one other model

* Generate Model as well as migration
```
rails generate model Post title:string
rails generate model Article description:string
rails generate model Comment content:text commentable:references{polymorphic}

rails db:migrate
```

* Table Structure:
```
class CreatePosts < ActiveRecord::Migration[7.1]
  def change
    create_table :posts do |t|
      t.string :title

      t.timestamps
    end
  end
end

class CreateArticles < ActiveRecord::Migration[7.1]
  def change
    create_table :articles do |t|
      t.string :description

      t.timestamps
    end
  end
end

class CreateComments < ActiveRecord::Migration[7.1]
  def change
    create_table :comments do |t|
      t.text :content
      t.references :commentable, polymorphic: true, null: false

      t.timestamps
    end
  end
end
```

* Associations in between model:
```
class Post < ApplicationRecord
  has_many :comments, as: :commentable
end

class Article < ApplicationRecord
  has_many :comments, as: :commentable
end

class Comment < ApplicationRecord
  belongs_to :commentable, polymorphic: true
end
```

* ORM Query for create and fetch object
```
post = Post.create(title: "My Post")
article = Article.create(description: "My Article")

post.comments   #  []
article.comments # []


post.comments.create(content: "Post Comment")
article.comments.create(content: "Article Comment")


post.comments # fetch all the post comments
article.comments # fetch all the article comments
```


# Other Example of has_many :through 
```
class Document < ApplicationRecord
  has_many :sections
  has_many :paragraphs, through: :sections
end

class Section < ApplicationRecord
  belongs_to :document
  has_many :paragraphs
end

class Paragraph < ApplicationRecord
  belongs_to :section
end


document = Document.find(id)
document.paragraphs
document.sections


section = Section.find(id)
section.document
section.paragraphs

paragraph = Paragraph.find(id)
paragraph.section
```


# Choosing Between has_many :through and has_and_belongs_to_many

```
set up a has_many :through relationship if you need to work with the relationship model as an independent entity. If you don't need to do anything with the relationship model, it may be simpler to set up a has_and_belongs_to_many relationship.

You should use has_many :through if you need validations, callbacks, or extra attributes on the join model.
```


# Person model so that any Person can be assigned as the parent of another Personn  What columns would you need to define in the migration creating the table for Person?

```
For example:
sally = Person.create(name: "Sally")
sue = Person.create(name: "Sue", parent: sally)
```
```
class Person < ApplicationRecord
  belongs_to :parent, class Person
  has_many :children, class Person, foreign_key :parent_id
end
```
