---
title: Setting up a Content Security Policy for Ruby on Rails Applications
author: Robert
layout: post
id: 574
---

A Content Security Policy(CSP) is an important piece of mitigating multiple different types of attacks including XSS, Clickjacking, and other various types of code injections attacks. It does this by informing the end-users browser to not make requests or perform actions that don't meet the CSP. Unfortunately, writing a generic CSP is not possible, and instead requires the developer to analyze their requirements and make appropriate exceptions before rolling out a CSP. In this article I will cover two sections: overall observations, and examples of Ruby on Rails specific gotcha's I encountered along the way.

# Overall Observations

My first step in writing a CSP was to enable the default one which Rails provides, and inspect what errors I received on the console. I was greeted with a few errors right away that looked very strange, which I quickly realized were not related to the application. They were caused by extensions which I had installed, specifically Vue Devtools and React Devtools. This lead me to my first takeaway: **Use Private Browsing mode when testing your CSP** to ensure the errors you are seeing are relevant!

Second, I noticed that the **CSP messages in Firefox were not quite as helpful as the ones in Chrome**. I ended up having a lot more luck tracking down the root cause of CSP issues in Chrome than in Firefox.

# Rails-Specific Observations

There are 4 gems and 1 javascript code block I had to refactor as part of this exercise. I will include a link to my full set of changes at the end of the article since this was done on an open source project. Note that testing changes to your CSP in rails is slightly annoying since you need to reload the server after every change, since the CSP is saved in an initializer. If you feel like your changes aren't taking effect, this is probably the reason.

### Gems
* [Bootstrap 3](#bootstrap-3)
* [jquery-rails](#jquery-rails)
* [Chartkick](#chartkick)
* [reCAPTCHA](#recaptcha)

### Code Refactoring
* [Rewriting Inline Scripts](#rewriting-inline-scripts)

## CSP Settings

The default CSP in rails looks as follows (any items commented out were done by me):

```ruby
# Be sure to restart your server when you modify this file.

# Define an application-wide content security policy
# For further information see the following documentation
# https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy

Rails.application.config.content_security_policy do |policy|
  policy.default_src :self, :https
  policy.font_src    :self, :https, :data
  policy.img_src     :self, :https, :data
  policy.object_src  :none
  policy.script_src  :self, :https
  policy.style_src   :self, :https

# Specify URI for violation reports
# policy.report_uri "/csp-violation-report-endpoint"
end

# If you are using UJS then enable automatic nonce generation
Rails.application.config.content_security_policy_nonce_generator = -> request { SecureRandom.base64(16) }

# Set the nonce only to specific directives
Rails.application.config.content_security_policy_nonce_directives = %w(script-src)

# Report CSP violations to a specified URI
# For further information see the following documentation:
# https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only
# Rails.application.config.content_security_policy_report_only = true

```

In this application, `//= require jquery_ujs` was included, so I went ahead and enabled the `content_security_policy_nonce_generator` feature. The only caveat was I also had to enable the `content_security_policy_nonce_directives` setting and limit it to only apply to `script-src` since all of the Chartkick inline styles broke and there was no straightforward way to pass the nonce to stylesheets.

## Nonce Helpers

First of all, what is a nonce? A nonce is a one-time-use code which is changed every time the page is refreshed. 

With the settings in the previous section enabled, it was necessary to modify the code in a few places to use the nonce. Specifically the following was the diff of the lines change in my `application.html.haml`:

```diff
+ = csp_meta_tag
- = javascript_include_tag "application"
+ = javascript_include_tag "application", nonce: true
```

The `csp_meta_tag` adds the a new tag to every page, and the `nonce: true` adds that same nonce to the javascript assets tag, for example: 
```html
<meta name="csp-nonce" content="Zp+yENSM5K9BKStdB22IbQ==" />
<script src="/assets/application-1798ca4812645a78f896665e43c690d14f965504736ca9fe705e2d59cce6d622.js" nonce="Zp+yENSM5K9BKStdB22IbQ=="></script>
```

This same nonce is accessible through the `content_security_policy_nonce` method and an example is used later on in the file.
# Gems
## Bootstrap 3

Bootstrap adds an inline style to navbar dropdowns. In my case the line:

```haml
%button.navbar-toggler{"aria-controls" => "navbarSupportedContent", "aria-expanded" => "false", "aria-label" => "Toggle navigation", "data-target" => "#navbarSupportedContent", "data-toggle" => "collapse", :type => "button"}
```

was throwing an error stating that the inline style 

```
color: #fff
```

was not able to be applied. I was not applying a custom style to the navbar with an inline style, so this must be something that Bootstrap 3 adds on. Chartkick (below) also requires inline styles enabled, so I fixed this by ensuring the following was set in my CSP.

```ruby
policy.style_src   :unsafe_inline, :self, :https
```

## jQuery Rails

This application was relying on an old version of jQuery without realizing it. In our application.js there was an include for `//= require jquery`. This was causing an error inside jQuery itself on the following lines:

```
// Support: IE<9 (lack submit/change bubble), Firefox (lack focus(in | out) events)
for ( i in { submit: true, change: true, focusin: true } ) {
  eventName = "on" + i;

  if ( !( support[ i ] = eventName in window ) ) {

    // Beware of CSP restrictions (https://developer.mozilla.org/en/Security/CSP)
    div.setAttribute( eventName, "t" );
    support[ i ] = div.attributes[ eventName ].expando === false;
  }
}
```

It turns out the `jquery-rails` gem includes `jQuery` 1, 2, and 3. I found that switching to `//= require jquery2` or `//= require jquery3` fixed the issue.

## Chartkick

Chartkick has a really good writeup on what is required to make it work with CSP: [chartkick CSP guide](https://github.com/ankane/chartkick/blob/master/guides/Content-Security-Policy.md)

In my case, I already had to enable `unsafe_inline` styles across the whole application due to Bootstrap 3, so this setting in conjunction with adding `Chartkick.options[:nonce] = true` to `config/initializers/chartkick.rb` solved all of my Chartkick issues.

## reCAPTCHA

The documentation for reCAPTCHA states that [whitelisting a few domains](https://developers.google.com/recaptcha/docs/faq#im-using-content-security-policy-csp-on-my-website.-how-can-i-configure-it-to-work-with-recaptcha) is required in order to allow reCAPTCHA to work with CSP enabled. This does not reflect what I experienced. I added `invisible_recaptcha_tags nonce: content_security_policy_nonce` (note that I am using the [ambethia/recaptcha](https://github.com/ambethia/recaptcha) gem and afterwards reCAPTCHA worked fine for me. The nonce used here pulls in the same nonce that is added as described in the nonce helpers section above.

# Code Refactoring

## Rewriting inline scripts

There was one instance of an inline script which used data from the controller to show the time remaining countdown. In order to still pass the data needed into Javascript, I had to add the data to a `data` attribute. The transformation can be seen from [here](https://github.com/mitre-cyber-academy/ctf-scoreboard/commit/b61f723cd1145b15b08101eab7c0fea2be5a572c#diff-23492f8696d5e59d70a27c592a1e7172efcb3e881e36436687456415bc857c88L5-R3) to [here](https://github.com/mitre-cyber-academy/ctf-scoreboard/commit/b61f723cd1145b15b08101eab7c0fea2be5a572c#diff-385d298b11967d2164b9b0889f3dd8b3c6eb7a11afacbf6b5ae2026ded30ff92R4-R33).

## Resulting CSP

With all of my investigation completed, I found the following CSP worked best for the application:

```ruby
# Be sure to restart your server when you modify this file.

# Define an application-wide content security policy
# For further information see the following documentation
# https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy

Rails.application.config.content_security_policy do |policy|
  policy.default_src :self, :https
  policy.font_src    :self, :https, :data
  policy.img_src     :self, :https, :data
  policy.object_src  :none
  policy.script_src  :self, :https
  policy.style_src   :unsafe_inline, :self, :https
end

# If you are using UJS then enable automatic nonce generation
Rails.application.config.content_security_policy_nonce_generator = -> request { SecureRandom.base64(16) }

# Set the nonce only to specific directives
Rails.application.config.content_security_policy_nonce_directives = %w(script-src)

```

Relevant commit: [mitre-cyber-academy/ctf-scoreboard](https://github.com/mitre-cyber-academy/ctf-scoreboard/commit/b61f723cd1145b15b08101eab7c0fea2be5a572c)