# Jenkins-robot-example Repository

Example implementation of a Jenkins pipeline running a RobotFramework test suite.
Purpose is to create a Jenkins pipeline running an integration test suite for
a website, enabling QA testers to write UI tests with a system based
on [Behaviour/Driven Development](https://en.wikipedia.org/wiki/Behavior-driven_development).

The backend uses [Jenkins](https://jenkins.io/), with [Selenium Browser Automation Library](https://seleniumhq.org/) at the core, 
which is run by [Robot Framework](https://robotframework.org/).

### The key components are:
- key element file structured as YAML
- keyword file structured as Python
- directory hierarchy based on test case groups
- test case files with RobotFramework format, stored within the directory structure

## Workflow of QA Tester
The following should give an outline on how to use this system as a QA dude when writing test cases.


1.  Write a test story using BDD principles. (modified example from [BDD Wikipedia page](https://en.wikipedia.org/wiki/Behavior-driven_development))

  > Story: Make sure key website elements are in place after UI updates
  >
  > As a website owner
  > When updating website code
  > I want to make sure appropriate elements are in place
  >
  > Scenario 1: Basic elements should be viewable
  > Given that website is accessible via a browser
  > When opening a page
  > Then Website title should be appropriate
  > And logo image element should exist

2. Create file hierarchy.
  *  Split the story into test scenarios. This will act as Test Case documentation.
  >Group 01: UI
  >Test Case 01: Basic elements should be viewable

  *  Create directory structure ```/tests/group_01/```.
  *  Create an *init file* (```/tests/group_01/__init__.robot```) into the directory. This will contain the group name for test report.
```
*** Settings ***
Documentation    UI Tests
```

3.  Using test keyword documentation, write appropriate Test Cases

```robotframework
*** Settings ***
Documentation	Test Case 01: Basic elements should be viewable
...	Given that website is accessible via a browser
...	When opening a page
...	Then Website title should be appropriate
...	And logo image element should exist
# add other required settings here


${url}	https://mysite.something/		# Site url should be written down somewhere else in the resource hierarchy. Here for convenience only.

Test Setup	log in
Test Teardown	log out

*** Test Cases ***
website is accessible via browser
	[Documentation]	After logging in
	...	address should be correct
	
	address should be	${url}

header elements should exist
	[Documentation]	After logging in
	...	title should exist
	...	logo should exist

	title should exist
	logo should exist

# EOF
```
4. Commit code to the repository.

The example above is not completely applicable to the example code below, but should provide an outline.

## Key Files

### pageName.yaml
This is the element file, which is read at runtime in the python script. The page sholud be named appropriately regarding the source page. The file must also be imported as *Variables* in the *Test Case* file. In this we give names to the appropriate elements, which are described with *[xpaths](https://en.wikipedia.org/wiki/XPath)*.
```yaml
---
page:
  title: 'my page title'
  header:
    logo: //*[@id="company_logo"]
  nav:
    that_link: //*[@class="navigation"]/a[contains(@title, 'That link')]
  content:
    header: //*[@class="content_header"]
    author: //*[@class="author_info"]
  footer:
    copyright: //*[@id="copyright_info"]

```

### pageName.py
The keyword file. This file should contain all actions and assertations, which the USERS are to apply when writing test cases.

```python
__version__ = '0.1'

import yaml
import robot.libraries.XML
from robot.libraries.BuiltIn import BuiltIn
from SeleniumLibrary import SeleniumLibrary
from robot.api.deco import keyword

class pageName(SeleniumLibrary):
    def open_yaml_file(self, file):
        page = open(file)
        data = yaml.load(page)
        self.site = data['page'] # the 'page' name is the highest (root) YAML element in the YAML file.
	return data

    @keyword    # this is a robotframework keyword, which can be called using .robot file
    def click_a_link_to_that_page(self):
	self.file = self.open_yaml_file('pageName.yaml')
	self.page = self.file['page']                     # open the root of yaml data
	self.click_element(self.site['nav']['that_link']) # this data is relative to the root selection

    @keyword
    def title_should_exist(self):
	self.file = self.open_yaml_file('pageName.yaml')
	self.page = self.file['page']                     # open the root of yaml data
	self.title_should_be(self.page['title'])

    @keyword
    def logo_should_exist(self):
	self.file = self.open_yaml_file('pageName.yaml')
	self.page = self.file['page']                     # open the root of yaml data
	self.page_should_contain(self.page['header']['logo'])
```

### group_01/case_01.robot
This is the actual test file. Test cacses are run in the *alphabetical order*, so numbering of test cases should be appropriate.
Also, when following the test log, and reading the reports later on, the **directory name** with the **init file label** are displayed with every test case. This makes reading easier.

```robotframework
*** Settings ***
Documentation	Test case 01.
...		This contains several tests per se.
...		All resources are in the upper directory.
Library		../pageName.py
Variables	../pageName.yaml
Resource	../resource.robot    # A general resource file. Would contain universal keywords such as 'log in'. Here as an example only.
Default Tags	noncritical          # Noncritical test cases will not break the test run if they fail. This is a matter of convenience.

Suite Setup	Start Browser	${target}	${browser}
Seuite Teardown	Close Browser

Test Setup	log in
Test Teardown	log out

*** Test Cases ***
this thing looks fine
	[Documentation]	Checks this thing. (the description is actually the test description)
	...	Given I log in to the service
	...	And click a link to a certain page
	...	Then title should be 'mypage'
	...	And logo ID should exist

	# we log in to the page in the test setup, which is run for every test case
	click a link to a page
	click a link to that page
	title should exist
	logo should exist
```


		

