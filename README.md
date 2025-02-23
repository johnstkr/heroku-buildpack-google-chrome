# this is essentially herokus headless chrome buildpack without the --remote-debugging-port=9222 flag shim'd in since it conflicts with screenshots and dom dump.


# heroku-buildpack-google-chrome

This buildpack downloads and installs (headless) Google Chrome from your choice
of release channels.

While headless Chrome is stable, some use cases (like filling in fields via
Selenium) require an X window server to be active. For those cases, please
see the [heroku-xvfb-google-chrome buildpack](https://github.com/heroku/heroku-buildpack-xvfb-google-chrome)
instead.

## Channels

You can choose your release channel by specifying `GOOGLE_CHROME_CHANNEL` as
a config var for your app, in your app.json (for Heroku CI and Review Apps),
or in your pipeline settings (for Heroku CI).

Valid values are `stable`, `beta`, and `unstable`. If unspecified, the `stable`
channel will be used.

## Shims and Command Line Flags

This buildpack installs shims that always add `--headless`, `--disable-gpu`, 
`--no-sandbox` to any `google-chrome` 
command as you'll have trouble running Chrome on a Heroku dyno otherwise.

You'll have two of these shims on your path: `google-chrome` and
`google-chrome-$GOOGLE_CHROME_CHANNEL`. They both point to the binary of
the selected channel.

## Selenium

To use Selenium with this buildpack, you'll also need Chrome's webdriver.
This buildpack does not install chromedriver, but there is a
[chromedriver buildpack](https://github.com/heroku/heroku-buildpack-chromedriver)
also available.

Additionally, chromedriver expects Chrome to be installed at `/usr/bin/google-chrome`,
but that's a read-only filesystem in a Heroku slug. You'll need to tell Selenium/chromedriver
that the chrome binary is at `/app/.apt/usr/bin/google-chrome` instead.

To make that easier, this buildpack makes `$GOOGLE_CHROME_BIN` available as
an environment variable. With it, you can use the standard location
locally and the custom location on Heroku. An example configuration for Ruby's
Capybara:

```
chrome_bin = ENV.fetch('GOOGLE_CHROME_BIN', nil)

chrome_opts = chrome_bin ? { "chromeOptions" => { "binary" => chrome_bin } } : {}

Capybara.register_driver :chrome do |app|
  Capybara::Selenium::Driver.new(
     app,
     browser: :chrome,
     desired_capabilities: Selenium::WebDriver::Remote::Capabilities.chrome(chrome_options)
  )
end
```
