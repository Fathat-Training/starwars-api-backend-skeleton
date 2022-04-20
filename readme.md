# FatHat.org starwars-api-backend-skeleton Learning Project


---

#### Version 1.0
#### copyright (c) 2022 - Fathat.org
#### Released under the MIT sublicense

---

This tutorial is free to use under the above mentioned MIT license. We encourage anyone who wishes to use it for self learning or teaching to do so without hindrance. 
Please leave the title, copyright and license information intact. 

> Notes for those starting to learn about Software Development. Sometimes it feels impossible, sometimes it feels that there is too much to learn. Don't let these feelings stop
you. You will have many breakthrough moments that will encourage you and provide a sense of achievement. Continue learning, it's what life is all about.

> Notes for all those existing Python gurus. If you have any constructive comments and recommendations we would love to hear about them. 
> The completed project without the learning tutorials is available on GitHub at [StarWars API](https://github.com/Fathat-Training/starwars-api.git).
> To provide feedback on this course or contact us at Fathat.org, drop us a line at hello@fathat.org

---

The aim of this project is to teach the fundamentals of building a Python API with openApi. The API itself 
is a pipeline API that uses the Star Wars Data at https://swapi.py4e.com/api/ and pass that through this API to 
clients. The openAPI enables the use of a Swagger definition and User interface. The definition acting as the definitive 
APi specification. The minimal use of the Python framework Flask in conjunction with Connexion https://connexion.readthedocs.io/en/latest/
are used to provide the bridge to our python API endpoints.

The project framework is empty save for a couple of database configuration declarations in the config.v1.app_config.py file. The

Apart from these, there are several markdown files under training-docs. These documents take the user through the build process 
of the project step by step with explanations of the code. The user is expected to copy and paste the code, using these
instructions, into the relevant files and test as described. 

The build process describes in detail all aspects of the project code-base without being over cumbersome. There is no time
specification for building the project. More important than time is the understanding of the project architecture and what is going on in the code itself. 

Before building this project the student must have at least a beginners understanding of Python, OpenAPI and APIs in general. An understanding of the http protocol is
recommended.

### Before you start
Before you start the build process the fundamentals must be put in place. The project uses two databases, 'Redis' and 'MySQL'. The project should
ideally be run in a virtual environment and requires python 3.10.

The databases need to be setup prior to the build process and kept running. We have included instructions for creating the database code and any tables
in the build code at the appropriate place. Other setup details for both Mac OS and Ubuntu are provided in the root directory of the project
along with some accompanying bash scripts for the setup of 'Redis'. Please read the files properly before running any of the commands.
If you have an existing virtual environment manager and the databases already setup on your system these can be ignored.

> [Mac OS - Setup](mac_setup.md)

> [Installing Redis on Mac OS](mac-redis.sh)

> [Ubuntu - Setup](ubuntu_setup.md)

when you have finished setting up - you can start building the project

> [Start Building - Markdown](training-docs/intro.md)

