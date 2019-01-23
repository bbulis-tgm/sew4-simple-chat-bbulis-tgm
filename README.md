# "Java GUI & Socket Programmierung - Simple Chat"

## Aufgabenstellung
Die detaillierte [Aufgabenstellung](TASK.md) beschreibt die notwendigen Schritte zur Realisierung.

## Nutzen des SimpleChats
Genutz kann der SimpleChat mit folgenden Commands mit Hilfe von Gradle:   
````
gradle server //Server muss vor den Clients gestartet werden
````
````
gradle client
````

## Implementierung
### Funktion die Implementiert wurden:
**Client**
* Verbindung der Clients zum Server
* Chaten der Clients
* Anzeigen der eingehenden Nachrichten am Client
* Schließen eines Clients trennt die Verbindung zum Server
* Sauberes Schließen und Rückgabe aller Resourcen
* Schließt sich ein Cllient wird er vom Server gelöscht 


**Server**
* Chaten des Servers
* Anzeigen der eingehenden Nachrichten am Server
* Schließen des Servers schließt alle Clients
* Sauberes Schließen und Rückgabe aller Resourcen
* Anzeigen aller Clients am Server

### Funktionen die (noch) nicht Implementiert wurden:
* Ändern des Chatnames
* Schließen des Chatclients durch Remove-Button
* Privates Chaten

### Probleme beim Implementieren
Anfangs war es schwer den Zusammenhang zwischen den Methoden und Klassen zu finden. Alle wirke Unschlüssig und reduntant.
Im laufe des Implementierens hat sich dies aber gelegt. Die Klassen haben einen Sinn ergeben welche für welchen Teil zuständig sind.
Die Implementierung des Server hat eine große Herausforderung gestellt da es die Aufgabe war jeden Client einzeln anzusprechen,
als auch alle gesamt Anzusprechen.  
**Ansprechen aller Clients**
````java
    public void send(String message) {
        for (ClientWorker worker : this.workerList.keySet()) {
            worker.send(message);
        }
    }
````  
**Ansprechen eines Clients**  
````java
    public void send(String message, String receiver) {
        for (ClientWorker worker : this.workerList.keySet()) {
            if (this.workerList.get(worker).equals(receiver)) {
                worker.send(message);
            }
        }
    }
````

Eine weitere Schwirigkeit war das Überprüfen ob es sich bei einer eingehenden Nachricht um 
einen Command handelt und das darauffolgende Ausführen des Commands.  
**Ausführen eines Commands**
````java
    public void received(String plainMessage, ClientWorker sender) {
        String message = "";
        if (!plainMessage.startsWith("!")) {
            message = MessageProtocol.textMessage(plainMessage, workerList.get(sender));
        } else {
            switch (MessageProtocol.getCommand(plainMessage)) {
                case EXIT:
                    this.server.removeClient(workerList.get(sender));
                    this.send(MessageProtocol.getMessage(EXIT), workerList.get(sender));
            }
        }
        this.server.sendMessage(message);
    }
```` 

Ein Verhalten das mit komisch vor kam und bisher auch nicht Sinn ergibt ist das saubere Schließen
und zurückgeben der Resourcen. Damit sauber Geschlossen werden kann **muss** zuerst der OutputStream
geschlossen werden und dann die anderen Resourcen. Warum dies zuerst geschehen muss ist leider nicht 
verständlich für mich.  
**Sauberes Schließen des Clients**
````java
    public void shutdown() {
        listening = false;
        try {
            this.send(MessageProtocol.getMessage(EXIT));
            out.close();
            in.close();
            socket.close();
            SimpleChat.clientLogger.log(INFO, "Shutting down Client ... listening=" + listening);
        } catch (IOException e) {
            SimpleChat.clientLogger.log(WARNING, e.toString());
        }
    }
```` 

## Quellen
* [Java API](https://docs.oracle.com/javase/7/docs/api/)  
* [stack**overflow**](https://stackoverflow.com/)

### Danksagung
Ich danke besonders 
* David Lanheiter
* Dominik Cerny
* Leo Halbritter  

 für die Hilfe, bei der Lösung schwiriger Probleme