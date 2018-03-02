---
id: 555
title: Enabling Poltergeist inspector while using capybara accessible + cucumber rails
date: 2017-04-25T18:51:45+00:00
author: Robert
layout: post
guid: http://wiredtron.com/?p=555
permalink: /2017/04/25/enabling-poltergeist-inspector-while-using-capybara-accessible-cucumber-rails/
categories:
  - Other
---
I know this is a very specific toolset to be using and doesn&#8217;t affect many people, however it took me some time to get it working so it seems that it may be worth documenting. If you are working on a project that uses rails + cucumber + capybara accessible + poltergeist for automated in browser tests and want the ability to use the poltergeist [remote debugging feature](https://github.com/teampoltergeist/poltergeist#remote-debugging-experimental) then you are in luck. Add the following to your `features/support/env.rb`

<pre class="prettyprint" data-start-line="1" data-visibility="visible" data-highlight="" data-caption="">Capybara.register_driver :poltergeist_debug do |app|
  driver = Capybara::Poltergeist::Driver.new(app, inspector: true)
  adaptor = Capybara::Accessible::SeleniumDriverAdapter.new
  Capybara::Accessible.setup(driver, adaptor)
end</pre>

Next update your Capabara default\_driver and javascript\_driver to use your newly registered driver:

<pre class="prettyprint" data-start-line="1" data-visibility="visible" data-highlight="" data-caption="">Capybara.default_driver    = :poltergeist_debug
Capybara.javascript_driver = :poltergeist_debug</pre>

Once the above code is inserted you can then follow the instructions on the poltergeist wiki on how to attach to the remote debugger.