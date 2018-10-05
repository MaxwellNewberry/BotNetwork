# Peach Bot Network

In this repository, due to popular request, I will outline the overall structure of how my bots on the social media app, Peach, are set up and how they function. It's quite a simple process, however, for those who may not have as sophisticated background in coding or development, it may seem a bit more complicated. I will begin with some background information and technical information necessary for outside users to understand. So if you're here for the coding and development explaination skip ahead a bit. **Please note that all bots are created and utilize the [PeachAPI (PAPI)](https://github.com/MaxwellNewberry/PeachAPI) that I created.**

## Background Information

### What is Peach?

Peach was created by a development company called Byte and founded by Dom Hoffman, a previous co-founder of popular social app, Vine. The app was introduced initially and took off around January 2016. Initially, the app was discovered by accident and took off without previous marketing. The app was put on the app store so that the developer's parents could download the app and check it out. The social media network worked to eliminate the standard hashtag, tagging, and news feed standards.

### What inspired DogBot and the bots to follow?

DogBot, and the bots created following, are a series of automated accounts that respond to requests made by users within the community. These bots respond within one minute of the request and most, if not all, provide a picture relating to whatever the bot is centered around. Such as dogs, capybaras, Jim Halpert, Guy Fieri, and the outlier being [tagging](https://github.com/MaxwellNewberry/AccountTags). 

Aside from the love of dogs, my biggest inspiration was a Twitter account that used to center around the rating of dogs that were sent to them. I had just created a library that accessed a private API network that I discovered and needed something to make of it. I did small jobs that didn't ever gain traction. When I saw the account, I instantly knew what Peach, given the community's interest, needed as their first major project. *P.S. Not a big fan of cats, however, do own two.*

## Bot Network Structure

Each bot is built structured the same way. As mentioned before, the bots use the [PeachAPI (PAPI)](https://github.com/MaxwellNewberry/PeachAPI) library that I created and recently published. This library has been used for all bots, as well as the Peach Election, Friends of Friends feature, and more. All bots within the system use three main features: PAPI, Cronjob, and Peach. The structure is setup on one page that is references by a cronjob. 

### Peach API (PAPI)

The beginning of each bot starts, of course, by loggin in using the authorization as such up in the library. Once the token is given, we are able to continue on. Then, everytime the page is opened by the cronjob, we reference the activity() function within the the post category (Usage: **$api->post->activity()**). This returns the account's notifications, which we then want to filter out to *just* the mentions. We can do that by understanding what activity() returns. Which is shown [here](https://github.com/MaxwellNewberry/PeachAPI/wiki/Post-Functions#activity). 

#### Deriving only-mentions from the return

##### Step 1 - Getting to the notifications

Since the return is an object, we want to cast the object to be an array. The return also includes other elements that we don't need so we want to isolate just the notifications.

```php
$activity = $api->post->activity(); // get notifications
$activity = (array) $activity; // cast the return to an array
$activity = $activity['data']; // get only the items within the data item
$activity = (array) $activity; // cast the data return to array
$activity = $activity['activityItems']; // activityItems are the notifications array
```

Now that we want to go through each of the items in the array and check if the type is a mention ('tag'). This can be accomplished by any type of loop, but my preference is **foreach**.

```php
foreach($activity as $a)
{
	$a = (array) $a;
	if($a['type']=="tag")
	{
    // next step goes here
  }
}
```

##### Step 2 - Getting the user who mentioned the bot

Now that we have the notifications and filtered them to only the mentions of the bot, we want to now figure out who mentioned the bot so that we can mention them in our reply. Using the code from above, we want to now include the following inside the if-statement:

```php
/* each item/notification in array is set to be object */
$a = (array) $a['body']; // cast to array and get body
$postID = $a['postID']; // get the post ID
$a = (array) $a['authorStream']; // get who sent request
```

For each mention I save two things: 1) the post id, and 2) the author who sent the request. These two things help us with a couple things. I save the post id to the database so that I don't sent photos or whatever the bot does, twice for the same request. Then the author who sent the request is saved to a) act as a secondhand verification we have already sent something back to this person for this specific request and b) so that they are mentioned when we send whatever the bot sends back (so they know we have replied of course)!

#### Selecting an image to send

I use a recursive function to select the image that hasn't been sent to the user yet. This is why saving the author username is so important as well. Saving just the post id won't allow us to compare image links sent to a specific user. When a request comes in, we have a series of links in a database we choose from. We choose one that hasn't already been chosen, and once we find an image link, we return it. 

That's essentially all there is to this side of the bot functionality.

### Cronjob

The cronjob calls the one page once per minute (as to not entirely overload the system).

### Conclusion

If you have other questions please contact me on Peach or via e-mail on mnewber1@kent.edu - thanks!
