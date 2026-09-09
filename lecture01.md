# Devops

## What is DevOps

DevOps is a new term emerging from the collision of two major trends: *"agile infrastructure"*
Devolved from **Agile** & **Lean** approaches for operations to work
*"Study of building, evolving and operating rapdily-changing resilent systems at a scale"*

- Is a combination of *software development* and *information technology* operations that enables businesses to deliver applications at a **faster** pace.
  - Brings together development and operations teams so there are fewer redundancies in the software development process
  - i.e. *a culture and automation practice focused on continuous integration, delivery (CI/CD), and rapid feedback loops between development and operations*

### Buzzwords

- **Waterfall**
  - a rigid, linear software development life cycle (SDLC) where each phase must finish completely before the next one starts
- **Agile**
  - focuses on fast, iterative software development
  - DevOps extends principles to automate and streamline software delivery and IT operations
- **Sprint**
  - Short fixed period (2-3 weeks) where a team commits to finishing a specific task, updates, or infrastructure goal
- **SDLC**
  - software development life cycle
- **Scrum**
- **Scrum Master**
- **BASH**
  - Bourne-again shell-unix shell and command language. Most operating system terminals run this language by default. BASH is used to have command line access to your system files, use commands to execute operations, write script files which contain BASH syntax for automating or simplifying tasks
- **Terminal or shell**
  - An interface that provides command line access to your machines system. Terminals are used to configure an operating system, perform CRUD operations on system files, execute software and automate tasks.
- **Node**
  - "Simply put: server-side JavaScript". *Have you ever ran JavaScript commands in a Chrome developer console? Node can be thought of as the engine behind that console (V8), ripped out and configured to serve as an applications back-end. Node also offers a command line tool called NPM (Node package manager). NPM allows us to install third-party dependencies and interface withNode on the command line
- *build* or **application build**
  - Typically some type of compilation process where all relevant project dependencies, source code, media and building some sort of binary or executable containing an application for use by an end-user or machine
- **build process**
- **CI/DC pipeline**

#### Why Devops adoption?

- Devops culture brings the cohesion of autonomous teams to fast paced, high-demanding environments.
- Devops has the ability to:
  - Drastically reduce IT overhead
  - Remove manual tasks both of a mundane or complex nature
  - Enforce best security practices
  - Expedite product delivery
  - Improve communication and collaboration across teams

## What is a build pipeline?

- A build pipeline
  - is the entity through which you define your automated **build pipeline**
  - in the **build pipeline** you compose a set of tasks, each of which perform a step in your build

### CI/CD Pipeline?

- A CI/CD pipeline implementation, or **Continuous Integration/Continuous Delivery or Continuous Deployment**, is the backbone of the modern devops environment
  - *Bridges the gap between development and operations team by automating the building, testing, and deployment of applications*

  -> Version Control > Build > Unit Test > Deployment > Auto test > Deploy to production > Measure + Validate
  **every step functions through a production feedback operations**

#### CI/CD Tools

- Tools are broken down into the following categories:
  - source control
  - build tools
  - containerisation
  - configuration management
  - monitoring/feedback

##### CI breakdown

- **Continuous Integration (CI)** is a practice in which developers will check their code into a version-controlled repository several times per day. Automated build pipelines are triggered by these check ins which allow for fast and easy to locate error detection

- Key benefits:
  - smaller changes are easier to integrate into larger code bases
  - easier for other team members to see what you have been working on
  - bugs in larger pieces of work are identified early making them easier to fix resulting in less debugging work
  - consistent code compile/build testing
  - fewer integration issues allowing rapid code delivery

###### CD breakdown

- **Continuous Delivery (CD)** is the process which allows developers and operations engineers to deliver bug fixes, features and configuration changes into production reliably, quickly, and sustainably.