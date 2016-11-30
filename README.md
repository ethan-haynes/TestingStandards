# Test Engineer BEST PRACTICES for Quality Assurance

## Testing Levels
Tests should be broken up into three levels:
```
  1. Unit Testing

  2. Integration Testing

  3. End to End Testing
```
#### Testing Pyramid
![Alt text](https://2.bp.blogspot.com/-YTzv_O4TnkA/VTgexlumP1I/AAAAAAAAAJ8/57-rnwyvP6g/s1600/image02.png "Google Testing Blog Pyramid")
<br/>
[Google Testing Blog Pyramid](https://2.bp.blogspot.com/-YTzv_O4TnkA/VTgexlumP1I/AAAAAAAAAJ8/57-rnwyvP6g/s1600/image02.pn)
<br/>

This stucture for testing is borrowed from the 2014 Google Test Automation Conference. It outlines where the bulk of tests coverage should be handled. Unit tests should be the most abundant, and have the greatest test coverage. `Integration Tests` should have greater coverage than `E2E Testing`, but less than the total coverage of `Unit Tests`.

## Unit Testing
`Unit Testing` is a component level testing pattern. In `OOP` (Object-Oriented Programing) it focuses on testing classes. In `Functional Programing` it tests a function's input against its output. Unit tests should be constructed in such a way that modular. One technique used to accomplish this is `Dependency Injection`.

#### Without Dependency Injection
```javascript
  // global data
  var testData = [1,2,3,4]
  testFunction( )

  function testFunction( ) {
    // modifying global data
    testData = testData.map(function( x ) {
      return x * 3
    })
  }
```
Testing using Karma
```javascript
describe('modifies an existing array of data', function() {
  it('multiplies all indices by a multiple of 3', function() {
    var testData = [1,2,3,4]
    testFunction( testData )
    expect( testData ).toEqual( [3,6,9,12] )
  })
})
```
Version in Python
```Python
testData = [1, 2, 3, 4]


def test_function(self):
        """
        modifies an existing array of data
        to multiply all indices by a 3
        """
        return map((lambda x: x * 3), testData)

testData = test_function()
```
Testing using Python (unittest)
```Python
def test_function(self):
        """
        tests that function modifies an existing array
        of data to multiply all indices by a 3
        """
        var testData = [1, 2, 3, 4]
        testFunction()
        self.assertEqual(testData, [3, 6, 9, 12])
```

#### With Dependency Injection
```javascript
  // global data
  var testData = [1,2,3,4]
  var newData = testFunction( testData )

  function testFunction( data ) {
    // modifying local data
    var newData = data.map(function( x ) {
      return x * 3
    })

    return newData
  }
```
Testing using Karma
```javascript
describe('makes a new array of data', function() {
  it('multiplies all indices by a multiple of 3', function() {
    var testData = [1,2,3,4]
    var newData = testFunction( testData )
    expect( newData ).toEqual( [3,6,9,12] )
  })
})
```

Version in Python
```Python
testData = [1, 2, 3, 4]


def test_function(self, data):
        """
        modifies an existing array of data
        to multiply all indices by a 3
        """
        return map((lambda x: x * 3), data)

newData = test_function(testData)
```
Testing using Python (unittest)
```Python
def test_function(self):
        """
        tests that function modifies an existing array
        of data to multiply all indices by a 3
        """
        var testData = [1, 2, 3, 4]
        newData = testFunction(testData)
        self.assertEqual(newData, [3, 6, 9, 12])
```

### Positional Dependencies
<br/>
[Connascence](https://en.wikipedia.org/wiki/Connascence_(computer_programming)) is a metric of code complexity caused by its dependencies. It is attributed to Meilir Page-Jones. Connascence of Position is when the components that we write, whether those are functions, methods, classes, etc., conform to a specific order of value or implementation. This order must be agreed upon by the caller and the callee, and if there is a change in the structure it will cause bugs that will not be found by unit tests alone. When functions know about the data outside of their scope it can create positional dependencies that are hard to test which can slow future feature development.

Say we take our first example and tweak it to show positional dependency.
```javascript
  // global data
  var testData = [1,2,3,4]
  testFunction( )
  someService( )

  function testFunction( ) {
    // modifying global data
    testData = testData.map(function( x ) {
      return x * 3
    })
  }
  function someService( ) {
    $.ajax({
      type: "POST",
      url: $BASE_API_URL,
      data: testData,
      dataType: "json",
      success: function( data ) {
        handleResponse( data ) // handle response from service endpoint
      },
      error: function( error ) {
        console.error( error )
      }
    })
  }
```
Our code has a dependency on the global testData and someFunction has a positional dependency on testFunction running just before it is called. This becomes and issue when something like this happens:
```javascript
  // global data
  var testData = [1,2,3,4]
  testFunction( )
  someOtherEvilFunction( ) // other developer needs the data this way for their service
  someService( ) // our service is called after the data has been mutated

  function someOtherEvilFunction( ) {
    // modifying global data
    testData = testData.reduce(function( x, y ) {
      return x - y
    })
    //... does stuff with the data
  }
```
Inside of someOtherEvilFunction the data in our global test object is mutated. This will cause a bug that our Unit Tests alone will not be able to find. Instead they could do something like this:
```javascript
  // global data
  var testData = [1,2,3,4]

  var newData = testFunction( testData )
  someService( newData ) // our service is called

  var theirData = someOtherNiceFunction( testData ) // other developer needs the data this way
  theirServiceCall( theirData )
```
Organized this way, both development teams can retrieve the data they need for their service and organize it in whatever way they need for the application without compromising the data model.

#### Function vs Method
If a function must know about data outside of its scope without having the data passed then it should be a method and the data model should be organized as a class (therefore binding it to the object who's data model it needs to know about). Functions, when implemented, should be stateless to create code reusability and reduce mutability.

### Mocking
Mocking is a technique to mimic the behavior of certain components of the application for the purpose of isolating a unit of code. An object or component of code might have a dependency that is hard to test in isolation, particularly if it need to communicate with an outside system.

#### Mocking With Unit Tests
Testing using Karma:
In this example bar has a dependency on a response from a service. That service goes off to a separate system, but Bar does not need to know that functionality works when it is under test. It just needs to receive a response from that service and handle it. We can mock a stub function to generate the minimal information needed for our Mock object to return the response Bar is expecting.
```javascript
import { Foo, Bar } from "my-module"
import { Service } from "my-mock-service"

describe('captures a response from a service', function() {
  it('should equal success when Bar captures the response', function() {
    var foo = Foo({ name: "tester", payment: 200.00 }) // Foo Stub
    var response = service( foo ) // Mock Service
    var newData = Bar( response )

    expect( newData ).toEqual({ "response" : "success" })
  })
})
```

Mocking example using python and nose
```Python
# Standard library imports...
from unittest.mock import Mock, patch

# Third-party imports...
from nose.tools import assert_is_none, assert_list_equal

# Local imports...
from project.services import get_some_service
from Bar import some_class


@patch('project.services.requests.get')
def test_getting_service_when_response_is_ok(mock_get):
    service = [{ "response" : "success" }]

    # Configure the mock to return a response with an OK status code. Also, the mock should have
    # a `json()` method that returns a list of todos.
    mock_get.return_value = Mock(ok=True)
    mock_get.return_value.json.return_value = service

    foo = Foo({ name: "tester", payment: 200.00 })
    # Call the service, which will send a request to the server.
    response = get_some_service(foo)
    newData = Bar(response.json())

    # If the request is sent successfully, then I expect a response to be returned.
    assert_list_equal(newData, { "response" : "success" })
```

#### Summary
Dependency Injection allows us to:
      * remove knowledge of service object's data model/ implementation details
      * reduce coupling between classes, modules, etc
      * remove complexity from classes and focuses instead on class interactions
Mocking is a good way to:
      * Test a component that has complex dependencies
      * Mimic the behavior of an outside system for the purpose of isolation

## Integration Testing

Integration testing is where we have to combine multiple units together to test them as a group. The purpose of a good integration test is measuring if your units of code are interacting correctly.

#### Better Units Make for Better Integration
Having properly written units makes for less setup and ultimately more efficient integration tests.
Here is an example:

Integration Test using Mocha
```javascript
var should = require( 'chai' ).should()
var expect = require( 'chai' )
var supertest = require( 'supertest' )
var api = supertest( $BASE_API_URL )

describe('api', function() {
  it('should list ALL locations with a siteId of 1', function( done ) {

    api.get( '/some/site/locations' )
    .set( 'Accept', 'application/json' )
    .send({
      siteId: 1
    })
    .expect( 200 )
    .end( function( err, res ) {
      expect( res.body ).to.have.property( "location" )
      expect( res.body.location ).to.not.equal( null )
      .done()
    })
  })
})
```

There is no business logic in this test. Why is that? Because Integration Tests test the integration of multiple units. If my units are working correctly this code should execute without error. If there is an error in my business logic, my unit tests should flush that out. If there is an error in my Integration Test, that more than likely involves an environmental issue, or that there is a flaw in the integration of my units of code.

## End to End Testing (E2E)
End to End Tests are well written if they focus on areas of code that are unable to get proper test coverage from smaller tests, such as Integration and Unit Tests. E2E tests should correspond to use cases, though it should not be limited to just use cases. They should focus on system behavior, and not lower-level implementation details that might change over time.

E2E Test with NightWatchJS:
```javascript
module.exports =
  'Demo test API' : function ( client ) {
    client
      .url( $BASE_API_URL )
      .waitForElementVisible( 'body', 1000 )
      .assert.title('Test API' )
      .assert.visible( 'input[type=text]' )
      .setValue( 'input[type=text]', 'Site Location 1' )
      .waitForElementVisible( 'button[name=btnG]', 1000 )
      .click( 'button[name=btnG]')
      .pause( 1000 )
      .assert.containsText( '#site-location',
        'Location - 1' )
      .end( )
  }
}
```

Example using Python and selenium:
```Python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

import unittest

# Local imports...
from project.constants import BASE_API_URL

class ApiTestCase(unittest.TestCase):

    def setUp(self):
        self.browser = webdriver.Firefox()
        self.addCleanup(self.browser.quit)

    def test_page_title(self):
        self.browser.get(BASE_API_URL)
        self.assertIn('Test API', self.browser.title)

    def test_button_on_page(self):
        self.browser.get(BASE_API_URL)
        self.browser.implicitly_wait(10)

        sl_element = self.browser.find_element_by_id("site-location")

        if sl_element.is_displayed():
          sl_element.click()

        self.browser.implicitly_wait(10)
        self.assertEquals('Location - 1', sl_element.text)

if __name__ == '__main__':
    unittest.main(verbosity=2)
```

#### Considerations
E2E tests are much larger and cover a wider gamut than their test companions. They are also slower to run an entire suite. E2E tests should test system behavior of groups of integrated components that have already been tested with Integration Tests. Additionally, E2E tests that go too far in their coverage or test implementation details too low in the architecture run the risk of being incredibly brittle and hard to maintain.  

### Other Testing types
#### System Testing
The interconnection of multiple systems.
#### Operational Acceptance Testing
The testing of non-functional features such as backup, operational stability
