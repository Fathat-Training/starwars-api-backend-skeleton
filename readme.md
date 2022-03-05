# FatHat.org starwars-api-backend-skeleton Learning Project

The aim of this project is to teach the fundamentals of building a Python API with openApi. The API itself 
is a pipeline API that uses the Star Wars Data at https://swapi.py4e.com/api/ and pass that through this API to 
clients. The openAPI enables the use of a Swagger definition and User interface. The definition acting as the definitive 
APi specification. The minimal use of the Python framework Flask in conjunction with Connexion https://connexion.readthedocs.io/en/latest/
are used to provide the bridge to our python API endpoints.

The project framework is empty save for a couple of database configuration declarations in the config.v1.app_config.py file. The

Apart from these, there are several markdown files and their html counterparts under training-docs. These documents take the user through the build process 
of the project step by step with explanations of the code. The user is expected to copy and paste the code, using these
instructions, into the relevant files and test as described. 

The build process describes in detail all aspects of the project code-base without being over cumbersome. There is no time
specification for building the project. More important than time is the understanding of the project architecture and what is going on in the code itself. 

Before building this project the student must have at least a beginners understanding of Python, OpenAPI and APIs in general. An understanding of the http protocol is
recommended.

###Before you start
Before you start the build process the fundamentals must be put in place. The project uses two databases, 'Redis' and 'MySQL'. The project should
ideally be run in a virtual environment and requires python 3.10.

The databases need to be setup prior to the build process and kept running. We have included instructions for creating the database code and any tables
in the build code at the appropriate place. Other setup details for both Mac OS and Ubuntu are provided in the root directory of the project
along with some accompanying bash scripts for the setup of 'Redis'. Please read the files properly before running any of the commands.
If you have an existing virtual environment manager and the databases already setup on your system these can be ignored.

>[Mac OS - Setup](mac_setup.md)

>[Installing Redis on Mac OS](mac-redis.sh)

>[Ubuntu - Setup](ubuntu_setup.md)

when you have finished setting up - you can start building the project

>[Start Building - Markdown](training-docs/intro.md)

>[Start Building - HTML](training-docs/hitml-docs/intro.html)
