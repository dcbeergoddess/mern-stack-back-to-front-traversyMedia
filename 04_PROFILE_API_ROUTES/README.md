# Profile API Routes

## Creating The Profile Model
* Create Profile Model File
    - create ref to user model
    ```js
      const ProfileSchema = new mongoose.Schema({
        user: {
          type: mongoose.Schema.Types.ObjectId,
          ref: 'user'
        }
      });
    ```
    - Add in fields from Brad's repo in github --> many fields but pretty straight forward
* Now we can bring profile model into our profile routes and query teh database --> get profiles, create them, update, and delete

## Get Current User Profile
* create route to fetch our own profile
```js
// @route   GET api/profile/me
// @desc    Get current users profile
// @access  Private
router.get('/me', auth, async (req, res) => {
  try {
    // find user by user ref id and populate the name and avatar
    const profile = await Profile.findOne({ user: req.user.id }).populate(
      'user',
      ['name', 'avatar']
    );
    //if no profile then send error
    if (!profile) {
      return res.status(400).json({ msg: 'There is no profile for this user' });
    }
    //if profile exists --> send it along
    res.json(profile);
  } catch (err) {
    console.error(err.msg);
    res.status(500).send('Server Error');
  }
});
```
* IN POSTMAN Try with our token --> error because we have no profile
![get current user profile test](assets/profile.png)

## Create & Update Profile Routes
* Route to create or update profile --> probably our longest route --> bring in express validator to routes file
    - POST request that takes in data
    - want ot bring in express validator stuff
    - need auth middleware and check for required items in model
    - create errors variable to check if req passes validation
    - if there are errors array we will throw client error and send JSON which will be an object with an errors array that we get from errors
        ```js
        // @route   POST api/profile/
        // @desc    Create or Update user profile
        // @access  Private
        router.post(
          '/',
          [
            auth,
            [
              check('status', 'Status is required').not().isEmpty(),
              check('skills', 'Skills is required').not().isEmpty()
            ]
          ],
          async (req, res) => {
            const errors = validationResult(req);
            if (!errors.isEmpty()) {
              return res.status(400).json({ errors: errors.array() });
            }
          }
        );
        ```
    - Test Validation in POSTMAN -> add content-type and token in Headers (save token and content-type as a Preset to quickly add in future)
    ![validation test](assets/profile1.png)
    - Add in Status and Skills --> it's passing but we haven't told it to send anything back yet
    ![validation test](assets/profile2.png)
* Pull out all fields from req.body
    - check to make sure that fields were added before we try to submit to database
    - build a profile object --> initialize profile fields to an empty object --> go one by one and add each field
    - we can get the user from req.user.id (knows that from token that is sent)
    - add all fields (expect social) --> need to make skills into array (right now this is going to be a comma separated list), take skills and `.split()` --> turns a string into an array --> it takes in a delimiter which is going to be the comma, want it to not matter if there's a space or even 10 space, --> map through array (chain on map method) --> and for each skill go ahead and trim it
    ```js
      // DESTRUCTURE ALL THE FIELDS WE WANT TO PULL OUT from req.body
          const {
            company,
            website,
            location,
            bio,
            status,
            githubusername,
            skills,
            youtube,
            facebook,
            twitter,
            instagram,
            linkedin
          } = req.body;
          // BUILD PROFILE OBJECTS
          const profileFields = {};
          // get from token
          profileFields.user = req.user.id;
          if (company) profileFields.company = company;
          if (website) profileFields.website = website;
          if (location) profileFields.location = location;
          if (bio) profileFields.bio = bio;
          if (status) profileFields.status = status;
          if (githubusername) profileFields.githubusername = githubusername;
          if (skills) {
            profileFields.skills = skills.split(',').map(skill => skill.trim());
          }

          console.log(profileFields.skills);
          res.send('Hello');
    ```
    - TEST IN POSTMAN
    ![new postman test of route](assets/profile3.png)
* Add in social fields --> 
    - we have to initialize --> set `profileFields.social` to an empty object (otherwise it's throw an error that it would be able to find anything we set to profileFields.social because social is undefined) 
        ```js
            // Build social object
            profileFields.social = {};
            if (youtube) profileFields.social.youtube = youtube;
            if (twitter) profileFields.social.twitter = twitter;
            if (facebook) profileFields.social.facebook = facebook;
            if (linkedin) profileFields.social.linkedin = linkedin;
            if (instagram) profileFields.social.instagram = instagram;

        ```
* now that we have all these properties inside of profileFields we are ready to actually update and insert the data
    - try and catch block
    - take profile model and find one by user (remember that user field is the object id and we can match that to req.user.id which comes from the token)
    - if there is a profile then that means we want to update it
    - anytime a mongoose method is used it returns a promise so we need to await it
    - find one by user and update it, and then set profile fields, and also add another parameter object with new and set that to true
    - we want to return the entire profile --> THIS IS FOR IF THERE IS A PROFILE
    - If there is no profile then we will want to create it and set it to a new profile, pass in profile fields, and then we need to save it, and then we want to return the profile
      ```js
          try {
            let profile = await Profile.findOne({ user: req.user.id });

            if (profile) {
              //UPDATE
              profile = await Profile.findOneAndUpdate(
                { user: req.user.id },
                { $set: profileFields },
                { new: true }
              );

              return res.json(profile);
            }
            // Create Profile (if does not exist yet)
            profile = new Profile(profileFields);

            await profile.save();
            res.json(profile);
          } catch (err) {
            console.error(err);
            res.status(500).send('Server Error');
          }
      ```
* TEST IN POSTMAN
![create profile in postman](assets/profile4.png)
- experience and education will initialize as empty arrays, we will have endpoints to create those later 

## Get All Profiles & Profile By User ID


## Delete Profile & User

## Add Profile Experience

## Delete Profile Experience

## Add & Delete Profile Education

## Get Github Repos For Profile