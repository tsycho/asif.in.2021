---
title: "Google OAuth and Rails"
date: 2012-03-03
tags: ["rails", "dev"]
---

While building [InboxSlasher](http://www.inboxslasher.com), I needed to enable users to give me access to their Gmail accounts via OAuth. The [gmail_xoauth](http://github.com/nfo/gmail_xoauth) gem helped me do what I needed once the authentication was set up, but referred to [Google's python code](http://code.google.com/p/google-mail-xoauth-tools/wiki/XoauthDotPyRunThrough) for generating the actual OAuth tokens. Since I obviously couldn't use that from within my Rails app, I translated the Python code to Ruby as shown below.

Note: The Readme on Github for [gmail_xoauth](http://github.com/nfo/gmail_xoauth) now has a link to some Sinatra code for the OAuth token generation. I haven't checked it out. However, anyone implementing my solution should probably do so. Also, please let me know if that is an easier or better solution, and I will update this post.

Here's the code for the OAuth token generator. This is also saved as a [gist](https://gist.github.com/1970639).
<script src="https://gist.github.com/1970639.js?file=ruby_oauth_token_generator.rb"></script>

### How to use this
1) Save the above into a file that is accessible to your Rails controllers.

2) From the controller where you want to initiate the token generation request (UsersController in my case), call the generate\_request\_token() method. Save the oauth\_token and oauth\_token\_secret from above into the User's model, and redirect the user to oauth\_request\_url.
<script src="https://gist.github.com/tsycho/5328705.js"></script>

3) Once the user has given your app permission (or refused to do so), Google will send a POST to the callback url that was specified above (see line #56). Modify the following code appropriately to handle the callback.
<script src="https://gist.github.com/tsycho/5328714.js"></script>

If everything above worked fine, you should now have the user's oauth\_token and oauth\_token\_secret for Gmail. Use the [gmail_xoauth](http://github.com/nfo/gmail_xoauth) gem for the rest of your work.

### Notes:
1. I used 'anonymous' for the consumer token and secret. If you have a consumer token and secret from Google, you should use that instead.
2. If you need access to more Google services and not just Gmail, add them to params['scope'] on line #58.
3. I had to define my own method for URL escaping since CGI.escape was converting spaces into '+' instead of '%20', which was causing me problems. I am not sure what is the right approach here.
4. I have not find a way to specify the email address of the user when requesting the oauth token. The above methodology asks the user for permission to whichever Google account they are currently signed in on their browser, which in some cases (such as if a user gave you an email address they want to use, but are signed in as a different one), can be a problem. Google's permission page does allow the user to log out and back in as the desired user, but people might not notice that. If someone knows a way to specifying the email account during the token generation, please let me know about it.
