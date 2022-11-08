# Wiederholung Tag 1

* Producer - schreibt Messages auf Partitionen.
  * Sendet Nachricht an Broker
  * Client, d.h. TCP-Verbindung zum Broker. 
  * Initial: Liste mit Bootstrapping-Servers, dann erhält der Producer Cluster-Metadaten
  * Teil an einer Anwendung. Verschiedene APIs für die vers. Progammiersprachen
  * Load-Balancing, Sharding über Strategie / Key.
  * Schreibt auf Leader
  * Erhält vom Cluster ein "ACK", wenn Nachrichten erfolgreich gesendet wurden sind.
* Topic
  * Wie "E-Mail Box", Adresse eines "Kanals" an den Nachrichten (Messages) gesendet werden
  * Konfugations-Parameter: U.a. Clean-Up-Policy (compact, delete)
  * Segment.ms / bytes (d.h. Details clean-up-policy)
  * Replication-Faktor, Anzahl der Partions
  * Topic: Logisch / Partitions: Physikalisch
* Partition
  * Teil des Topics, d.h. ein Topic besteht aus Partitions
  * Hat eine lokale Ordnung, append-only
  * Offset: Index / ID einer Nachricht, **in einer Partition**
  * Besteht aus Head und Tail. Head ist (auch) im Arbeitsspeicher, Tail ist (nur) auf dem Massenspeicher
  * Clean-Up-Policy bezieht sich nur auf den Tail
  * Jede Partition enthält einen Teil der Nachrichten. Jede Nachricht ist ein genau einer Partionen gespeichert (bis auf Replikation).
* Consumer
  *  Active Polling, empfängt Messages
     * Bleibt der Poll aus, wird der Consumer als offline gewertet,  
  *  Hat einen Consumer-Offset (d.h. pro Topic / pro Partionen)
    * Index der nächsten zu lesenden Nachricht
    * Persistenz: Eigenes Topic, extern
  * Teil einer Anwendung, d.h. API für verschiedenen Programmiersprachen.
  * Consumer-Groups: Fassen Consumer zusammen. Jeder Consumer ist Teil einer Consumer-Group (Hinweis: Eine Consumer kann auch nur einen Consumer enthalten)
    * Falls zwei Consumer in der gleichen Consumer-Group sind, lesen sie nicht aus der gleichen Partition, d.h. sie bekommen verschiedene Nachrichten.
      * falls nicht: Polling von der gleichen Partition möglich (jede Consumer bekommt *alle* Nachrichten)
    * Ändern sich die Consumer in der Group (d.h. durch ausbleibendes Polling), dann führt kafka ein "re-balancing" durch. D.h. die Partions werden neu an die
      Consumer verteilt. Nicht commited consumer-offset, führen dann z.B. zu doppelt verarbeiteten Nachrichten. 
   Aber: die Festlegung zwischen Partitons und Consumers kann manuell erfolgen.
* Broker
  * Serverprozess
  * Empfängt und speichert Nachrichten (Storage und Messaging Komponenten)
  * Repliziert
  * Gibt default-Settings für topics vor
  * Eindeutige Broker-ID.
  * Müssen für Producer, Consumer und Zookeeper über das Netzwerk erreichbar sein.
  * Verwaltet Partitions, z.B. Partitions
  * Nutzt Zookeeper für das Cluster-Management 
* Nachrichten
  * Sollten recht klein sein - Ideal: Innerhalb eines Ethernet-Frames (~ 1 KiB).
  * Optional: Kompression ist möglich
  * Key, Value-Paar, beliebige Folge von Bytes. Serialisierung frei wählbar. XML: eher schwierig, da recht groß.
  * JSON auch möglich, compression kann sinnvoll sein.
 * Aufruf-Semantiken
    1. At least once: Eine Nachricht wird mind. 1x verarbeitet
    2. At most once: Eine Nachtricht wird maximal 1x verarbeitet
    3. Exactly once: Eine Nachricht wird genau 1x verarbeitet
  * Praxis: Häufig exactly once
  * Herausforderung kafka
     * Asynchrone Kommunikation, kein definierter Timeout, kein zuverlässiger Kanal
     * Alternativ: Synchrones System, warten auf Timeouts (z.B. Rest-API).
 * Exactly once in der Praxis:
   * Sehr schlechte Idee 2-Phase commit / Verteilte Transaktion über Kafka (>> 100ms Transaktionsdauer bis hin zu einigen Sekunden)
   * Consumer merkt den offset lokal, transaktionssicher. D.h. die Verarbeitung passiert innerhalb einer Transaktion.
   * Fehlerhafte Konfiguration, Retention (d.h. Löschen)       
   * Producer: Muss auf das ACK warten, obacht:
     * Blocking: Niederiger Durchsatz: z.B. 25 msec Netzwerklatenz => "nur" 40 Nachrichten pro Sekunden.
     * Non-Blocking: Über Java-Future: Retry im Hintergrund, Verabreitung nicht garantiert, falls JVM terminiert.

  
