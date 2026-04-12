# Architecture Diagram

ProductLaunch is an event-driven .NET Framework 4.5.2 application with an ASP.NET WebForms frontend, NATS message broker for asynchronous processing, SQL Server for transactional storage, and Elasticsearch for search indexing.

## Application Architecture

```mermaid
flowchart TD
    subgraph Web["Presentation Layer - ASP.NET WebForms / IIS"]
        UI["SignUp.aspx / Default.aspx / About.aspx\nBootstrap 3.0 / jQuery 1.10"]
        CodeBehind["SignUp.aspx.cs\nForm Handling and Postback"]
    end

    subgraph Core["Shared Libraries"]
        Entities["ProductLaunch.Entities\nProspect / Country / Role"]
        Model["ProductLaunch.Model\nProductLaunchContext - Entity Framework 4.3.1\nStaticDataInitializer - Seed Data"]
        Messaging["ProductLaunch.Messaging\nMessageQueue / MessageHelper\nProspectSignedUpEvent - JSON Serialization"]
        CoreLib["ProductLaunch.Core\nEnvironment Configuration"]
    end

    subgraph Handlers["Message Handlers - Console Applications"]
        SaveHandler["SaveProspect Handler\nDeserialize Event\nReload References\nPersist to Database"]
        IndexHandler["IndexProspect Handler\nDeserialize Event\nMap to Search Document\nIndex in Search Engine"]
    end

    subgraph Data["Data Storage"]
        SQL["SQL Server\nProspects / Countries / Roles\nSource of Truth"]
        ES["Elasticsearch 5.x\nIndex: prospects\nDenormalized Documents\nFull-Text Search"]
    end

    NATS["NATS Message Broker\nSubject: events.prospect.signedup\nPub-Sub Messaging"]

    UI -->|HTTP POST Form Submission| CodeBehind
    CodeBehind -->|Reads Static Data| Model
    CodeBehind -->|Publishes ProspectSignedUpEvent| Messaging
    Messaging -->|JSON over TCP| NATS
    Model -->|Entity Framework ORM| SQL
    NATS -->|Subscribe| SaveHandler
    NATS -->|Subscribe| IndexHandler
    SaveHandler -->|Entity Framework Save| SQL
    IndexHandler -->|NEST Client Index| ES

    CodeBehind -.->|References| Entities
    SaveHandler -.->|References| Model
    SaveHandler -.->|References| Messaging
    IndexHandler -.->|References| Messaging
```
