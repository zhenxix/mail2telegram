
<h1 align="center">
mail2telegram
</h1>

<p align="center">
    <br> English | <a href="README_CN.md">中文</a>
</p>
<p align="center">
    <em>Use Telegram Bot to get your temporary email..</em>
</p>


This is a Telegram Bot based on Cloudflare Email Routing Worker, which can convert emails into Telegram messages. You can forward emails from recipients with any prefix to the Bot, and then a temporary mailbox Bot with an infinite address will be created.

<details>
<summary>Click to view the demo.</summary>
<img style="max-width: 600px;" alt="image" src="example.png">
</details>



## Installation

### 1. Deploy Workers

#### 1.1 Deploy via Command Line

- Clone the repository:

    `git clone git@github.com:TBXark/mail2telegram.git`
- Copy the configuration template and modify it with your own Telegram configuration: 

    `cp wrangler.example.toml wrangler.toml`
- Deploy 

    `yarn & yarn pub`

#### 1.2 Deploy via Copy and Paste

- If you don't want to deploy using the command line and prefer to copy and paste, you can use the precompiled version > [`index.js`](./build/index.js)
- When deploying via copy and paste, you need to manually set environment variables in the project's configuration page.
- To generate a whitelist/blacklist of regular expressions as a JSON array string, you can use this small tool which also includes some demos: [regexs2jsArray](https://codepen.io/tbxark/full/JjxdNEX)


### 2. Configure Cloudflare Email Routing

- Follow the official tutorial to configure [Cloudflare Email Routing](https://blog.cloudflare.com/introducing-email-routing/).
- Configure routing by changing the action of `Catch-all address` in `Email Routing - Routing Rules` to `Send to a Worker:mail2telegram`. Forward all remaining emails to this worker.
- If you set `Catch-all address` as workers, you won't be able to forward all remaining emails to your own email. If you need to backup emails, simply fill in your backup email in the `FORWARD_LIST` environment variable of the worker.



## Configuration

| KEY                    | Description                                                                                                                                          |
|------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| TELEGRAM_ID            | Your Telegram ID, pure numbers can be obtained through the `/id` command.                                                                            |
| TELEGRAM_TOKEN         | Telegram Bot Token                                                                                                                                   |
| DOMAIN                 | Workers domain name                                                                                                                                  |
| FORWARD_LIST           | Backup emails, can be forwarded to your own email for backup, leave blank if not forwarding, multiple values can be separated by `,`                 |
| WHITE_LIST             | Sender whitelist, an array of regular expressions converted to a string, example: `[\".*@10086\\\\.cn\"]`                                            |
| BLOCK_LIST             | Sender blacklist, an array of regular expressions converted to a string                                                                              |
| MAIL_TTL               | Email cache retention time in seconds, default is one day. After expiration, emails will no longer be previewable. Please make sure to back them up. |
| OPENAI_API_KEY         | OpenAI API Key, Used for summarizing email content. If not filled out, the "Summary" button will not appear.                                         |
| OPENAI_COMPLETIONS_API | Customizable API, default value is `https://api.openai.com/v1/chat/completions`                                                                      |
| OPENAI_CHAT_MODEL      | Customizable model, default value is `gpt-3.5-turbo`                                                                                                 |
| SUMMARY_TARGET_LANG    | The language for customizing the summary, with a default value of `english`                                                                          |
| GUARDIAN_MODE          | Guard mode, default off, if you want to enable it, fill in `true`.                                                                                   |
| DB                     | Bind `KV Namespace Bindings` database to worker with the name `DB`.                                                                                  |

> `WHITE_LIST` and `BLOCK_LIST` take effect on both recipients and senders at the same time, with `WHITE_LIST` having a higher priority than `BLOCK_LIST`.

> The email address in `FORWARD_LIST` should be added to Email Routing - Destination addresses after authentication in order to receive emails.

## Usage


The default message structure is as follows.
```
[Subject]

-----------
From : [sender]
To: [recipient]

(Preview)(Summary)(Text)(HTML)

```

When the email forwarding notification is sent to Telegram, only the title, sender, recipient, and four buttons are included.

Using `Preview` allows you to directly preview the plain text mode of the email in the bot. However, there is a limit of 4096 characters. If it exceeds 4096 characters, you can use `TEXT` or `HTML` to view the complete email. Below the preview message, there is a `Back` button that can be clicked to close the preview.

When you configure `OPENAI_API_KEY`, a `Summary` button will appear so that you can summarize the content of the email. Below the summary message, there is a `Back` button that can be clicked to close the summary.

Using `TEXT`, you can see plain text emails; using `HTML`, you can see rich-text emails. However, they may contain certain scripts or other tracking links. It is recommended to use rich-text mode only when necessary or when confirming that there are no issues with its source.

For security reasons, when exceeding the mail cache retention time (`MAIL_TTL`), links opened by buttons cannot be accessed. You can modify environment variables yourself to adjust expiration time.

This Bot does not support attachments. If you need attachment support, you can combine it with my another project [testmail-viewer](https://github.com/TBXark/testmail-viewer) and forward emails to your testmail using `FORWARD_LIST`. This way, you can download your attachments using [testmail-viewer](https://github.com/TBXark/testmail-viewer).

Due to the limitation of Workers, if the email size (especially the attachment size) is too large, it will cause function timeout. At this time, Workers will retry multiple times, which may result in receiving multiple notifications and backup emails. Moreover, there is a limit on the number of retries. Once exceeded, no further retries will be made. This may lead to your email being lost. Therefore, it is recommended that you fill in your backup email in `FORWARD_LIST`, so that even if your email is lost, you can find it in your backup mailbox. If you frequently encounter this problem, you can try enabling Guardian Mode (`GUARDIAN_MODE`), which will record the operations already performed and not execute them again during the next retry. This reduces interference from duplicate messages and increases the success rate of workers. However, this operation consumes more KV write operations, so it is recommended to enable it when necessary.

## License

**mail2telegram** is released under the MIT license. [See LICENSE](LICENSE) for details.

