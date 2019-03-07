
<!-- README.md is generated from README.Rmd. Please edit that file -->

<p align="center">

<img /><img src="man/figures/logo.svg" align="center" height="500px" />

</p>

<br />

[![Travis-CI Build
Status](https://travis-ci.org/rich-iannone/blastula.svg?branch=master)](https://travis-ci.org/rich-iannone/blastula)
[![Coverage
status](https://codecov.io/gh/rich-iannone/blastula/branch/master/graph/badge.svg)](https://codecov.io/github/rstudio/gt?branch=master)

The **blastula** package makes it easy to produce and send HTML email
from **R**. The message body can be highly customized with **Markdown**
text, the results of **R** expressions, and even raw HTML.

### Installation Requirements

Blastula is moving toward using a new binary for `smtp` mailing,
provided by the **mailsend-go** project. This binary is cross-platform
and works on Windows, macOS (via Homebrew), and Linux (Debian and RPM
packages). Installation instructions can be found at
[here](https://github.com/muquit/mailsend-go#downloading-and-installing).

Once the `mailsend-go` binary is installed and on the system path, we
can use the new `smtp_send()` function for sending email. The other
function for sending email (`smtp_send()`) will undergo deprecation.

### Composing an Email Message Object

Here’s an example that shows a basic workflow for composing the message,
creating optional on-disk credentials for email, and sending out the
message.

These functions can help us do just that:

  - `compose_email()`: generates the email message content inside the
    `email_message`-class object
  - `create_email_creds_file()`: generates an optional on-disk file with
    email credentials
  - `smtp_send()`: sends the HTML-based email to one or more recipients

A few other functions allow us to easily insert HTML fragments into the
message body. These are:

  - `add_image()`: for embedding a local image file into the message
    body
  - `add_ggplot()`: converts a ggplot plot object to an image for
    embedding
  - `add_imgur_image()`: simultaneously deploys an image to the
    **Imgur** service and provides an `<img>` tag pointing to the
    external image
  - `add_readable_time()`: get the current time as a nicely readable
    string
  - `add_cta_button()`: add a call-to-action (CTA) button with button
    text and a link

When you compose an email, you can use objects from the global workspace
and work them into the message content. Here, I’ll create a nicely
formatted date/time string (`current_date_time`) with the package’s
`add_readable_time()` function, and, assign a link to an web image to an
object (`img_link`).

``` r
# Get a nicely formatted date/time string
current_date_time <- add_readable_time()

# Assign an URL with an image to `img_link`
img_link <- "https://i.imgur.com/p1KvgYj.png"
```

Now we use the `compose_email()` function to compose the email. There
are two main arguments here, `body` and `footer`. You can supply
**Markdown** text to each of these. All other valid **Markdown**
conventions should render to valid HTML.

The insertion of HTML fragments or text can be performed by enclosing
valid R code inside of curly braces (`{...}`). Below the image URL (as
part of the `![...](...)` **Markdown** link construction) is referenced
to the `img_link` object from the global workspace. Note also that
`{current_date_time}` references the `current_date_time` character
object generated earlier via the `add_readable_time()` function. The end
result is the insertion of the date/time string into the footer of the
email. (Alternatively, `add_readable_time()` could have been called
directly.)

We can also supply variables in the `compose_email()` function directly.
For example, the `{sender}` part references an object *not* in the
global workspace. Rather, it refers the named argument `sender =
"Shelly"` in the function call. The order of searching is from within
the function first, then, the search moves to variables in the global
environment.

``` r
# Generate the body text for the email message
email_text <- 
"
Hello,

This is a great photo of Lake Tahoe. I took \\
it using my new camera.

![]({img_link})
      
You should go if you get the chance.

Sincerely,

{sender}
"

email_object <-
  compose_email(
    body = email_text,
    footer = 
  "Email sent on {current_date_time}.",
    sender = "Shelly")
```

Some more notes on style are useful here. The `\\` is a helpful line
continuation marker. It’ll help you break long lines up when composing
but won’t introduce line breaks or new paragraphs. I recommend
formatting like above with few indents so as not to induce the
`quote`-type formatting. Any literal quotation marks should be escaped
using a single `\`. Blank lines separating blocks of text result in new
paragraphs. And, again, any valid **R** code can be enclosed inside
`{...}` (e.g., `{Sys.Date()}`).

After creating the email message, we can look at it to ensure that the
formatting is as expected. Simply call the object itself and it will be
displayed in the Viewer pane.

``` r
# Preview the email
email_object
```

<img src="man/figures/rstudio_preview_email.png">

### Sending an Email Message

We can store email credentials in a file using the
`create_email_creds_file()` function. Here is an example showing how to
create a credentials file as a hidden file in the working directory.

``` r
# Create a credentials file to facilitate
# the sending of email messages
create_email_creds_file(
  user = "user@site.org",
  password = "<user_password>",
  host = "smtp.blastula.org",
  port = 465,
  use_ssl = TRUE,
  sender = "The User's Name",
  creds_file_name = ".email_creds"
)
```

We can also use preset SMTP settings. For example, if we would like to
send email through **Gmail**, we can supply `provider = gmail` to not
have to worry about SMTP settings:

``` r
# Create a credentials file for sending
# email through Gmail
create_email_creds_file(
  user = "user_name@gmail.com",
  password = "<user_password>",
  provider = "gmail",
  sender = "Sender Name"
)
```

This will create a hidden credentials file in the working directory, the
name of which is based on the provider (you can optionally specify the
name with the `creds_file_name` argument, as in the first example).

Having generated the credentials file, we can use the `smtp_send()`
function to send the email.

``` r
# Sending email using a credentials file
smtp_send(
  message = email_object,
  from = "user@site.org",
  to = "another_user@web.net",
  subject = "Testing the `smtp_send()` function",
  creds_file = ".email_creds"
)
```

Alternatively, one can set a number of environment variables and use
`Sys.getenv()` calls for email credentials arguments in the
`smtp_send()` statement.

``` r
# Sending email using environment variables
smtp_send(
  message = email_object,
  from = "user@site.org",
  to = "another_user@web.net",
  subject = "Testing the `smtp_send()` function",
  sender = Sys.getenv("BLS_SENDER"),
  host = Sys.getenv("BLS_HOST"),
  port = Sys.getenv("BLS_PORT"),
  user = Sys.getenv("BLS_USER_NAME"),
  password = Sys.getenv("BLS_PASSWORD")
)
```

The underlying HTML/CSS is meant to display properly across a wide range
of email clients and webmail services.

### Installation Requirements

Blastula is moving toward using a new binary for `smtp` mailing,
provided by the **mailsend-go** project. This binary is cross-platform
and works on **Windows**, **macOS** (via **Homebrew)**, and **Linux**
(**Debian** and **RPM** packages). Installation instructions can be
found at the
[here](https://github.com/muquit/mailsend-go#downloading-and-installing).

Once the `mailsend-go` binary is installed and on the system path, we
can use the new `smtp_send()` function for sending email. The other
function for sending email (`send_email_out()`) will undergo
deprecation.

Currently, only the development version of the package (on **GitHub**)
has the `smtp_send()` function.

### Notes on Sending Email through Gmail

Before using **Gmail** to send out email through **blastula**, there is
a key **Gmail** account setting that must changed from the default
value. We have to allow *Less Secure Apps* to use your the **Gmail**
account. Details on how to make this account-level change can be found
in [this support
document](https://support.google.com/accounts/answer/6010255).

### Installation of the R package

**blastula** is used in an **R** environment. If you don’t have an **R**
installation, it can be obtained from the [**Comprehensive R Archive
Network (CRAN)**](https://cran.r-project.org/).

The **CRAN** version of this package can be obtained using the following
statement:

``` r
install.packages("blastula")
```

You can install the development version of **blastula** from **GitHub**
using the **remotes** package.

``` r
install.packages("devtools")
remotes::install_github("rich-iannone/blastula")
```

If you encounter a bug, have usage questions, or want to share ideas to
make this package better, feel free to file an
[issue](https://github.com/rich-iannone/blastula/issues).

## Code of Conduct

[Contributor Code of
Conduct](https://github.com/rich-iannone/blastula/blob/master/CONDUCT.md).
By participating in this project you agree to abide by its terms.

## License

MIT © Richard Iannone
