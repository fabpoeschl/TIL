# How to debug a browser test using Capybara and Cuprite
Javascript components usually only have unit tests, therefore a browser test covering all Javascriptcomponents
components and Rails code is helpful to verify a feature.

However, finding bugs in Javascript from a browser test can be a real nightmare, since the debug output will
not show any browser data.

Consequently, to debug such a test the browser should run in headful mode, so that console logs can be analyzed
in the developer console.

```
it "tests something" do
  Capybara.register_driver :cuprite_headed do |app|
    Capybara::Cuprite::Driver.new(
      app,
      headless: false,
      window_size: [1200, 800]
    )
  end

  Capybara.using_driver(:cuprite_headed) do
    visit some_url
    ...
  end
end
```

Alternatively use console.log to print data in the browser console. Using headless mode the console ouput can be printed with SELENIUM_DRIVER_LOG_LEVEL=debug.
