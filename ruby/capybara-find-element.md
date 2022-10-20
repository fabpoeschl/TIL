# How to find a (hidden) element in Capybara tests
1. Find element id in Chrome developer tools (see [here](../chrome/devtools-show-all-elements.md))

2. Show hidden elements by adding `Capybara.ignore_hidden_elements = false` to test helper or setting it in debug console.

3. Find element in Capybara test with `page.find(<element id>)`.
