---
title: Using chrome debug tools to debug flaky Capybara system tests
---

Chrome debug tools are super useful when trying to debug an issue while running Capybara system tests. In order to debug this failing test I really wanted the Chrome devtools Network tab open in order to see what requests were going between the server and browser. Unfortunately in my case the tests had to be run many times in order to reproduce the failure, and the Chrome network tab only begins logging requests after it is opened. This means unless the Network tab was opened right when the test began there was no way to see the traffic I was curious about. In order to achieve the desired result I ended up registering a new Capybara driver with the following config:

```
Capybara.register_driver :chrome_with_network_tab do |app|
  options = Selenium::WebDriver::Chrome::Options.new(args: %w[auto-open-devtools-for-tabs])
  options.add_preference(
    'devtools',
    'preferences' => {
      'currentDockState' => '"bottom"',
      'panel-selectedTab' => '"network"',
    }
  )

  Capybara::Selenium::Driver.new(
    app,
    browser: :chrome,
    options: options,
  )
end
```

I then ran my failing test in a loop until it failed:

```
while true; do                                                                    
    bundle exec rspec ./spec/system/spec_that_fails_spec.rb:<line number of failing spec>
    if [ $? -ne 0 ]; then
        break
    fi
    sleep 1
done
```

At this point I was able to see all of the network requests and was able to narrow down the problem. If you prefer a different dock location or need a different active tab, feel free to change the setting to fit your needs.
