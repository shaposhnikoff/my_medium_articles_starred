
# Setting your Telegram Bot WebHook the easy way

In order to make your Bot answering to requests from your Telegram users you need to manually request for updates to the Bot API or you can register a WebHook to automatically being called once updates are available.

The latter is a better and more efficient solution.

That being said, the quickest and easiest way to set a WebHook for your Bot is to issue a GET request to the Bot API (it’s enough to open an url in your browser).

All you have to do is to call the setWebHook method in the Bot API via the following url:

    [https://api.telegram.org/bot{my_bot_token}/setWebhook?url={](https://api.telegram.org/bot(mytoken)/setWebhook?url=https://mywebpagetorespondtobot/mymethod)url_to_send_updates_to}

where

* **my_bot_token** is the token you got from BotFather when you created your Bot

* **url_to_send_updates_to is the url** of the piece of code you wrote to implement your Bot behavior (must be HTTPS)

For instance:

    [https://api.telegram.org/bot](https://api.telegram.org/bot(mytoken)/setWebhook?url=https://mywebpagetorespondtobot/mymethod)123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11[/setWebhook?url=](https://api.telegram.org/bot(mytoken)/setWebhook?url=https://mywebpagetorespondtobot/mymethod)https://www.example.com/my-telegram-bot

And you’ve got it.

Now if you go to the following url (you have to replace {my_bot_token} with your Bot Token)

    [https://api.telegram.org/bot{my_bot_token}/getWebhookInfo](https://api.telegram.org/bot{my_bot_token}/getWebhookInfo)

you should see something like this:

    {
     "ok":true,
     "result": 
     {
       "url":"[https://www.example.com/my-telegram-bot/](https://www.example.com/my-telegram-bot/)",
       "has_custom_certificate":false,
       "pending_update_count":0,
       "max_connections":40
     }
    }

For a complete list of parameters for the setWebHook method have a look at the [official API reference](https://core.telegram.org/bots/api#setwebhook).

This story is also available on [xabaras.it](http://www.xabaras.it/posts/687) (in italian).
