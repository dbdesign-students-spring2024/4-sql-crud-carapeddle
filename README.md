# SQL CRUD REPORT

## Part 1: Restaurant finder
### Code to create each table
```
DROP TABLE IF EXISTS restaurants;
CREATE TABLE restaurants (
    restaurant_id INTEGER PRIMARY KEY,
    restaurant_name TEXT NOT NULL,
    food_type TEXT NOT NULL,
    price_tier TEXT NOT NULL,
    neighorhood TEXT NOT NULL,
    opening_time TEXT NOT NULL,
    closing_time TEXT NOT NULL,
    ratings REAL NOT NULL,
    suitable_for_kids BOOLEAN NOT NULL
);
```

```
DROP TABLE IF EXISTS reviews;
CREATE TABLE reviews (
    id INTEGER PRIMARY KEY,
    restaurant_id INTEGER NOT NULL,
    rating INTEGER NOT NULL,
    review TEXT NOT NULL
);
```
### Link to practice CSV data file
- [Link to CSV File](./data/restaurants.csv)

### Code to import practice CSV data file into table
```
.mode csv
.headers on
.separator ","
.import /Users/carapeddle/Desktop/nyu3/dbase_design/4-sql-crud-carapeddle/data/restaurants.csv restaurants  --skip 1
```

### Tasks
1. Find all cheap restaurants in a particular neighborhood (pick any neighborhood as an example).
```
SELECT * FROM restaurants WHERE price_tier='Cheap' AND neighborhood='Greenwich Village';
```
2. Find all restaurants in a particular genre (pick any genre as an example) with 3 stars or more, ordered by the number of stars in descending order.
```
SELECT * FROM restaurants WHERE food_type='Italian' AND ratings>=3 order by ratings desc;
```
3. Find all restaurants that are open now (see hint below).
```
SELECT * FROM restaurants WHERE strftime('%H:%M', 'now','localtime') < closing_time AND strftime('%H:%M', 'now','localtime') >= opening_time;
```
4. Leave a review for a restaurant (pick any restaurant as an example; note that leaving a review has no automatic effect on the average rating of the restaurant).
```
INSERT INTO reviews (restaurant_id, rating, review) values ('190', '3', 'cheap food but not clean');
```
5. Delete all restaurants that are not good for kids.
```
DELETE FROM restaurants WHERE suitable_for_kids='FALSE';
```
6. Find the number of restaurants in each NYC neighborhood.
```
SELECT neighborhood, COUNT(restaurant_id) FROM restaurants GROUP BY neighborhood;
```

## Part 2: Social media app
### Code to create each table
```
DROP TABLE IF EXISTS users;
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    username TEXT NOT NULL UNIQUE,
    email TEXT NOT NULL,
    password TEXT NOT NULL
);
```

```
DROP TABLE IF EXISTS posts;
CREATE TABLE posts (
    id INTEGER PRIMARY KEY,
    user_id INTEGER NOT NULL,
    date_time_posted TEXT NOT NULL,
    views BOOLEAN NOT NULL,
    receiving_id INTEGER NOT NULL,
    post_type TEXT NOT NULL,
    content TEXT NOT NULL,
    visible BOOLEAN NOT NULL DEFAULT 'true'
);
```
### Links to practice CSV data files
- [Link to Users CSV File](./data/users.csv)
- [Link to Posts CSV File](./data/posts.csv)

### Code to import practice CSV data files into tables
```
.mode csv
.headers on
.separator ","
.import /Users/carapeddle/Desktop/nyu3/dbase_design/4-sql-crud-carapeddle/data/users.csv users  --skip 1
```
```
.mode csv
.headers on
.separator ","
.import /Users/carapeddle/Desktop/nyu3/dbase_design/4-sql-crud-carapeddle/data/posts.csv posts  --skip 1
```

### Tasks
1. Register a new User.
```
INSERT INTO users (id, username, email, password) VALUES ('1001', 'carapeddle', 'cep454@nyu.edu', 'Rand0m-P@ss');
```
2. Create a new Message sent by a particular User to a particular User (pick any two Users for example).
```
INSERT INTO posts (id, user_id, date_time_posted, views, receiving_id, post_type, content) VALUES ('2001', '465', '2024-02-25 17:22:58', 'false', '120', 'message', 'hi, how are you doing today?');
```
3. Create a new Story by a particular User (pick any User for example).
```
INSERT INTO posts (id, user_id, date_time_posted, views, receiving_id, post_type, content) VALUES ('2002', '100', '2024-02-25 15:15:15', 'true', '1', 'story', 'today is my birthday, everyone!');
```
4. Show the 10 most recent visible Messages and Stories, in order of recency.
```
SELECT * FROM posts WHERE (post_type='message' AND views='false') OR (post_type='story' AND ROUND((JULIANDAY('now','localtime') - JULIANDAY(date_time_posted)) * 24)<24) ORDER BY date_time_posted desc LIMIT 10;
```
5. Show the 10 most recent visible Messages sent by a particular User to a particular User (pick any two Users for example), in order of recency.
```
SELECT * FROM posts WHERE post_type='message' AND views='false' AND user_id='789' AND receiving_id='583' ORDER BY date_time_posted desc LIMIT 10;
```
6. Make all Stories that are more than 24 hours old invisible.
```
UPDATE posts SET visible = 'false' WHERE post_type = 'story' AND ROUND((JULIANDAY('now','localtime') - JULIANDAY(date_time_posted)) * 24) > 24;
```
7. Show all invisible Messages and Stories, in order of recency.
```
SELECT * FROM posts WHERE (post_type='message' AND views='true') OR visible='false' ORDER BY date_time_posted desc;
```
Can also update table so that visible is equal to 'false' when messages have been viewed and select all information when visible is 'false':
```
UPDATE posts SET visible = 'false' WHERE post_type = 'message' AND views='true';
SELECT * FROM posts WHERE visible='false' ORDER BY date_time_posted desc;
```
8. Show the number of posts by each User.
```
SELECT users.username, users.id, IFNULL(COUNT(posts.id),0) AS post_count FROM users LEFT JOIN posts ON users.id = posts.user_id GROUP BY users.id;
```
9. Show the post text and email address of all posts and the User who made them within the last 24 hours.
```
SELECT users.username, posts.user_id, users.email, posts.content FROM posts INNER JOIN users ON posts.user_id=users.id WHERE ROUND((JULIANDAY('now','localtime') - JULIANDAY(date_time_posted)) * 24) <= 24;
```
10. Show the email addresses of all Users who have not posted anything yet.
```
SELECT users.email FROM users LEFT JOIN posts ON users.id=posts.user_id WHERE posts.id IS NULL;
```