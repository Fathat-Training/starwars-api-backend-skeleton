# starwars-api-backend-skeleton

---

### Backend API Learning Workflow:

---
### Stage-5:
<span style="color:#FF1B55FF">Users</span>

#### Introduction: 

    In this section we shall add some user functionality to our API. Adding users will allow us to
    facilitate our Authentication to the API. 

    In the last section we ended with placing authentication requirements on our second films endpoint. 
    As a cconsequence we could not use that endpoint without a JWT. In order to issue a JWT we require a 
    a user of some form. Without users or the notion of a user there is no real way tofaciliate authentication. 
    Users and authentication go hand in hand.

Summary of objectives:

  * Add the required openAPI specifications for users.
  * Develop endpoints and data access for user signup,  login, logout and email verification
  * Develop the helper functions for user credentials and validation.
  * Develop MySQL database connectivity to store user data.
  * Add authorisation to user endpoints
  

<details>
<summary style="color:#4ba9cc">Introduction to Basic User Authentication</summary>

    Before we look at the specifications for users let's first see what we require for a basic user strategy.

      * A Signup flow
        Obviously, in order to have users that are allowed access to private data, we need to sign them up.
        To signup you need some way of identifying a user, username or email or other ID. You also need a gateway
        security mechanism, i.e. for us its a password. In some odern systems it might be face id or finger print detection,
        palm prints, retina scanning etc etc.

        The signup process is simply that signing up for a service. A signup flow should alway involve some form of verification
        A lot of services utilise email verification as one form of signup verification. This involves sending an
        email with a verification link to the provided email address. 

      * A Login flow 
        Once signup is complete, including verification, users are then able to login to access a system or service.
        Our example of the login process for our authenticated API access was already described in the 
        'Understanding the Authentication Flow'. Here it is again...

![](images/login-api.drawio.png)

        This is the flow that we shall create for our users login process. Once the user receives their access and refresh tokens 
        The user or client on behalf of the user is responsible for storing those tokens safely. 

        Thereafter, each call to a protected endpoint shall require a token.

      * A logout flow
        Logging out of our system requires a valid token to access the logout endpoint. The logout event itself must ensure that any
        user tokens issued to the user at the last login event must be revoked , meaning they no longer usable for access to any secured
        endpoint on our API.

        Once a logout event occurs, the user has to login again and receive new tokens for access.

    The above represent the core of user account fundamentals. However, there's some extra functionality we need to consider.

    Apart from the email verification flow, we need to consider what hapens when tokens expire or get lost or become exposed.
    What we do not want is to issue short lived tokens and force our users to have to frequently login each time their token expires.

    There are a couple of possible solutions here. First, we could issue long lived tokens, say several days or more, and secondly we can
    implement a framework such that when a client/user understands that the user's token has expired they can ask our API to issue new tokens
    without the user being logged out. The second of these options is the one we'll use in our application. However, it is important to remember
    that it is upto the client to ask for those new tokens and not just to send them after a token has expired.

</details>
       
<details>
<summary style="color:#4ba9cc">User openAPI Specifications</summary>

    Now we have our understanding of the basic user authentication flows, let's start to build our openAPI specification for users.

    First up let's get our three basic user endpoint requests dealt with, signup, login and logout, shown below.

#### Signup

```yaml
/users/v1/signup:
    post:
      summary: Signup up a new user
      tags:
        - Users
      description: >
        
        Errors:
        
            password-invalid, 400
            email-invalid, 400
            user-already-exists, 400
            unsupported media type, 415

      operationId: users.v1.endpoints.signup
      requestBody:
        description: Signup Data
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UserSignup'

      responses:
        '200':
          description: Returns a success Response.
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/SuccessResponse'
```

    As we can see from first glance, the endpoint specification looks very similar to our other endpoint specifications.
    However, the key differences here are:

      * The signup endpoint is a 'POST' endpoint because it is going to save some user information to our database.
      * The client/user sends user data in the request body as is standard practise with 'POST' requests and not in the query.
      * Our request body uses a schema to identify the structure of the user data that is sent. The user data in the request 
        body will be a set of credentials, email, password and an access role, which shall be defined in a schema. We'll get
        to that after we deal with the requests.

    Other than the above it's pretty much the same. The endpoint function is pointed to by the 'operationId' and as usual, 'connexion'
    is the interface between this specification and our API endpoints.

    Our response is a simple success response, again we'll get to that after requests.

#### login
```yaml
  /users/v1/login:
    post:
      summary: Login with user's credentials
      tags:
        - Users
      description: >
        
        Errors:
        
            not-found, 404
            password-invalid, 400
            email-invalid, 400
            account-disabled, 400

      operationId: users.v1.endpoints.login
      requestBody:
        description: Login Data
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Credentials'

      responses:
        '200':
          description: Returns a JWT
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserLoginResponse'

```

    The login request is very similar but without the access role in the request body.
    We define a response schema called 'UserLoginResponse' for detailing the structure of the response
    which will include the user tokens created during the login flow.

#### logout

```yaml
  /users/v1/logout:
    get:
      summary: Logout
      tags:
        - Users
      description: >

        Required Headers:

          Authorization request header

            Bearer Valid Token

        Errors:

            'token-invalid', 401
            'authorisation-required', 401
            'User NOT logged out - problem accessing token in request', 400

      operationId: users.v1.endpoints.logout
      responses:
        '200':
          description: Returns 'ok' or an Error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/SuccessResponse'

      security:
        - jwt: []
```
    The logout request has no incoming data specified, although it is a secured endpoint request
    as ccame be seen from teh security tag at the bottom. Therefore there is incoming data in the 
    form of a token in the request header as per the Security specification schema that we put
    in place whilst building our authentication.

    A quick reminder

```yaml
securitySchemes:
    jwt:
      type: http
      scheme: bearer
      bearerFormat: JWT
      x-bearerInfoFunc: auth.endpoints.decode_token
```

#### Email verification

```yaml
/users/v1/email_verification:
    get:
      summary: Verifies user's email
      tags:
        - Users
      description: >

          Verifies user's email using an email-token generated when signing-up the user

          Errors:

              'token-invalid', 401
              'authorisation-required', 401

              'user-not-found', 404

      operationId: users.v1.endpoints.email_verification
      parameters:
        - name: token
          description: Email verification token generated at sign-up time
          in: query
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Returns OK
```

    This request is interesting as it does not have security but it does carry a token in the query.
    The request itself is made via na email link and not a normal client on a website. 
    The token is one that was will be issued specifically via our API user signup endpoint. It is used
    to confirm that the user we sent the email to is the actual endpoint requester. More of that when we
    code our endpoints.

    It responds with a straightforward http 200. Remember it's being requested from a link in an email.
    so we don't need to pass any data back.

#### Generate tokens

```yaml
/users/v1/generate_tokens:
    get:
      summary: Generates new user access and refresh tokens
      tags:
        - Users
      description: >

        Required Headers:

          Authorization request header

            Bearer Valid Refresh Token

        Errors:

          'token-invalid', 401
          'authorisation-required', 401
          'user-not-found', 404

      operationId: users.v1.endpoints.generate_new_tokens

      responses:
        '200':
          description: Returns a new access token (token) and a new refresh token (refresh_token)
          content:
            application/json:
              schema:
                 $ref: '#/components/schemas/UserTokens'

      security:
        - jwt: []

```

    Our final endpoint for users is the generated tokens endpoint above. This endpoint is called 
    with a user's legitimate refresh_token to generate a new access token and a new refresh token.

    It can be called as described earlier when a user's access token has expired and the client application
    does not want the user to have to login again. It's a convienient way of allowing login continuation.

    The refresh token that is used for authentication will be revoked after the new tokens have been generated.
    It is up to the client to ensure that the old user tokens are discarded on their side as they are no longer valid.

    The response is a vanilla 'UserTokens' schema.

    Copy the above request specifications, as usual, to the openapi.yaml file in our root directory. remember to stick theme
    in the right place, i.e. where the requests go and before 'components'

### User Schemas

    Now that we have our endpoints let's look at the required schemas for both requests and responses.

    We'll start with the request schemas.

```yaml
UserSignup:
  allOf:
    - $ref: '#/components/schemas/AccessRole'
    - $ref: '#/components/schemas/Credentials'

AccessRole:
  properties:
    access_role:
      description: Access role of user
      type: string
      enum:
        - admin
        - basic
      default: basic

Credentials:
  type: object
  required:
    - password
    - email
  properties:
    password:
      $ref: '#/components/schemas/Password'
    email:
      $ref: '#/components/schemas/Email'
```

    We have a simple 'UserSignup' Schema that uses two other schema references, which you can see 
    below it.

    The 'AccessRole' schema dictates a basic choic via an enumrated set of two options, basic and admin.
    These being the only choices that the enpoints will allow for access roles.

    The 'Credentials' schema uses two other schemas for password and email. We will look at those next.

### Password and Email Schemas 

```yaml
Email:
  description: Email Address
  type: string
  pattern: ([-!#-'*+/-9=?A-Z^-~]+(\.[-!#-'*+/-9=?A-Z^-~]+)*|"([]!#-[^-~ \t]|(\\[\t -~]))+")@[0-9A-Za-z]([0-9A-Za-z-]{0,61}[0-9A-Za-z])?(\.[0-9A-Za-z]([0-9A-Za-z-]{0,61}[0-9A-Za-z])?)+

Password:
  description: Password
  type: string
  pattern: (?=\S{8,32})(?=\S*[A-Z])(?=\S*[a-z])(?=\S*[0-9])(?=\S*)(?<!\S)\S{8,32}(?=\s|\Z)
```
    These two schemas are interesting because they dictate a pattern for how the parameter should
    be written (syntax validation). The patterns are what are known as regex patterns. Regex (Regular Expressions) are used
    as a sort of shorthand notation to verify parameter structure. It takes a while to get used to writing regex but it is
    worth looking into further as it is widely used by developers, especially for pattern verification as in this example.

    You're luck is in, as you can see, you don't have to write the patterns, they are there.

    In short these two schemas verify that any email addresses and passwords sent in requests are syntatically
    valid.

#### User token response
```yaml
# -----------------------------------------------
#  AUTH TOKEN RESPONSE SCHEMAS
# -----------------------------------------------

UserTokens:
  type: object
  properties:
    token:
      type: string
      format: byte
      description: User's api calls token
    refresh_token:
      type: string
      format: byte
      description: User's refresh token
```
    The above schemas represent the tokens. Each token is of type string but of format byte.
    The format byte declaration ensures the string token is base64 encoded. It does this to make
    the token fit nicely into the transfer protocol of http.

    for more on Base64 encoding you can visit

[Base64 Encoding](https://www.base64encoder.io/learn/)

    That's it for our users openAPI specifications.

    Copy all the above schemas to the 'Schemas' section of the openapi.yaml file.

</details>

<details>
<summary style="color:#4ba9cc">User Endpoints and Data Access</summary>

</details>

<details>
<summary style="color:#4ba9cc">MySql database connectivity</summary>

</details>

<details>
<summary style="color:#4ba9cc">User Endpoint Authorisation</summary>

</details>

<details>
<summary style="color:#4ba9cc">Testing</summary>

</details>
