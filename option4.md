# XPaths

## Learning Goals

- You know how to navigate XPaths
- You are aware of common pitfalls with XPaths

## Introduction

When web development is done in a way that includes plans for test automation,
the creation of automated tests is easy. This is not always happening, and the
result is that web UI elements have no `id` attribute. Aside from unique `id`s,
we could use other attributes like name, and class, or CSS paths. When even
those are not enough, you can still use XPaths.

Locators made with XPath can be good and reliable, but they can be constructed
poorly too. Be cautious in making them, so they will not break often, especially
on websites still under development. You do not want to waste time fixing them.

Tips on working with XPath:

1. Don't use the browser's `Copy XPath` button. They return quite bad locators.
Firefox provides `/html/body/section[1]/nav/div/a` to Bad Flask App's dropdown
menu.

2. Construct the locator with few segments, and use `//` between them, instead
of direct relation. When one of the elements referred to in a segment changes,
or an additional element is put in-between, locator still works.

3. Start locator with `//` rather than `/html/body`. This locator will look
anywhere in the page. If the element is moved further from the `body`, XPath
will still work.

4. Remember to always **test your locator** in the console before adding it to
your test case. You can use JavaScript queries in browser console:
`$x("//path/to/my/element");`

5. Ensure your locator matches **exactly** 1 element. The first element that
matches a given XPath is used even if more were found. If they load in the web
page in a different order, your test will interact with wrong one.

4. If you have same locators used in multiple places, store them in variables,
and use those variables. When XPath breaks, you need to correct it in one place,
and a descriptive variable name helps to figure our what it was pointing to
before.

## Exercise

### Overview

- Implement the logic to close the dropdown _if it's opened_ as part of the test setup.
- Implement the keyword to open the form, also as part of the test setup.
  - Test should pass with or without the dropdown initially opened.
- **Optional:** Investigate what happens if you don't close the dropdown beforehand.
What errors do you see?

### Step-by-step

<details>
  <summary>Implement the logic to close the dropdown if it's open.</summary>

<br />

- Create a variable called `OPENED DROPDOWN` and set it to `//div[contains(@class, 'open')]`
- Implement the keyword called `Close Dropdown If Opened` that clicks the element `OPENED DROPDOWN`.

<details>
  <summary>SeleniumLibrary</summary>

- Use `Run Keyword And Return Status` and `Page Should Contain Element` to check if the dropdown is opened.
Store the result in a variable called `element visible`.
- Using your new variable, use `Run Keyword If` to conditionally close the dropdown.

</details> <!-- SeleniumLibrary -->

<details>
  <summary>Browser</summary>

- Use `Get Element State` with `visible` as the state and store it in a variable.
- Using your new variable, use `Run Keyword If` to conditionally close the dropdown.

</details> <!-- Browser -->

<details>
  <summary>What just happened?</summary>

<br />

As we land on Bad Flask App, we _might_ see a huge dropdown opened
covering the whole website. It opens at random, so there's no knowing whether it
will open in our test case or not. While we're looking at the Bad Flask App, let's
open our developer console by right-clicking anywhere on the screen and selecting `inspect`.
It's a good idea to keep the developer console always opened when you're writing web tests.
We notice, that the dropdown doesn't have an `id` field that would allow us to
easily access that element.

The dropdown button is an
`a` element, which has classes we could use, for example `dropdown-toggle`. However, there's a similar, but hidden `a`
element before in the HTML, so we can't use that `a` alone. Instead, we can use its parent
`div` element to handle the click, as it is the size of the button. Also, it has a class called `open` when the dropdown is opened which disappears when it's closed. So, in other words _if_ the `div` element has a class called `open`, we can click it to close it.

Since our locator is pretty generic, we add it to the `Variables` table in our resource file.
Following Robot Framework's best practices, we give our
variable an UPPER CASE name. As we write more code, we can add next XPaths into the table of `Variables`, every time giving them meaningful names.

</details>

> :bulb: When you click the dropdown in your browser window, there is an additional attribute
> added to the dropdown element: `aria-expanded: "true"` (or `false`). However, using this
> **doesn't** work, since the element doesn't have that attribute when the page is
> initially loaded. It loads the first time the element is clicked.
>
> In this case, we could've also used the `style="display: none;"` attribute of the first
> `a` element to determinate our dropdown element. Yet another way would be to check if the `ul` with class
> `dropdown-menu` is visible in the page, after checking that the page is fully loaded, to avoid
> creating race conditions. Often with XPaths, there is more than "one true answer".

</details> <!-- Implement the logic to close the dropdown if it's open. -->

---

<details>
  <summary>Implement the keyword to open the form, also as part of the test setup.</summary>

<br />

- Add a variable for our `//button` XPath.
- Implement the keyword called `Show Form` which clicks the `//button` element.

- Add `Test Setup` to your `Settings` table and call `Close Dropdown If Opened` and `Show Form`.

> It's possible that your line becomes quite long when you call multiple keywords.
> You can always split your keywords into multiple lines using `...` at the beginning
> of the next line.
>
> E.g.
>
> ```robot
> *** Settings ***
> Test Setup    Run Keywords
> ...           My First Keyword
> ...           AND
> ...           My Second Keyword
> ```

<details>
  <summary>What just happened?</summary>

Now we're able to close the dropdown if it's opened. We still need to show our form.
Again, we don't have an `id` for our element, but luckily the page has only one `button`,
so our XPath is fairly straightforward: `//button`. Again, even though our XPath is short,
it sounds too general, so better add it to the `Variables` table.

Now we have two new keywords: one that closes the dropdown if it is opened and one
that clicks the "Show Form" button. Let's add this to our `Test Setup`. We could
write a wrapper keyword that calls both our new keywords, or we can use the `Run Keywords`
keyword from the BuiltIn library directly. Using `Run Keywords` is a way to group
keywords into a single step if needed. We can link different keywords with `AND` after
each keyword and its parameters.

<br />

</details>

We can still validate our test behaves as expected by running `robot -d output tests/form.robot`.
Our test should open the browser to Bad Flask App, check if the dropdown is opened and close it
when possible, click the "Show Form" button, and finally close the browser.

</details> <!-- Implement the keyword to open the form, also as part of the test setup. -->

---

### Possible Errors

<details>
  <summary>ElementClickInterceptedException</summary>

If you don't close the dropdown you might get an error which says something like this:

```text
ElementClickInterceptedException: Message: element click intercepted: Element <button id="showForm" style="width: 100px; margin: -100 auto 20 auto;">...</button> is not clickable at point (120, 206). Other element would receive the click: <ul class="dropdown-menu" role="menu" aria-labelledby="dLabel">...</ul>
```

This means that you're trying to access an element that is _behind_ another element.
If you try to click the area where the element is, but another element is on top of it, that top
element will receive our click instead, just as if a human was interacting with it. This occurs even
if the top element is completely transparent.

This is common with hover tooltips or menus. Some fields are hidden
behind other elements, and typically you need to close a menu or move your
cursor somewhere else to make the hover go away. For example, some forms
show helpful tooltips, but when a tooltip covers the "Submit" button,
your test execution will fail.

</details>
