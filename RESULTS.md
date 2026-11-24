# UI lab results

Tool parts that I chose: Part 1: Selenium IDE and Part 3: Robot Framework

## Part 0
- Step 0.1, Chrome version: 157.0.8103.79 (Official Build) (64-bit)
- Step 0.3, ChromeDriver version: ChromeDriver 157.0.8103.78
- Step 0.4, locator of Pizza, Please!: button[name="pizza-please"]
- Step 0.4, locator of Excluded tools: class = "block-text-sm"
- Step 0.4, locator of Must be vegetarian: #default-checkbox
- Step 0.4, locator of Custom Pizza Name: input#pizza-name
- Step 0.5, pizza name, tool and calories of the run by hand: Lab Pizza 20261124-110011, Pizza cutter, 525 calories per slice

| Step | Actual result | Pass or fail |
|------|---------------|--------------|
| 1 | The heading reads Looking to break out of your pizza routine? | Pass |
| 2 | Max Calories per Slice, Excluded tools, Must be vegetarian and Custom Pizza Name appear | Pass |
| 3 | The fields show 600, Knife, vegetarian ticked, and Lab Pizza 20261124-110011 | Pass |
| 4 | The recommendation shows Lab Pizza 20261124-110011, Tool: Pizza cutter (not Knife), and Calories per slice: 525 (<= 600) | Pass |
| 5 | The page says Please log in first. | Pass |
| 6 | The page shows Latest pizza recommendations | Pass |
| 7 | The admin list shows Lab Pizza 20261124-110011 with (tool=Pizza cutter, ingredients_number=7) | Pass |

## Part 1: Selenium IDE
- Firefox version and Node.js version: Firefox 160.0.1, Node.js v24.21.0
- Step 1.4, the line of the failed check and the line Tests:
  - Failed check: Actual value 'Received!' did not match 'Received' during assertText:id=message:Received
  - Tests: Tests: 1 failed, 1 total

- Step 1.5, target chosen for Submit, and why the recorded one was fragile:
  - Target chosen: xpath=//button[@type='submit']
  - Why recorded was fragile: The recorder chose css=.btn, which relies on a CSS class describing visual styling that multiple buttons on the page can share. If another button is placed above it or classes change, the test clicks the wrong element. xpath=//button[@type='submit'] relies on the functional HTML attribute describing the button's action, making it robust against layout and style changes.

- Step 1.6, commands changed, added or deleted after recording:
  - Added waits: waitForElementVisible on css=h1, xpath=//label[normalize-space()='Max Calories per Slice']/../input, css=select, id=default-checkbox, css=input#pizza-name, xpath=//p[starts-with(., 'Name:')], id=rate-result; and waitForElementPresent on xpath=//select/option[normalize-space()='Knife'].
  - Added script & echo: executeScript generating a dynamic timestamped pizzaName (Lab Pizza yyyyMMdd-HHmmss) and echo ${pizzaName}.
  - Added assertions: assertText on heading and recommendation name, assertValue on calories input and pizza name, assertSelectedLabel on css=select, assertChecked on id=default-checkbox, assertNotText on Tool: Knife, storeText for calories, executeScript checking calories <= 600, assert caloriesOk true, and assertText on id=rate-result.
  - Changed: Changed open target to full URL https://quickpizza.grafana.com/, replaced static pizza name with ${pizzaName}, changed tool selection to addSelection with label=Knife, and used stable XPath locators for the Advanced toggle and calories input.
  - Deleted: Redundant click and focus commands captured during manual interaction.

- Step 1.6, the pizza names of run 1 and run 2, from the lines info: echo:
  - Run 1: Lab Pizza 20261124-170510
  - Run 2: Lab Pizza 20261124-170544

- Step 1.6, the line Tests: of both runs:
  - Run 1: Tests: 2 passed, 2 total
  - Run 2: Tests: 2 passed, 2 total

- Step 1.7, one line of QP01customerTest.java that needs rewriting, and why:
  - Line: `driver.findElement(By.xpath("//label[normalize-space()='Max Calories per Slice']/../input")).sendKeys("600");`
  - Why: The input box already defaults to 1000 upon toggling Advanced. Without calling `clear()` first, `sendKeys("600")` appends 600 to 1000, leaving `1000600` in the field instead of `600`.

## Part 3: Robot Framework
- robot --version: Robot Framework 7.5.1 (Python 3.15.0 on win32)
- Step 3.2, failure message for the wrong title: Title should have been 'Web Form' but was 'Web form'.
- Step 3.8, run 1, result, elapsed time and pizza name: PASS, 11.39 s (00:00:11), Lab Pizza 20261124-200611
- Step 3.8, run 2, result, elapsed time and pizza name: PASS, 13.14 s (00:00:13), Lab Pizza 20261124-200624

## Part B
1. All three number boxes on QuickPizza share the duplicate id `floating_filled`, so `#floating_filled` returns 3 elements instead of 1. Because IDs need to be unique, we had to locate the box using its label text with an XPath sibling traversal. Giving the field a unique attribute like `id="max-calories"` would let testers write a clean, direct selector that won't break if the layout changes.

2. This check is a consistency test oracle because it compares the customer recommendation with the admin database view. In our run, Step 4 generated tool `Pizza cutter` and Step 7 confirmed the admin list showed `(tool=Pizza cutter)`. This catches backend persistence bugs, such as the server failing to save the chosen tool or returning stale data from another session.

3. We could read the ingredients list under `#recommendations` and check that it contains no meat items like ham, bacon, or pepperoni. Alternatively, we could check the `/api/pizza` network response to see that `"vegetarian": true`. For pure UI testing, the test would need a known list of non-vegetarian ingredients to compare against, or the page would need a visible "Vegetarian" badge.

4. Selenium IDE required adding individual `waitForElementVisible` commands with fixed 10000ms timeouts, while Robot Framework handled waiting cleanly using keywords with a global `timeout=10s`. Selenium IDE was fast to record but tedious to edit and debug when click commands timed out. Robot Framework took slightly more setup, but writing modular keywords in `quickpizza.resource` felt much cleaner, and its `report.html` with embedded screenshots was far more readable than the console logs of `selenium-side-runner`.

5. I would choose Robot Framework for Project 2. First, its resource files make it easy to use page objects, keeping locators in one place so tests don't break when the UI changes. Second, it generates clear HTML reports and logs with embedded screenshots for run evidence, which makes debugging much simpler.

6. The recorder chose `css=.btn`, which is fragile because it's just a styling class that other buttons can share. If another button is added above it or classes change, the test will click the wrong element. The replacement `xpath=//button[@type='submit']` is more stable because it targets the button's functional role rather than its appearance.

7. I would only need to change line 9 in `quickpizza.resource` where `${MAX CALORIES}` is defined. Because `pizza_scenario.robot` only calls high-level keywords like `Fill In The Pizza Request` and never sees the locator, the test file doesn't need to change at all. This keeps the maintenance small and isolates page changes from test scenarios.
