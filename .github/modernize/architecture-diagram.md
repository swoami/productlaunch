# Architecture Diagram

ProductLaunch is an event-driven .NET Framework 4.5.2 application with an ASP.NET Web Forms frontend, NATS messaging, SQL Server persistence, and Elasticsearch indexing.

## Application Architecture

```mermaid
flowchart TB
    subgraph WebLayer["Presentation Layer"]
        WebApp["ProductLaunch.Web\nASP.NET Web Forms\n.NET Framework 4.5.2\nBootstrap 3 / jQuery"]
    end

    subgraph CoreLayer["Shared Libraries"]
        Entities["ProductLaunch.Entities\nDomain Models\nProspect, Country, Role"]
        Core["ProductLaunch.Core\nConfiguration\nEnvironment Variables"]
        Messaging["ProductLaunch.Messaging\nNATS Client\nEvent Publishing\nJSON Serialization"]
    end

    subgraph MessageHandlers["Event Handlers - Console Applications"]
        SaveHandler["ProductLaunch.MessageHandlers\n.SaveProspect\nPersists prospect to database"]
        IndexHandler["ProductLaunch.MessageHandlers\n.IndexProspect\nIndexes prospect for search"]
    end

    subgraph DataLayer["Data Access Layer"]
        Model["ProductLaunch.Model\nEntity Framework 4.3.1\nProductLaunchContext\nDbContext - Code First"]
    end

    subgraph ExternalServices["External Services"]
        NATS["NATS Message Queue\nSubject: events.prospect.signedup\nPub-Sub Pattern"]
        SQLServer["SQL Server Database\nTables: Prospects,\nCountries, Roles"]
        Elasticsearch["Elasticsearch\nIndex: prospects\nNEST Client v5.0.1"]
    end

    WebApp -->|"Reads countries and roles"| Model
    WebApp -->|"Saves prospect"| Model
    WebApp -->|"Publishes ProspectSignedUpEvent"| Messaging
    Messaging -->|"Sends event via NATS"| NATS
    NATS -->|"Delivers event"| SaveHandler
    NATS -->|"Delivers event"| IndexHandler
    SaveHandler -->|"Persists prospect via EF"| Model
    IndexHandler -->|"Indexes document via NEST"| Elasticsearch
    Model -->|"Reads and writes data"| SQLServer
    WebApp --> Entities
    Messaging --> Entities
    SaveHandler --> Messaging
    IndexHandler --> Messaging
    SaveHandler --> Entities
    IndexHandler --> Entities
    SaveHandler --> Core
    IndexHandler --> Core
```
