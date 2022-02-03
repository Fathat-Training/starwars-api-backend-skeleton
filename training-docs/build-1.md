
# starwars-api-backend-skeleton

---

### Backend API Learning Workflow:

---
### Stage 1: 
<span style="color:#FF1B55FF">API Foundation - app, endpoint - Api specification</span>

#### Description: 
Getting Flask running and creating our first API call, defining the required OpenApi Yaml configuration for it.
Connecting that to our endpoint and finally running it.

<details>

<summary style="color:#4ba9cc">1. Get flask running</summary>
   <br/><br/>
   Copy the following code and place it in main.py in the root folder. 

   ```python


   # -*- coding: utf-8 -*-
   
   # -------------------------------------------------
   #  External Imports
   # -------------------------------------------------
   from flask import Flask
   
   # -------------------------------------------------
   #  Python Imports
   # -------------------------------------------------
   
   # -------------------------------------------------
   #  Module Imports
   # -------------------------------------------------
   
   # -------------------------------------------------
   #  Setup
   # -------------------------------------------------
   
   app = Flask(__name__)
   
   
   if __name__ == '__main__':
       app.run()

   ```
   
   This provides a basic flask application that runs but does nothing. Try running main.py now.
   <br/><br/>
   What we have is a running flask app on port 5000, as can be seen below:

   ```python
       * Serving Flask app "main" (lazy loading)
       * Environment: production
         WARNING: This is a development server. Do not use it in a production deployment.
         Use a production WSGI server instead.
       * Debug mode: off
       * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
   ```

   If you click on the http://127.0.0.1:5000/ you will go to the browser but all you'll get is a not found page.
   This is because the Flask app is simply running on a port on localhost but not pointing to anything. 
   <br/><br/>
</details>

<details>
<summary style="color:#4ba9cc">2. Add our first endpoint for the API</summary> 

   Our first endpoint is a films endpoint
   <br/><br/>
   Navigate to the films/v1/folder and copy the following code and append it to endpoints.py

   ```python
    def get_film(film_id, **kwargs):
        """
            Fetch a film's entity from its name
        :param film_id: The id of the film to be retrieved
        :return: Film Entity
        :errors:
            raises an APIError
        """
        pass
   ```
   
   This is the basic python function for our first films endpoint.

   Now copy the following openAPi yaml markup to the openapi.yaml file in the root folder.

```yaml
openapi: 3.0.0

info:
  title: "{{title}}"
  version: "1.0.0"


# Avoid having a definitive base path here. Set the path in the actual paths - facilitate versions
# Example v1.0.0/login and v1.0.2 can both be specified

servers:
  - url: http://127.0.0.1:5003/
    description: relative path example

paths:

  # -----------------------------------------------
  # Film paths - REQUESTS
  # -----------------------------------------------

  /films/v1/{film_id}:

    get:
      summary: Retrieve a specific star wars film data set
      tags:
        - Film
      description: >
        
        Errors:

          token-invalid, 401
          authorisation-required, 401
          not-found, 404

      operationId: films.v1.endpoints.get_film
      parameters:
        - name: "film_id"
          description: Films Unique id
          in: path
          required: true
          schema:
            type: string
        - name: "options"
          in: query
          description: Optional Film Data
          required: false
          style: deepObject
          schema:
            $ref: '#/components/schemas/FilmExtras'
      responses:
        '200':
          description: Returns a data object containing a Films data
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/FilmResponse'

# -----------------------------------------------
# COMPONENTS
# -----------------------------------------------
components:


# -----------------------------------------------
  # SCHEMAS
  # -----------------------------------------------
  schemas:

    # -----------------------------------------------
    #  FILM SCHEMAS
    # -----------------------------------------------

    # -----------------------------------------------
    #  Film DATA SCHEMAS
    # -----------------------------------------------

    BaseFilm:
      properties:
        title:
          description: Film's Title
          type: string
        episode_id:
          description: Films id representing it's order of creation
          type: string
        opening_crawl:
          description: Films opening text
        director:
          description: Film Director
        producer:
          description: Film Producer
          type: string
        release_date:
          description: Date the film was released in to Cinemas
          type: string
        created:
          description: Date when record of this film was created in the database
          type: string
        edited:
          description: Date when record of this film was last edited in the database
          type: string
        url:
          description: The URL of the film
          type: string

    FilmCharacters:
      properties:
        characters:
          description: List of urls for characters in the film
          type: array
          items:
            type: string

    FilmVehicles:
      properties:
        vehicles:
          description: List of urls for vehicles in the film
          type: array
          items:
            type: string

    FilmStarships:
      properties:
        starships:
          description: List of urls for starships used in the film
          type: array
          items:
            type: string

    FilmPlanets:
      properties:
        planets:
          description: List of urls for planets in the film
          type: array
          items:
            type: string

    FilmSpecies:
      properties:
        species:
          description: List of urls for the different species of characters in the film
          type: array
          items:
            type: string

    # -----------------------------------------------
    #  Film Extras REQUEST SCHEMA
    # -----------------------------------------------
    FilmExtras:
      type: object
      properties:
        characters:
          description: provide film character urls
          type: boolean
        planets:
          description: provide all film planet urls
          type: boolean
        species:
          description: provide all film species urls
          type: boolean
        starships:
          description: provide all film starship urls
          type: boolean
        vehicles:
          description: provide all film vehicle urls
          type: boolean

    # -----------------------------------------------
    #  Film RESPONSE SCHEMAS
    # -----------------------------------------------

    FilmResponse:
      allOf:
        - $ref: '#/components/schemas/BaseFilm'
      anyOf:
        - $ref: '#/components/schemas/FilmCharacters'
        - $ref: '#/components/schemas/FilmPlanets'
        - $ref: '#/components/schemas/FilmSpecies'
        - $ref: '#/components/schemas/FilmStarships'
        - $ref: '#/components/schemas/FilmVehicles'

```
    Now we have our first Request, Response and Schema definitions for our first API call to get a films data via the films endpoint, but no way of connecting the two together. 
    However, before we move on to fixing that let's take a good long look at what we've just placed in our openapi.yaml file.


</details>

<details>
<summary style="color:#4ba9cc">3. Add the connection between our openapi.yaml specification and our first film endpoint.</summary>
   
   Now we understand the openapi.yaml specification for our API call's Request and Response let's add the connection (connexion) between our openapi.yaml specification and our first film endpoint.
   Copy the following code and replace everything in main.py with this new code. 

   ```python
      
   # -------------------------------------------------
   #  External Imports
   # -------------------------------------------------
   import connexion
   
   # -------------------------------------------------
   #  Python Imports
   # -------------------------------------------------
   
   
   # -------------------------------------------------
   #  Module Imports
   # -------------------------------------------------
   
   
   # -------------------------------------------------
   #  Setup
   # -------------------------------------------------
   # Setup the connexion app - for swagger self documenting API routes
   app = connexion.FlaskApp(__name__)
   app.add_api('openapi.yaml',
               strict_validation=True,
               arguments={'title': 'Fathat.io Star Wars Project'})
   
   
   # -------------------------------------------------
   #  Kick off
   # -------------------------------------------------
   def startup():
       """
           Method to fire any startup config stuff up
       :return:
       """
       pass
   
   
   if __name__ == '__main__':
       startup()
       app.run(host="127.0.0.1", port=5003)

   ```
   

   Checkout what we have added in this latest main.py code.

   * We have imported a python package called connexion
   * We have connected the connexion package to flask app - with strict validation and a title 
   * We have added a startup function to the app in case we want to run any code prior to running the app. Perhaps some config loading?
  
   * We have added a host and a port to the app.run function. This tells the app to run on 
     this out localhost at port 5003.

   So, to recap:
   <br/><br/>
   * We have an app that will run on our locahost at port 5003.
   * We have an openAPi yaml specification for films and we have a single endpoint for films.
   <br/><br/>
   But, if we take a closer look at our openapi.yaml file we notice that the film endpoints require a response
   
   ```yaml
      responses:
        '200':
          description: Returns a data object containing a Films data
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/FilmResponse'
   ```
   
   The response is the schema FilmResponse. But how are we sending the response back from the film endpoint
   to the client? Checking that endpoint, you will see that it has a 'pass'.
   
   To recap a pass in python does nothing but allows the function to be semantically correct without any functional code.
   <br/><br/>
   So we have an endpoint that will receive arguments based on our OpenApi specification but 
   actually does nothing.
   
   Let's run pour API application from main.py.

   ```python
      * Running on http://127.0.0.1:5003/ (Press CTRL+C to quit)
      * Serving Flask app "main" (lazy loading)
      * Environment: production
        WARNING: This is a development server. Do not use it in a production deployment.
        Use a production WSGI server instead.
      * Debug mode: off
   ```

   Copy the following http://127.0.0.1:5003/ui/ and put it in a new tab/window of your browser that
   <br/><br/>
   - Note this has the /ui/ appended to the host and port
   <br/><br/>
   You will see the following:
   <br/><br/>
   
   ![](.build-1_images/92dc16da.png)

   Take a moment to check the details of the API call
   * Check what parameters it requires for the Request, what optional parameters might be passed
   * Check the Response it requires
   * Take a look at the schemas for Films
   

   Once you are comfortable with the openAPi specification, 
   Click on the GET film API and you will see the following:

   ![](.build-1_images/873778c7.png)

   * Click on Try it out
   * Enter a 1 into the field where it says 'film_id'
   * Click the blue execute button below
   <br/><br/>
   You will see the following response from the server
   <br/><br/>
   ![](.build-1_images/102b7afe.png)
   <br/><br/>
   
   The server returned a 204 - No Content response. The call did not fail in as much as it was successfully routed, however,
   the endpoint returned nothing.
   <br/><br/>
   Let's take a step further in fixing that!
   <br/><br/>

</details>

<details>
<summary style="color:#4ba9cc">4.Add our API Response</summary>
  
   Copy the following code into the basehandler.py

   ```python
      
   # *-* coding: UTF-8 *-*
   
   # ------------------------------------------------
   #     Python Library Imports
   # ------------------------------------------------
   
   # ------------------------------------------------
   #    External Python Library Imports
   # ------------------------------------------------
   
   # ------------------------------------------------
   #     Module Imports
   # ------------------------------------------------
   
   # ------------------------------------------------
   #    Base Handling Functions Begin Here
   # ------------------------------------------------
   
   def api_response(payload=None):
       """
           Generate and return an appropriate response to the API request
   
       :param payload:
       :return:
       """
       if isinstance(payload, dict):
           # If something in dict return with data else just status
           if len(payload) > 0:
               return payload
           else:
               return {}
       else:
           return {}

   ```
   
  Here we are adding our function that handles the api responses from every endpoint that we code.
  <br/><br/>
  Let's briefly discuss this function and understand what it does.

  Now let's get a call to this response function into our Films endpoint.
  
  Go to the films endpoint file - films/v1/endpoints.py, and add the following under the section
  Module Imports.

  ```python
  from basehandler import api_response
  ```
  
  This will import the function api_response from the basehandlers.py file that we added earlier
  
  Add the following to the get_film endpoint, removing 'pass' first.
   
  ```python
  return api_response()
  ```
 
  Run the application again.
  <br/><br/>
 This time we get a 200 response and two curly braces signifying an empty object. 

  ### Summary of what we have achieved so far!

  * We have got a running flask app
  * We have coded our first films endpoint.
  * We have defined our Films specification for OpenAPI
  * We have linked our Flask App, endpoint and openAPI specification via Connexion.

   Good Work!

  But hold on a minute, all that work and we still have no data. Obviously we are missing something. Yes, you guessed it some route into the Star Wars Data.
  Which, takes us neatly into stage-2, Defining our Data Access layers and defining our interface to the Star Wars API.

</details>
