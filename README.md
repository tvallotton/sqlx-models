# Models
Models is a SQL migration management tool. It supports PostgreSQL, MySQL, and SQLite.


# Quick Start

install the CLI by running the following command: 
```
$ cargo install models-cli
```

Now run the following command to create an environment file with the `DATABASE_URL` variable set: 
```
$ echo "DATABASE_URL=sqlite://database.db" > .env
```
Alternatively it can be set as a environment variable with the following command: 
```
$ export DATABASE_URL=sqlite://database.db
```
We now can create the database running the following command: 
```
$ models database create
```
This command will have created an SQLite file called `database.db`. 
You can now derive the `Model` trait on your structures, 
and `models` will manage the migrations for you. For example, write at `src/main.rs`: 
```rust
#![allow(dead_code)]
use models::Model; 

#[derive(Model)]
struct Profile {
    #[primary_key]
    id: i32,
    #[unique]
    email: String,
    password: String,
    is_admin: bool,
}

#[derive(Model)]
struct Post {
    #[primary_key]
    id: i32,
    #[foreign_key(Profile.id)]
    author: i32,
    #[default("<Untitled Post>")]
    title: String,
    content: String,
}

#[derive(Model)]
struct PostLike {
    #[foreign_key(Profile.id, on_delete="cascade")]
    #[primary_key(post_id)]
    profile_id: i32,
    #[foreign_key(Post.id, on_delete="cascade")]
    post_id: i32,
}

#[derive(Model)]
struct CommentLike {
    #[foreign_key(Profile.id)]
    #[primary_key(comment_id)]
    profile_id: i32,
    #[foreign_key(Comment.id)]
    comment_id: i32,
    is_dislike: bool,
}

#[derive(Model)]
struct Comment {
    #[primary_key]
    id: i32,
    #[foreign_key(Profile.id)]
    author: i32,
    #[foreign_key(Post.id)]
    post: i32,
}
fn main() {}
```

If you now run the following command, your migrations should be automatically created.
``` 
$ models generate
```
The output should look like this: 
```
Generated: migrations/1632280793452 profile
Generated: migrations/1632280793459 post
Generated: migrations/1632280793465 postlike
Generated: migrations/1632280793471 comment
Generated: migrations/1632280793476 commentlike
```
You can check out the generated migrations at the `migrations/` folder. 
To execute these migrations you can execute the following command: 
```
models migrate run
```
The output should look like this: 
```
Applied 1631716729974/migrate profile (342.208µs)
Applied 1631716729980/migrate post (255.958µs)
Applied 1631716729986/migrate comment (287.792µs)
Applied 1631716729993/migrate postlike (349.834µs)
Applied 1631716729998/migrate commentlike (374.625µs)
```
If we later modify those structures in our application, we can generate new migrations to update the tables. 

## Reverting migration
Models can generate down migrations with the `-r` flag. Note that simple and reversible migrations cannot be mixed: 
```
$ models generate -r
```
In order to revert the last migration executed you can run: 
```
$ models migrate revert
```
If you later want to see which migrations are yet to be applied you can also excecute: 
```
$ models migrate info
```
Applied migrations need to be reverted before they can be deleted. 
## Available Attributes
### primary_key
It's used to mark the primary key fo the table. 
```rust
    #[primary_key]
    id: i32, 
```
for tables with multicolumn primary keys, the following syntax is used: 
```rust
    #[primary_key(second_id)]
    first_id: i32, 
    second_id: i32, 
```
This is equivalent to:
```sql
    PRIMARY KEY (first_id, second_id),
```

### foreign_key
It is used to mark a foreign key constraint. 
```rust
    #[foreign_key(Profile.id)]
    profile: i32, 
```
It can also specify `on_delete` and `on_update` constraints: 
```rust
    #[foreign_key(Profile.id, on_delete="cascade")]
    profile_id: i32, 
```
This is equivalent to:
```sql
    FOREIGN KEY (profile_id) REFERENCES profile (id) ON DELETE CASCADE,
```
### default
It can be used to set a default value for a column. 
```rust
    #[default(false)] // when using SQLite use 0 or 1
    is_admin: bool, 
    #[default("")]
    text: String, 
    #[default(0)]
    number: i32, 
```

### unique
It is used to mark a unique constraint. 
```rust
    #[unique]
    email: String, 
```
For multicolumn unique constraints the following syntax is used: 
```rust
    #[unique(post_id)]
    profile_id: String,
    post_id: i32,
```
This is equivalent to:
```sql
    UNIQUE (profile_id, post_id),
```
## CLI Short cuts
The CLI includes the following shortcuts: 
* `models database` -> `models db`
* `models generate` -> `models gen`
* `models migrate` -> `models mig`
