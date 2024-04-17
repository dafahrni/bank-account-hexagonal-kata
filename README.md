# Bank Account Hexagonal Kata
Answer to crafters meetup of April 12 in Zurich

## Crafters Meetup Zurich 
Hexagonal-Architecture, Event Sourcing and TDD

### Overview 

Workshop:
- 18:00 - 21:00
- a mixture between lectures and exercises

Exceprcises:
- 10' Ex 1 -> discuss about technology, code structure etc.
- 30' Ex 2 -> identify and name the components of your system
- 10' Ex 3 -> discuss with another pair (how does context affect naming, testing strategies etc.)
- 45' Ex 4 -> implementation

### Example of the evening

![](./docs/Hexagonal%20Bank%20Account%20Kata%20-%20Events.jpg)

- Event Storming -> https://miro.com/app/board/uXjVKUmMhn8=/
- Todo: think about similar ABAP example in PLM context (to repeat this workshop with my team)

### Structure and Technology

- .NET / C# (Rider, VSC)
- MongoDB -> Atlas Free Tier
- REST Client -> `System.Net.Http`

Rules:
1. application must not depend on the adapters

Enforcement Tools:
- Code Review
- Arch Unit, e.g. https://www.archunit.org/
- Structure 101 (for visualization) -> https://structure101.com/
- Module Manager -> NuGet
- Linter
- micro vs. macro -> C4 container diagram as a first step

Packages:
1. Application (IF of Ports)
2. Driving Adapters (Impl. of IF)
3. Driven Adapters

Question
- Where does bootstraping belong to?

Answer (plenum)
- Application (for the use cases, i.e. Application Services)
	- Settings
	- Use Cases
	- Domain Models (Account, Customer, Transfer, Balance)
	- Domain Services (requires no IF - what !? I don't buy it)
	- Contracts (Ports)
- Adapters
	- DTOs
	- Infrastructure Services (such as Repositories etc.)
	- Driver/Primary/Inbound (e.g. REST controller)
	- Driven/Secondary/Outbound (e.g. Mongo adapter)
	- Configurations (Prod, Int, Test, Dev)
		- e.g. MongoDB Stuff (Connection etc.)

### Modelling

Container Diagram:

![](./docs/Hexagonal%20Bank%20Account%20Kata%20-%20Containers.jpg)

Component Diagram:

![](./docs/Hexagonal%20Bank%20Account%20Kata%20-%20Components.png)

Class Diagram:

![](./docs/Hexagonal%20Bank%20Account%20Kata%20-%20Code.png)

Some thoughts about testing:
- more complex, separated domain logic might require unit tests
- the simple creation of an account might be sufficient to be tested on the application level only

### Sources

Implementation
- [GitHub Repo](https://github.com/dafahrni/bank-account-hexagonal-kata)
- [Q+A via AI](https://chat.openai.com/share/a94e8922-0460-430b-b40d-02dfd1247db2)
- [MongoDB Driver](https://www.mongodb.com/docs/drivers/csharp/current/)

Related katas
- https://github.com/pdeffendol/BankAccountKata
- https://github.com/abachar/hexagonal-bank

Further links
- [Domain-Driven Design Crew](https://github.com/ddd-crew)
- [Explizit Architecture](https://medium.com/the-software-architecture-chronicles/ddd-hexagonal-onion-clean-cqrs-how-i-put-it-all-together-f2590c0aa7f6)
- [C4 PlantUML](https://github.com/plantuml-stdlib/C4-PlantUML)
- [PlantUML Server](https://www.plantuml.com/)
- [Atlas Free Tier](https://www.mongodb.com/blog/post/free-your-genius-on-mongodb-atlas-free-tier)

---

# Appendix

### Container Diagram

``` PlantUML
@startuml

!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Container.puml

!define DEVICONS https://raw.githubusercontent.com/tupadr3/plantuml-icon-font-sprites/master/devicons
!define FONTAWESOME https://raw.githubusercontent.com/tupadr3/plantuml-icon-font-sprites/master/font-awesome-5
!include DEVICONS/dotnet.puml
!include DEVICONS/mongodb.puml
!include FONTAWESOME/users.puml

'LAYOUT_WITH_LEGEND()

title Hexagonal Bank Account Kata - Containers

Person(user, "Tester", "Team that tests the API", $sprite="users")
Container(spa, "Postmen Web Client", "API testing", "The main test interface to interact with", $sprite="postmen")
Container(api, "API Application", ".NET/C#", "Handles all business logic (uses ports and adapters)", $sprite="csharp")
ContainerDb(db, "Database", "Atlas MongoDB", "Holds account and money transfer infos", $sprite="mongodb")

Rel(user, spa, "Selects", "HTTP requests")
Rel(spa, api, "Invokes", "HTTP requests")
Rel_R(api, db, "Reads/Writes")

@enduml
```

### Component Diagram

``` PlantUml
@startuml

!include https://raw.githubusercontent.com/plantuml-stdlib/C4-PlantUML/master/C4_Component.puml
' uncomment the following line and comment the first to use locally
' !include C4_Component.puml

' LAYOUT_WITH_LEGEND()

title Hexagonal Bank Account Kata - Components

Container(pwc, "Postmen Web Client", "API testing", "The main test interface to interact with", $sprite="postmen")
ContainerDb(db, "Database", "Atlas MongoDB", "Holds account and money transfer infos", $sprite="mongodb")

Container_Boundary(api, "API Application") {
    Component(httpAdptr, "REST Controller", "MVC Controller", "Adapter for HTTP requests")
    Component(app, "Application", "MVC Rest Controller", "Application for account creation and money transfers")
    Component(mongoAdptr, "Repository Implementations", "MongoDB Driver", "Adapter for persistence of money transfers and bank accounts")
    
    Rel(httpAdptr, app, "Uses", "Primary Ports")
    Rel(app, mongoAdptr,  "Uses", "Secondary Ports")
}

Rel(pwc, httpAdptr, "Invokes", "HTTP requests")
Rel_R(mongoAdptr, db, "Reads/Writes", "Mongo DB")

@enduml
```
### Class Diagram

``` PlantUML
@startuml

title Hexagonal Bank Account Kata - Code

class Account{
  +deposit(amount : Money)
  +withdraw(amount : Money)
  +getBalance(): Money
  '+verifyBalance()
}

class Accounts{
  +create(): Account
}

class Transfers {
  +create(from: Account, to: Account, amount: Money): Transfer
}

class Transfer{
  +from: Account
  +to: Account
  +amount: Money
}

Transfers "*" --> Transfer
Transfers .> Accounts
Accounts "*" --> Account
Transfer "2" -> Account

@enduml
```