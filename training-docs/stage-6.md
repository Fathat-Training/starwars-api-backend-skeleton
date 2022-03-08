# starwars-api-backend-skeleton

---

### Backend API Learning Workflow:

---
### Testing:

When we write code we need to check that it's correct, it does what it is supposed to do and handles cases when it can't.
In order to do this we test our code. There are many techniques for testing code. Here

1. Write a bunch of calls to various functions that will test each part of the code you have written.
2. Use Python Unittests or other testing framework to write and run tests.
3. Use people or an automated system such as Jenkins to automatically run tests on your system.

In this project we shall develop a set of unittests for various parts of the code we have put in place.

Before we start, there are a few issues to discuss regarding testing both APIs and anything that touches a database.

Testing APIs is not as straight forward as simply testing the odd function or two. You have to test the API Specification,
the http interface between that and the APi endpoints. In our case, we also have a data access layer which in the user module and this
touches the MySQL database. 

Fortunately in our case our API Specification is validated by 'Connexion' and all the handling of our http requests and responses
between the API Specification and endpoints is taken care of. However, connexion will not validate or test our endpoint code or indeed our 
data access layers. Regardless, we can test the whole flow from request to response of each of our endpoints manually via the Swagger UI. This is cool because
it allows anyone outside the immediate tech team to try out the API calls without any knowledge if the internal tech. Having this UI for both documenting the API and testing it is very useful for
providing access to an API for different user bases (All those non-tech people out there that may want to use the service). But, as far as testing is concerned, it is not enough.

Ideally we would create a bunch of tests which use something called Mocking. Mocking is a form of introducing artificial code and data, i.e. Mocking an API call and access to the database without
actually calling the actual API call and touching the real database. The deal is not to have to deal with external links in the chain, just the code that handles any of those dependencies.

For this project, we will avoid getting into complete endpoint to data access layer testing but will test some of our utility functions, our starwars.py code which
calls the external Star Wars API and our authentication module.  We will achieve this by developing some basic python unittests.

Let's start with our `utils.py` file:

### Testing Utils.py

In this file we have a few functions that we are going to write tests for. The `test_utils.py` file in the root of the project is currently an empty file.

>Open that file

>Copy the following imports into the file:

```python
from unittest import TestCase
from utils import dict_excludes, dict_sort, read_json_file
```

Two of those imports should be familiar, but not the `read_json_file` import. That's because it does not exist yet. So let's get that into the utils.py file in the root directory.

>Open `utils.py` and place the following python code before the other functions but below the imports.

```python
def read_json_file(json_file):
    import json

    # Opening JSON file
    with open(json_file) as jf:
        # returns JSON object as
        # a dictionary
        data = json.load(jf)

    return data
```

The above function reads a file and loads the data from the file in json format. We could dress this function up more and make sure we cope with 
data that cannot be formatted to Json, but that is for later. For now, we'll keep it simple.

Back to our `test_utils.py` file into which we copied the imports.

>Notice that the first import is `unittest` and the class from that called TestCase. TestCase is the basic unittest class for testing which allows for the `unitests` to be
run. 

>For a full explanation visit [Python Unittests](https://docs.python.org/3/library/unittest.html)

Ok, so now we have our `unitests` and it's base class imported we can set up our testing subclass. It's pretty simple! 

>Copy the code below to the `test_utils.py` file.

```python
class Test(TestCase):
```
As you can see it inherits the class TestCase.

Now we can write or in this case copy our first test. This will be to test our `dict_excludes` function in utils.py. It's imported already.

```python
def test_dict_excludes(self):
    result = dict_excludes(test_dict, excludes)
    assert excludes_result == result
```
As can be seen it's a simple test that calls the function `dict_excludes` with two parameters, a dictionary and a list of keys to be excluded from
that dictionary in a newly created returned dictionary, i.e. `result`. We are then comparing the result with a variable called `excludes_result`. 

Before we can run the test we need to define the dictionary and our list of excludes and the excludes_result variable. We'll do this by including the following code above the 
class but below the imports.

>Copy the following code

```python
test_dict = {'one': 1, "five": 5, "ten": 10, "two": 2, "fire": "fire", "bird":"Parrot"}
excludes = ["ten", "fire", "two"]
excludes_result = {'one': 1, "five": 5, "bird": "Parrot"}
```

There we have the variables assigned to data structures ready for our test.

**Running the test**

There are several ways tests can be run:

1. From an IDE, such as PyCharm. You can run the tests directly from the test file itself by just clicking on the green play icon, next to the test.
2. from the command line as follows:

```bash 
 python -m unittest discover
```
Make sure you are in the project root directory when you run this command line.

>Try it out and run the test now and see what happens. It should report 1 Test run - OK or similar.

If it fails check that you have copied the code correctly and try again.

Now let's quickly copy the other two tests for this test batch.

```python
def test_dict_sort_ascending(self):
    data = read_json_file('starwars-backup-data/films.json')
    data_sorted = dict_sort(data['results'], "title", "ascending")
    assert data_sorted[0]['title'] == 'A New Hope'

def test_dict_sort_descending(self):
    data = read_json_file('starwars-backup-data/films.json')
    data_sorted = dict_sort(data['results'], "title", "desc")
    assert data_sorted[0]['title'] == 'The Phantom Menace'
```

The above tests test the dictionary sort function, which takes Json data from a file in this case `starwars-backup-data/films.json` and 
passes the results list of that data as the first parameter, followed by the string we wish to sort by and the order. There is a test for sorting
in ascending order and one for descending order.

>Copy those tests and place them in your test class below the last test and run the tests again.

We have created our first unittests for this project.

### Exercise:

>There was one function we did not test in the `utils.py` file. The `options_filter` function. Write a test for that function.

Once you have completed that function here are some more tests to implement:

### Testing the starwars.py API requests:

Copy the following code to the test_`starwars.py` file, study what they are doing and run them.

```python
from unittest import TestCase
from starwars import StarWars
from utils import read_json_file


class TestStarWars(TestCase):
    """
        Tests the two main Starwars api request types, the async one with and without batch and max sizes
    """
    def test_request_data_sync(self):
        starwars = StarWars()
        starwars.request_data_sync('films/1')
        assert starwars.swars_data['title'] == "A New Hope"

    def test_request_data_async(self):
        starwars = StarWars()
        starwars.request_data_async('films')
        control_set = read_json_file('starwars-backup-data/films.json')
        assert starwars.swars_data == control_set['results']

    def test_request_data_async_batch_max(self):
        starwars = StarWars()
        starwars.request_data_async('people', 10, 10)
        assert len(starwars.swars_data) == 10

```

### Testing the Authorisation Module

There are three test files under auth:

1. test_core.py
2. test_endpoints.py
3. test_utils.py

They contain the following tests for testing our Authorization code:

#### test_core.py

```python
from unittest import TestCase
from auth.core import generate_jwt, decode_auth_token, verify_payload, permission
from config.v1.app_config import JWT_SECRET, JWT_REFRESH_SECRET, JWT_EMAIL_SECRET


class Test(TestCase):

    def test_generate_jwt_standard(self):
        token = generate_jwt(user_id=1, access_role='basic', payload_claim={'standard_claim': True})
        payload = decode_auth_token(token, JWT_SECRET)
        assert payload['user_id'] == 1 and payload['access_role'] == 'basic'
        assert verify_payload(payload, 'basic')

    def test_generate_jwt_refresh(self):
        token = generate_jwt(user_id=1, access_role='basic', payload_claim={'refresh_claim': True})
        payload = decode_auth_token(token, JWT_REFRESH_SECRET)
        assert payload['user_id'] == 1 and payload['access_role'] == 'basic'
        assert verify_payload(payload, 'basic')

    def test_generate_jwt_email(self):
        token = generate_jwt(user_id=1, access_role='basic', payload_claim={'email_claim': True})
        payload = decode_auth_token(token, JWT_EMAIL_SECRET)
        assert payload['user_id'] == 1 and payload['access_role'] == 'basic'
        assert verify_payload(payload, 'basic')

    def test_pemissions(self):
        token = generate_jwt(user_id=1, access_role='basic', payload_claim={'standard_claim': True})
        payload = decode_auth_token(token, JWT_SECRET)
        assert payload['user_id'] == 1 and payload['access_role'] == 'basic'
        assert permission(payload, 'basic')

```

#### test_enpoints.py

```python
from unittest import TestCase
from auth.core import generate_jwt
from auth.endpoints import decode_token, decode_refresh_token


class Test(TestCase):

    def test_decode_token(self):
        token = generate_jwt(user_id=1, access_role='basic', payload_claim={'standard_claim': True})
        payload = decode_token(token)
        self.assertIs(isinstance(payload, dict), True)

    def test_decode_refresh_token(self):
        token = generate_jwt(user_id=1, access_role='basic', payload_claim={'refresh_claim': True})
        payload = decode_refresh_token(token)
        self.assertIs(isinstance(payload, dict), True)

```

#### test_utils.py 

```python
from unittest import TestCase

from auth.utils import prep_password, check_password


class Test(TestCase):

    def test_password_correct(self):
        pwd = prep_password("00Apassword7")
        result = check_password("00Apassword7", pwd)
        self.assertIs(result, True)

    def test_password_incorrect(self):
        pwd = prep_password("00Apassword7")
        result = check_password("00Apassword", pwd)
        self.assertIs(result, False)
```

>Copy all the tests to the appropriate files and study each test and what its purpose is. Then run all the tests again.

## That's it! Congratulations you have successfully completed building your Star Wars backend API
### Give yourself a pat on the back and may the force be with you.


