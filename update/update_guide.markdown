# Diesel Update Guide

This guide is intended for those who are somewhat familiar with Rust and have gone through the [basic diesel guide](http://diesel.rs/guides/getting-started/).

This guide will walk you through more examples of updating records using Diesel and will build on top of the original tutorial.

## Updating from the Diesel guide

If you've followed the Diesel getting started guide, you'll have something that looks like this for updating a post:

```rust
/// src/bin/publish_post.rs
extern crate diesel_demo;
extern crate diesel;

use self::diesel::prelude::*;
use self::diesel_demo::*;
use self::diesel_demo::models::Post;
use std::env::args;

fn main() {
    use diesel_demo::schema::posts::dsl::{posts, published};

    let id = args().nth(1).expect("publish_post requires a post id")
        .parse::<i32>().expect("Invalid ID");
    let connection = establish_connection();

    let post = diesel::update(posts.find(id))
        .set(published.eq(true))
        .get_result::<Post>(&connection)
        .expect(&format!("Unable to find post {}", id));
    println!("Published post {}", post.title);
}
```

First, we find the post by the id that we pass into the CLI. Then we call .set on the result where we set the column published to be true. Then we grab the result and pass that to our print line. 

The next example will take you through updating other types of columns besides a bool. 

## Passing multiple values into set

Let's say instead of just setting published to true on your post, you also wanted to update the title to tell the world you've published this post. 

```rust
fn main() {
    use diesel_update::schema::posts::dsl::{posts, published, title};

    let id = args().nth(1).expect("publish_post requires a post id")
        .parse::<i32>().expect("Invalid ID");
    let connection = establish_connection();

    let post = diesel::update(posts.find(id))
        .set((published.eq(true), title.eq("I've been updated")))
        .get_result::<Post>(&connection)
        .expect(&format!("Unable to find post {}", id));
    println!("Published post {}", post.title);
}
```

As you can see here, we've now passed in multiple values to set and it will change both of the columns. If we even wanted to pass in description too, we could do that.

One note is the first like of the function, you can see that we added title to `use diesel_update::schema::posts::dsl::{posts, published, title};`. We have to explicitly tell Diesel what columns we want to update.

## Updating a Post with structs

Diesel will also let us create custom structs that we can use to update our posts. 

Let's create a new file `src/bin/update_post_with_struct.rs`

And in that file, first we will import some things from Diesel:

```rust
#![feature(custom_derive, custom_attribute, plugin)]
#![plugin(diesel_codegen, dotenv_macros)]

extern crate diesel_demo;
extern crate diesel;

use self::diesel::prelude::*;
use self::diesel_demo::*;
use self::diesel_demo::models::Post;
use self::diesel_demo::schema::posts;
use std::env::args;

```

Our setup is identical to `publish_post` with 3 exceptions. 

In order to use a struct to update our posts, we have to create a custom changeset for Posts. Adding the first two lines will allow us to do this. The other addition we made was adding `use self::diesel_update::schema::posts;`

Now let's add our struct below the setup.

```rust
#[changeset_for(posts)]
struct UpdatePost {
    id: i32,
    title: String,
    body: Option<String>,
}
```

Here we're creating a change set for a Post. The struct is called UpdatePost and the attributes we want are id, title, and body. Though here, we are saying that the body can be optional.

Now to the meat, the main function below where we define the changeset.

```rust
fn main() {
    use diesel_update::schema::posts::dsl::{posts, title, body};

    let id = args().nth(1).expect("update_post requires a post id")
        .parse::<i32>().expect("Invalid ID");
    let connection = establish_connection();

    let changes = UpdatePost { id: id, title: "Updated title".into(), body: None };

    let updated_post = changes.save_changes::<Post>(&connection);
}
```

There are a few differences from our publish post example. First on the first line of the function, we are defining the attributes that we'll want to work with, namely title and body. Here we don't care about published, so we leave that out.

We say that the changes are equal to a new UpdatePost struct we define, where the id equals the id we pass in, the title is a string of `"Updated title"` and we leave the body alone.

Then, we save the changes to the post borrowing our connection. Try it out! Try updating a post you've published from the diesel tutorial. `cargo run --bin update_post_with_struct 1`. When you run `cargo run --bin show_posts` you should see your updated title!