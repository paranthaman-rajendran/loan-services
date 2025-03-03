```mermaid
sequenceDiagram
    participant JS as sqlite.js
    participant DB as SQLite Database
    participant FS as File System

    Note right of JS: Initialization and Database Setup
    JS->>FS: Check if .data/my.db exists
    activate FS
    alt File exists
        FS-->>JS: File exists
        deactivate FS
    else File does not exist
        FS-->>JS: File does not exist
        deactivate FS
        JS->>FS: Create directory .data (recursive)
        activate FS
        FS-->>JS: Directory created
        deactivate FS
    end

    JS->>DB: Open database connection
    activate DB
    DB-->>JS: Connection opened
    Note over JS,DB: Check for tables (Choices, Log, CustomerData)
    alt Tables do not exist
        JS->>DB: CREATE TABLE Choices
        DB-->>JS: Choices table created
        JS->>DB: INSERT INTO Choices (default data)
        DB-->>JS: Default choices inserted
        JS->>DB: CREATE TABLE Log
        DB-->>JS: Log table created
        JS->>DB: CREATE TABLE CustomerData
        DB-->>JS: CustomerData table created
        DB-->>JS: Tables created
    else Tables already exist
        JS->>DB: SELECT name FROM sqlite_master WHERE name='Choices'
        DB-->>JS: Choices table exists
        JS->>DB: SELECT name FROM sqlite_master WHERE name='Log'
        DB-->>JS: Log table exists
        JS->>DB: SELECT name FROM sqlite_master WHERE name='CustomerData'
        DB-->>JS: CustomerData table exists
        DB-->>JS: Tables exists
    end
    deactivate DB

    Note right of JS: Functions
    loop getOptions()
        JS->>DB: SELECT * FROM Choices
        activate DB
        DB-->>JS: Return Choices data
        deactivate DB
    end

    loop insertCustomerData()
        JS->>DB: INSERT INTO CustomerData
        activate DB
        DB-->>JS: Data inserted
        deactivate DB
    end

    loop loadCustomerData()
        JS->>FS: open customers.csv
        activate FS
        FS-->>JS: customers.csv file opened
        deactivate FS
        JS->>JS: Iterate over rows in customers.csv
        loop Each row in CSV
            JS->>DB: INSERT INTO CustomerData
            activate DB
            DB-->>JS: Data inserted
            deactivate DB
        end
    end

    loop getOptions()
        JS->>DB: SELECT * FROM Choices
        activate DB
        DB-->>JS: Return Choices data
        deactivate DB
    end

    loop processVote(vote)
        JS->>DB: SELECT * FROM Choices WHERE language = vote
        activate DB
        alt vote valid
        DB-->>JS: Vote valid
        JS->>DB: INSERT INTO Log
        DB-->>JS: Log inserted
        JS->>DB: UPDATE Choices SET picks = picks + 1
        DB-->>JS: Choices updated
        end
        DB-->>JS: Return Choices data
        deactivate DB
    end

    loop getLogs()
        JS->>DB: SELECT * FROM Log ORDER BY time DESC LIMIT 20
        activate DB
        DB-->>JS: Return Log data
        deactivate DB
    end

    loop clearHistory()
        JS->>DB: DELETE from Log
        activate DB
        DB-->>JS: Log data deleted
        JS->>DB: UPDATE Choices SET picks = 0
        DB-->>JS: Choices updated
        deactivate DB
    end
