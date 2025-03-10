---
title: Functional Testing
summary: Test controllers, forms and HTTP responses.
---

# Functional Testing

[FunctionalTest](api:SilverStripe\Dev\FunctionalTest) test your applications `Controller` logic and anything else which requires a web request. The 
core idea of these tests is the same as `SapphireTest` unit tests but `FunctionalTest` adds several methods for 
creating [HTTPRequest](api:SilverStripe\Control\HTTPRequest), receiving [HTTPResponse](api:SilverStripe\Control\HTTPResponse) objects and modifying the current user session.

## Get

```php
$page = $this->get($url);
```

Performs a GET request on $url and retrieves the [HTTPResponse](api:SilverStripe\Control\HTTPResponse). This also changes the current page to the value
of the response.

## Post
```php
$page = $this->post($url);
```

Performs a POST request on $url and retrieves the [HTTPResponse](api:SilverStripe\Control\HTTPResponse). This also changes the current page to the value
of the response.

## Other Requests
```php
$page = $this->sendRequest('PUT', $url);
```

Performs a request on $url with the HTTP method provided (useful for PUT, PATCH, DELETE, etc.). This also changes the current page to the value of the response.

## Submit


```php
$submit = $this->submitForm($formID, $button = null, $data = []);
```

Submits the given form (`#ContactForm`) on the current page and returns the [HTTPResponse](api:SilverStripe\Control\HTTPResponse).

## LogInAs


```php
$this->logInAs($member);
```

Logs a given user in, sets the current session.

When doing a functional testing it's important to use `$this->logInAs($member);` rather than simply `Security::setCurrentUser($member);` or `$this->session()->set('loggedInAs', $member->ID);` as the latter two will not run any logic contained inside login authenticators.

## LogOut

Log out the current user, destroys the current session.

```php
$this->logOut();
```

## Assertions

The `FunctionalTest` class also provides additional asserts to validate your tests.

### assertPartialMatchBySelector


```php
$this->assertPartialMatchBySelector('p.good',[
    'Test save was successful'
]);
```

Asserts that the most recently queried page contains a number of content tags specified by a CSS selector. The given CSS 
selector will be applied to the HTML of the most recent page. The content of every matching tag will be examined. The 
assertion fails if one of the expectedMatches fails to appear.


### assertExactMatchBySelector


```php
$this->assertExactMatchBySelector("#MyForm_ID p.error", [
    "That email address is invalid."
]);
```

Asserts that the most recently queried page contains a number of content tags specified by a CSS selector. The given CSS 
selector will be applied to the HTML of the most recent page. The full HTML of every matching tag will be examined. The 
assertion fails if one of the expectedMatches fails to appear. 

### assertPartialHTMLMatchBySelector

```php
$this->assertPartialHTMLMatchBySelector("#MyForm_ID p.error", [
    "That email address is invalid."
]);
```

Assert that the most recently queried page contains a number of content tags specified by a CSS selector. The given CSS 
selector will be applied to the HTML of the most recent page. The content of every matching tag will be examined. The 
assertion fails if one of the expectedMatches fails to appear.

[notice]
`&nbsp;` characters are stripped from the content; make sure that your assertions take this into account.
[/notice]

### assertExactHTMLMatchBySelector
```php
$this->assertExactHTMLMatchBySelector("#MyForm_ID p.error", [
    "That email address is invalid."
]);
```

Assert that the most recently queried page contains a number of content tags specified by a CSS selector. The given CSS 
selector will be applied to the HTML of the most recent page.  The full HTML of every matching tag will be examined. The 
assertion fails if one of the expectedMatches fails to appear.

[notice]
`&nbsp;` characters are stripped from the content; make sure that your assertions take this into account.
[/notice]

## Related Documentation

* [How to write a FunctionalTest](how_tos/write_a_functionaltest)

## API Documentation

* [FunctionalTest](api:SilverStripe\Dev\FunctionalTest)
