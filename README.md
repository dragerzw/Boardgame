# BoardgameListingWebApp

## Description

**Board Game Database Full-Stack Web Application.**
This web application displays lists of board games and their reviews. While anyone can view the board game lists and reviews, they are required to log in to add/ edit the board games and their reviews. The 'users' have the authority to add board games to the list and add reviews, and the 'managers' have the authority to edit/ delete the reviews on top of the authorities of users.  

## Technologies

- Java
- Spring Boot
- Amazon Web Services(AWS) EC2
- Thymeleaf
- Thymeleaf Fragments
- HTML5
- CSS
- JavaScript
- Spring MVC
- JDBC
- H2 Database Engine (In-memory)
- JUnit test framework
- Spring Security
- Twitter Bootstrap
- Maven


graph TD
    %% Define Styles
    classDef main fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef sec fill:#ffebee,stroke:#b71c1c,stroke-width:2px;
    classDef deploy fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px;

    %% Nodes
    Start([User Push to Main]) -->|Trigger| Checkout[Checkout Code & Install Utilities]
    Checkout --> Build[Build JAR (Maven)]
    
    subgraph Security_Checks [Security & Quality Gates]
        direction TB
        Build --> TrivyFS[Trivy FS Scan]
        Build --> Sonar[SonarQube Analysis]
        Sonar --> QGate{Quality Gate Passed?}
    end

    QGate -- No --> Fail([Pipeline Failed])
    QGate -- Yes --> DockerBuild[Docker Build]
    TrivyFS --> DockerBuild

    subgraph Containerization
        DockerBuild --> TrivyImg[Trivy Image Scan]
        TrivyImg --> DockerPush[Push to Docker Hub]
    end

    subgraph Deployment
        DockerPush --> K8sAuth[Auth via KubeConfig]
        K8sAuth --> K8sApply[kubectl apply -f deployment.yaml]
    end

    K8sApply --> End([App Live in K8s])

    %% Apply Styles
    class Start,Checkout,Build,DockerBuild,DockerPush main;
    class TrivyFS,Sonar,QGate,Fail,TrivyImg sec;
    class K8sAuth,K8sApply,End deploy;



## Features

- Full-Stack Application
- UI components created with Thymeleaf and styled with Twitter Bootstrap
- Authentication and authorization using Spring Security
  - Authentication by allowing the users to authenticate with a username and password
  - Authorization by granting different permissions based on the roles (non-members, users, and managers)
- Different roles (non-members, users, and managers) with varying levels of permissions
  - Non-members only can see the boardgame lists and reviews
  - Users can add board games and write reviews
  - Managers can edit and delete the reviews
- Deployed the application on AWS EC2
- JUnit test framework for unit testing
- Spring MVC best practices to segregate views, controllers, and database packages
- JDBC for database connectivity and interaction
- CRUD (Create, Read, Update, Delete) operations for managing data in the database
- Schema.sql file to customize the schema and input initial data
- Thymeleaf Fragments to reduce redundancy of repeating HTML elements (head, footer, navigation)

## How to Run

1. Clone the repository
2. Open the project in your IDE of choice
3. Run the application
4. To use initial user data, use the following credentials.
  - username: bugs    |     password: bunny (user role)
  - username: daffy   |     password: duck  (manager role)
5. You can also sign-up as a new user and customize your role to play with the application! 😊
