## TechStack 

This is an outline of the tech stack chosen for the Cheetah Sign project and the reason behind those choices.
These technologies were selected by the client to meet their specific requirements. 

## Frontend 
- [Node.js 20](https://nodejs.org/en/blog/announcements/v20-release-announce) - 
Node.js 20 is the runtime we need to run and build the frontend. It also gives us access to npm, which is used to install extra tools and packages. 
Our client wanted us to use Node.js because it supports Vue and Vite, which are required parts of their setup.

- [Vue 3](https://vuejs.org) - 
Vue.js is the main framework for the frontend. It lets us build a single-page application  where the user can move between screens smoothly without the page reloading.
Vue is easier to learn than some other frameworks and is already used in the client’s other products.
 
- [Vite 5](https://vite.dev) - Vite is the build tool and development server for Vue.
It makes starting the app very fast which means we can see code changes right away without restarting.
This makes development smoother and was included by the client in the base setup for Cheetah Sign.

- [Visual Studio Code](https://code.visualstudio.com) - VS Code is the IDE chosen by the client for this project.
  It is lightweight, has extensions for Vue, Node.js, and Docker, and is easy for the team to use together.
  The client specifically recommended VS Code with the Docker Compose extension for working on Cheetah Sign.  



## Backend
- [.NET 8 SDK](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
.NET 8 is the framework we use for the backend services. It is reliable, cross-platform, and supports high-performance web APIs.
 We will write the backend in C#, which is well supported by .NET.
The client wanted us to use .NET because it is already used in other Cheetah projects and integrates well with Docker and PostgreSQL.


 
## DataBase 
- [PostgreSQL](https://www.postgresql.org) -  PostgreSQL is the relational database chosen by our client.
- [PgAdmin](https://www.pgadmin.org) - pgAdmin is the tool we use to manage and check the database.


## Development Tools
- [Docker](https://www.docker.com) - Docker is used to containerize the frontend, backend, and database so the system runs the same on every team member’s computer. 
Running docker starts all four parts of the project (API, frontend, Postgres database, and pgAdmin).
