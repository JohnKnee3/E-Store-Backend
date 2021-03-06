mysql -u root -p

# 13.1.4

We installed mySQL12, sequalize and express.js. Then we went through and set up all the folders and files. We added a schema.sql to the db file that simply creates a drops and creates a new database named just_tech_news_db and ran it with source b/schema.sql to build this new database. Finally we created a new file called connection.js in the config folder and pasted in the sequalize set up. The sequalize() asks for our database name, user name and password. This time instead of making that infomation visible we downloaded an npm package named dot env with npm install dotenv. Then be created a file in the root of our directory called .env and set these things to variables like this.

## Setting username and password to mine

DB_NAME='just_tech_news_db'
DB_USER='your-mysql-username'
DB_PW='your-mysql-password'
--.

Then we went in the .gitignore to make sure that the .env folder does not get pushed to gitHub.

# 13.1.5

Created our first model using sequelize. We made the User.js file and copy and pasted in the info to start it's table.

## At the top we required

const { Model, DataTypes } = require("sequelize");
const sequelize = require("../config/connection");

// create our User model
class User extends Model {}
--.
The first one grabs the Model class from sequelize, then we require our SQL connection. Afterwards we make a new class called USER and give it all the functionality of the Model class we brought in from sequelize.

## Then we ran User.it() and made it look like this.

// define table columns and configuration
User.init(
{
// define an id column
id: {
// use the special Sequelize DataTypes object provide what type of data it is
type: DataTypes.INTEGER,
// this is the equivalent of SQL's `NOT NULL` option
allowNull: false,
// instruct that this is the Primary Key
primaryKey: true,
// turn on auto increment
autoIncrement: true,
},
// define a username column
username: {
type: DataTypes.STRING,
allowNull: false,
},
// define an email column
email: {
type: DataTypes.STRING,
allowNull: false,
// there cannot be any duplicate email values in this table
unique: true,
// if allowNull is set to false, we can run our data through validators before creating the table data
validate: {
isEmail: true,
},
},
// define a password column
password: {
type: DataTypes.STRING,
allowNull: false,
validate: {
// this means the password must be at least four characters long
len: [4],
},
},
},
{
sequelize,
timestamps: false,
freezeTableName: true,
underscored: true,
modelName: "user",
}
);
--.
In here the first object we created holds all the columns and defines each one one at a time giving the sequelize syntax so it can talk to mySQL and create the parameters for these columns. The second object configures certain options for the entire table.

Lastly we export the user with madule.exports = User; and went and made an index.js in the models folder that requires this User.js and then uses module.exports = {User}; to future proof the app so we can be ready for multiple exports.

# 13.1.6

Pretty big page. The gist is we added the get all, get by id, post, put and delete to the user-routes.js. Then we created an index.js in the routes/api folder, then we created another index in the routes folder. Each of these talk to each other and append to the URL in order.

## Finally we got our server,js up and running like this.

const express = require('express');
const routes = require('./routes');
const sequelize = require('./config/connection');

const app = express();
const PORT = process.env.PORT || 3001;

app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// turn on routes
app.use(routes);

// turn on connection to db and server
sequelize.sync({ force: false }).then(() => {
app.listen(PORT, () => console.log('Now listening'));
});
--.
We required the routes folder which will work because of the two index files which serve as a hub to talk to everything. Then we used app.use(routes); to turn them on. Then we started the sequalize at the bottom and are using force false for now. It claimed that if it was true it would drop and create the db each time we load and that this will counter that.

# 13.2.4

We downloaded bcrypt to protect our passwords using npm install bcrypt and requiring at the top of the User.js file that creates the rules for the User table. We are also planning on adding async to this because running this package is intensive and could take time, making it asynchronous will allow the function to continue while the bcrypt hash does it's thing.

# 13.2.5

Added code to the second object in the User,js which controls the table settings to wait for the passwords to hash out the brcypt when we create an account and when we update the account.

## The added hooks looks like this in the second object

{
hooks: {
// set up beforeCreate lifecycle "hook" functionality
async beforeCreate(newUserData) {
newUserData.password = await bcrypt.hash(newUserData.password, 10);
return newUserData;
},
// set up beforeUpdate lifecycle "hook" functionality
async beforeUpdate(updatedUserData) {
updatedUserData.password = await bcrypt.hash(
updatedUserData.password,
10
);
return updatedUserData;
},
},
sequelize,
timestamps: false,
freezeTableName: true,
underscored: true,
modelName: "user",
}
--.

Finally we had to slide into the user-routes.js and tweak with the put. The create was fine but in the put we needed to add a line

## just before the where in the put.

User.update(req.body, {
individualHooks: true,
where: {
id: req.params.id,
},
})

# 13.2.6

Created the post that verifies the user name and password.

## In user-routes.js it looks like this

router.post("/login", (req, res) => {
// expects {email: 'lernantino@gmail.com', password: 'password1234'}
User.findOne({
where: {
email: req.body.email,
},
}).then((dbUserData) => {
if (!dbUserData) {
res.status(400).json({ message: "No user with that email address!" });
return;
}

    const validPassword = dbUserData.checkPassword(req.body.password);
    if (!validPassword) {
      res.status(400).json({ message: "Incorrect password!" });
      return;
    }

    res.json({ user: dbUserData, message: "You are now logged in!" });

});
});
--.
First we set up the URL to perform a post with /login. Afterwards we use teh sequelize findOne to look things up by email this time. If it finds a matching email address everything is fine and just to the checkPassword function that passes in the current password.

## This function is on the User.js page and looks like this

class User extends Model {
// set up method to run on instance data (per user) to check password
checkPassword(loginPw) {
return bcrypt.compareSync(loginPw, this.password);
}
}
--.

Then over here bycrypt will work it's magic to compare the typed in plain text password to it's post hashed password. If it matched it sends back a true and you are good to log in.

# 13.3.4

Created the Post table in the Post.js file in the Models folder. Then we went into the index.js in the models folder and required and exported it. The main new thing about this post table is that it uses a foreign key that references the User table .

## The code for the table looks like this.

Post.init(
{
id: {
type: DataTypes.INTEGER,
allowNull: false,
primaryKey: true,
autoIncrement: true,
},
title: {
type: DataTypes.STRING,
allowNull: false,
},
post_url: {
type: DataTypes.STRING,
allowNull: false,
validate: {
isURL: true,
},
},
user_id: {
type: DataTypes.INTEGER,
references: {
model: "user",
key: "id",
},
},
},
{
sequelize,
freezeTableName: true,
underscored: true,
modelName: "post",
}
);
--.

Here we have the new guy user_id that is using the references:. Within that it targets the table by referencing model by with the table's name of user. Finally we use the key: which both simultaneously is our foreign key and references their primary key, we also target it by calling the foreign key by it's name of id as it is named in the User's table.

# 13.3.5

Went into the models index.js and had to define the relationship between the Post and User table so sequelize can undertand it.

## The code to define their relationship looked like this.

// create associations
User.hasMany(Post, {
foreignKey: "user_id",
});

Post.belongsTo(User, {
foreignKey: "user_id",
});
--.

This reads pretty well but to sum it up it says that the User table can have multiple post tables associated with it. Then we in and let it know that Post can only belong to ONE user. Both times we reference the fk which is user_id.

# 13.3.6

Added the post GET All, GET by id, POST, PUT and the DELETE. This one is pretty dense but walks you through the syntax that we have mostly seen before.

## One thing that juked me was the PUT looked like this.

router.put("/:id", (req, res) => {
Post.update(
{
title: req.body.title,
},
{
where: {
id: req.params.id,
},
}
)
.then((dbPostData) => {
if (!dbPostData[0]) {
res.status(404).json({ message: "No post found with this id" });
return;
}
res.json(dbPostData);
})
.catch((err) => {
console.log(err);
res.status(500).json(err);
});
});
--.

We needed to reference the title before the where. Everything else pretty much was a copy paste of the User routes.

## Also the GET looked like this.

// get all posts
router.get("/", (req, res) => {
Post.findAll({
attributes: ["id", "post_url", "title", "created_at"],
order: [["created_at", "DESC"]],
include: [
{
model: User,
attributes: ["username"],
},
],
})
.then((dbPostData) => res.json(dbPostData))
.catch((err) => {
console.log(err);
res.status(500).json(err);
});
});
--.

We needed the include object so it new how to go to the USER model and get the the info from the username column.

# 13.4.3

Created the vote table model. This time we had to include both user_id and the post_id since this will be talking to both tables.

## Talking to both tables looks like this.

Vote.init(
{
id: {
type: DataTypes.INTEGER,
primaryKey: true,
autoIncrement: true,
},
user_id: {
type: DataTypes.INTEGER,
allowNull: false,
references: {
model: "user",
key: "id",
},
},
post_id: {
type: DataTypes.INTEGER,
allowNull: false,
references: {
model: "post",
key: "id",
},
},
},
--.

Then in the index.js we had to add several associations to get this to work.

## The first one we created

User.belongsToMany(Post, {
through: Vote,
as: 'voted_posts',
foreignKey: 'user_id'
});

Post.belongsToMany(User, {
through: Vote,
as: 'voted_posts',
foreignKey: 'post_id'
});
--.
Here we link both Post and User through the Vote's table. we display the info as voted posts and use the fk we set up in the vote table for each one. This lets us see which Users voted on a single post or which posts a single user voted on.

## Finally we set up some one to many relationships like this

Vote.belongsTo(User, {
foreignKey: "user_id",
});

Vote.belongsTo(Post, {
foreignKey: "post_id",
});

User.hasMany(Vote, {
foreignKey: "user_id",
});

Post.hasMany(Vote, {
foreignKey: "post_id",
});
--.
This is some pretty important stuff that I may have to reread for the challenge.

# 13.4.4

We learned how to add an upvote through the PUT method. First reference the vote table by user_id and post_id and make sure the first two values insert match. The first being who is voting and the second being what is voted on. Then we go into the Post table and pull out info to display. The weird guy at the end had to be written in raw SQL because we are counting the Vote table and not the Post table which apparently sequelize does not know how to do. The raw code simply counts how many different users have voted on this specific post.

## The code looks like this

router.put("/upvote", (req, res) => {
// create the vote
Vote.create({
user_id: req.body.user_id,
post_id: req.body.post_id,
}).then(() => {
// then find the post we just voted on
return Post.findOne({
where: {
id: req.body.post_id,
},
attributes: [
"id",
"post_url",
"title",
"created_at",
// use raw MySQL aggregate function query to get a count of how many votes the post has and return it under the name `vote_count`
[
sequelize.literal(
"(SELECT COUNT(*) FROM vote WHERE post.id = vote.post_id)"
),
"vote_count",
],
],
})
.then((dbPostData) => res.json(dbPostData))
.catch((err) => {
console.log(err);
res.status(400).json(err);
});
});
});
--.
the "vote_count" means that is what the number of votes is named when displayed.

# 13.4.5

We updated the get routes in both post-routes.js and user-routes.js. We simply added the sequelize literal to the end of the attibutes in both of the GET routes in post-routes.js

## post-routes.js GET example

router.get("/", (req, res) => {
Post.findAll({
attributes: [
"id",
"post_url",
"title",
"created_at",
[
sequelize.literal(
"(SELECT COUNT(*) FROM vote WHERE post.id = vote.post_id)"
),
"vote_count",
],
],
order: [["created_at", "DESC"]],
include: [
{
model: User,
attributes: ["username"],
},
],
})
--.

Then in the user-routes we had to use include this time and used the

## "through" for the first time.

router.get("/:id", (req, res) => {
User.findOne({
attributes: { exclude: ["password"] },
where: {
id: req.params.id,
},
include: [
{
model: Post,
attributes: ["id", "title", "post_url", "created_at"],
},
{
model: Post,
attributes: ["title"],
through: Vote,
as: "voted_posts",
},
],
})
--.

# 13.4.6

Moved all the code that added the vote to the Post.js table up at the class near the top.

## Post.js class look like this

class Post extends Model {
static upvote(body, models) {
return models.Vote.create({
user_id: body.user_id,
post_id: body.post_id,
}).then(() => {
return Post.findOne({
where: {
id: body.post_id,
},
attributes: [
"id",
"post_url",
"title",
"created_at",
[
sequelize.literal(
"(SELECT COUNT(*) FROM vote WHERE post.id = vote.post_id)"
),
"vote_count",
],
],
});
});
}
}
--.

## Finally we went and cleaned up the code from post-routes.js

/ PUT /api/posts/upvote
router.put("/upvote", (req, res) => {
// custom static method created in models/Post.js
Post.upvote(req.body, { Vote })
.then((updatedPostData) => res.json(updatedPostData))
.catch((err) => {
console.log(err);
res.status(400).json(err);
});
});
--.
To call this new function we created. Here we call the upvote function and pass in the body from the request and the {Vote} model. Since we never had to call it Vote in Post.js we never had to require it up top.

# 13.5.3

We wet up the comment table using all existing code we have seen before and were encouraged to do this on our own. I didn't get any errors so this SHOULD be fine.... Then we went into index.js and set up the associations that both the POST and USER table have with the COMMENT table.

## It looked like this.

Comment.belongsTo(User, {
foreignKey: "user_id",
});

Comment.belongsTo(Post, {
foreignKey: "post_id",
});

User.hasMany(Comment, {
foreignKey: "user_id",
});

Post.hasMany(Comment, {
foreignKey: "post_id",
});
--.
The top 2 basically say when you check User or Post you will beable to check any comment tied to them while the second grouping allows you to see multiple comments tied to them.

# 13.5.4

Set up the GET, POST and DELETE for sequelize in the comment-routes.js. Nothing really new here that hasn't been used in previous code.

## Here is an example of the code

router.get("/", (req, res) => {
Comment.findAll()
.then((dbCommentData) => res.json(dbCommentData))
.catch((err) => {
console.log(err);
res.status(500).json(err);
});
});

router.post("/", (req, res) => {
Comment.create({
comment_text: req.body.comment_text,
post_id: req.body.post_id,
user_id: req.body.user_id,
})
.then((dbCommentData) => res.json(dbCommentData))
.catch((err) => {
console.log(err);
res.status(400).json(err);
});
});

router.delete("/:id", (req, res) => {
Comment.destroy({
where: {
id: req.params.id,
},
})
.then((dbCommentData) => {
if (!dbCommentData) {
res.status(404).json({ message: "No comment found with this id" });
return;
}
res.json(dbCommentData);
})
.catch((err) => {
console.log(err);
res.status(500).json(err);
});
});
--.  
I was basically able to copy and paste from the previous routes and just had to make sure everything was named properly as is usually the error that occurs when you eat copy pasta.

# 13.5.5

We added the Comment model to existing GETs.

## In the user-routes GET ALL we added

router.get("/", (req, res) => {
console.log("======================");
Post.findAll({
order: [["created_at", "DESC"]],
attributes: [
"id",
"post_url",
"title",
"created_at",
[
sequelize.literal(
"(SELECT COUNT(*) FROM vote WHERE post.id = vote.post_id)"
),
"vote_count",
],
],
include: [
// include the Comment model here:
{
model: Comment,
attributes: ["id", "comment_text", "post_id", "user_id", "created_at"],
include: {
model: User,
attributes: ["username"],
},
},
{
model: User,
attributes: ["username"],
},
],
})
--.  
This makes sure that when you look at all posts it also pulls up all of their comments.

## Next we added this same code to the post's find one

router.get("/:id", (req, res) => {
Post.findOne({
where: {
id: req.params.id,
},
attributes: [
"id",
"post_url",
"title",
"created_at",
[
sequelize.literal(
"(SELECT COUNT(*) FROM vote WHERE post.id = vote.post_id)"
),
"vote_count",
],
],
include: [
// include the Comment model here:
{
model: Comment,
attributes: ["id", "comment_text", "post_id", "user_id", "created_at"],
include: {
model: User,
attributes: ["username"],
},
},
{
model: User,
attributes: ["username"],
},
],
})
--.
This will pull up one post by id a make sure to display all comments attached to that specific post.

## Finally we did the same thing in user-routes.js for then we find one user.

router.get("/:id", (req, res) => {
User.findOne({
attributes: { exclude: ["password"] },
where: {
id: req.params.id,
},
include: [
{
model: Post,
attributes: ["id", "title", "post_url", "created_at"],
},
{
model: Comment,
attributes: ["id", "comment_text", "post_id", "user_id", "created_at"],
include: {
model: User,
attributes: ["username"],
},
},
{
model: Post,
attributes: ["title"],
through: Vote,
as: "voted_posts",
},
],
})
--.
This makes it so when you pull up as user by it's id it will also grab all of it's comments. I also discovered if you want to reference multiple tables in the same include the syntax must look something like this.

## Multiple Tables

include: [
// include the Comment model here:
{
model: Comment,
attributes: ["id", "comment_text", "post_id", "user_id", "created_at"],
include: {
model: User,
attributes: ["username"],
include: {
model: Post,
attributes: ["title"],
},
},
},
{
model: User,
attributes: ["username"],
},
],
})

It looks a little silly but this will grab every comment. Then it will create a user name associated with that post and then it will also show every post by title that this user has ever made.

# 13.5.6

Learned how to add the JawsDb to the heroku page. It'll be best to just come back to this page when relevant. We had to change some code in connection.js to make Heroku happy.

## This

const sequelize = new Sequelize(process.env.DB_NAME, process.env.DB_USER, process.env.DB_PW, {
host: 'localhost',
dialect: 'mysql',
port: 3306
});
--.

## Turned into this

let sequelize;

if (process.env.JAWSDB_URL) {
sequelize = new Sequelize(process.env.JAWSDB_URL);
} else {
sequelize = new Sequelize(
process.env.DB_NAME,
process.env.DB_USER,
process.env.DB_PW,
{
host: "localhost",
dialect: "mysql",
port: 3306,
}
);
}
