# Post API Routes

## Creating The Post Model
* Create Model in models folder
* create variable called Schema so you don't have to type mongoose.Schema later on
* posts are connected to user --> can make is so user can only delete their own posts and stuff like that
* add in name and avatar, this relates to user and doing that because we will want to have the option to not delete posts if you don't want to, say a user deletes account want to be able to keep the post, also want to be able to easily add avatar into post without having to dig into user collection
* system to like and remove your like --> create array of likes, and in array have the user ref as well (now we now what likes came from what user) --> it will basically be an array of user IDs
* similar for comments --> add text, etc for post --> along with name and avatar like we did with Post --> add date to comments
* add date to posts as well
    ```js
    const mongoose = require('mongoose');
    const Schema = mongoose.Schema;

    const PostSchema = new Schema({
      user: {
        type: Schema.Types.ObjectId,
        ref: 'users'
      },
      text: {
        type: String,
        required: true
      },
      name: {
        type: String
      },
      avatar: {
        type: String
      },
      likes: [
        {
          user: {
            type: Schema.Types.ObjectId,
            ref: 'users'
          }
        }
      ],
      comments: [
        {
          user: {
            type: Schema.Types.ObjectId,
            ref: 'users'
          },
          text: {
            type: String,
            required: true
          },
          name: {
            type: String
          },
          avatar: {
            type: String
          },
          date: {
            type: Date,
            default: Date.Now
          }
        }
      ],
      date: {
        type: Date,
        default: Date.Now
      }
    });

    module.exports = Post = mongoose.model('post', 'PostSchema');
    ```

## Add Post Route
* Go to posts.js route file -- create post -- private (need to be logged in)
* auth middleware, express validator, etc.
```js
const express = require('express');
const router = express.Router();
const { check, validationResult } = require('express-validator');
const auth = require('../../middleware/auth');

const Post = require('../../models/Post');
const Profile = require('../../models/Profile');
const User = require('../../models/User');

// @route   POST api/posts
// @desc    Create a post
// @access  Private
router.get(
  '/',
  [auth, [check('text', 'Text is required').not().isEmpty()]],
  async (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    try {
      // GRAB User info, using token, exclude password
      const user = await User.findById(req.user.id).select('-password');
      // name and avatar coming from user find
      const newPost = {
        text: req.body.text,
        name: user.name,
        avatar: user.avatar,
        user: req.user.id
      };

      const post = await newPost.save();

      res.json(post);
    } catch (err) {
      console.log(err.message);
      res.status(500).send('Server Error');
    }
  }
);

module.exports = router;
```
* TEST IN POSTMAN to see if error works with no text being sent
![error for add post route working](assets/post.png)
* now text with Text being sent --> actually got server error because at first we did not `instantiate a new post when we created nePost object`
* `const newPost = new Post({`
![add post route working](assets/post1.png)
* post with a different user now so we have multiple posts

## Get & Delete Post Routes
### GET ALL POSTS
* Create Get All Posts Route (private for now, can make public if you wanted), but we are building the front end so you would have to be logged in to even see the post, profiles are public but not posts --> main reason is to get people to sign up to communicate with other developers 
* find all posts and sort by date (-1) = most recent post first (oldest first is 1 and is also the default)
```js
// @route   GET api/posts
// @desc    Get all posts
// @access  Private
route.get('/', auth, async (req, res) => {
  try {
    const posts = await Post.find().sort({ date: -1 });
    res.json(posts);
  } catch (err) {
    console.log(err.message);
    res.status(500).send('Server Error');
  }
});
```
* Try in POSTMAN --> need token
![get all posts working](assets/post2.png)
### GET SINGLE POST BY ID
* copy get all, and find by id instead for route and mongoose query
* need to check if there is a post with that id before we send the post back --> also have same issue as profile if they pass in something that is not a valid ObjectId
```js
// @route   GET api/posts/:id
// @desc    Get post by id
// @access  Private
router.get('/:id', auth, async (req, res) => {
  try {
    const post = await Post.findById(req.params.id);

    if (!post) {
      return res.status(404).json({ msg: 'Post not found' });
    }

    res.json(post);
  } catch (err) {
    console.log(err.message);
    if (err.kind === 'ObjectId') {
      return res.status(404).json({ msg: 'Post not found' });
    }
    res.status(500).send('Server Error');
  }
});
```
* TEST IN POSTMAN
![get post by id test](assets/post3.png)
### DELETE POST BY ID
```js
// @route   DELETE api/posts/:id
// @desc    Delete a post
// @access  Private
router.get('/:id', auth, async (req, res) => {
  try {
    const post = await Post.findById(req.params.id);
    // if post does not exist
    if (!post) {
      return res.status(404).json({ msg: 'Post not found' });
    }
    // Check User --> can only delete posts owned by user
    // post.user is a an object ID and req.user.id is a string so we want them to match
    if (post.user.toString() !== req.user.id) {
      return res.status(401).json({ msg: 'User not Authorized' });
    }
    // else (user matches) remove post
    await post.remove();

    res.json({ msg: 'Post removed' });
  } catch (err) {
    console.log(err.message);
    if (err.kind === 'ObjectId') {
      return res.status(404).json({ msg: 'Post not found' });
    }
    res.status(500).send('Server Error');
  }
});
```
* TEST IN POSTMAN --> create dummy post
![delete post route test](assets/post4.png)

## Post Like & Unlike Routes

## Add & Remove Comment Routes