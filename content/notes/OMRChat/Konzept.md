---
title: "OMRChat"
---
![test](/notes/images/hey-raphaelo.png)
Mit mit dieser einfachen Frage, begannen 5 intensive Tage. 5 Tage an denen ich viel über Prompt Engineering und AI Chatbots lernte.

Als mir Philipp Glöckler - einer der Hosts des Doppelgänger Tech Talk Podcast - schrieb, hatte nichts als einen simplen und hässlichen Prototypen eines Chatbots.
Etwa 38 Stunden später stand der OMRChat in seiner Rudimentären form. 

 In den nächsten 3 Tagen wurden einige Optimierungsmaßnahmen getroffen, die dazu führten, dass der Chatbot kommunikativ wurde und Fragen immer besser beantworten konnte. Was ich dabei gelernt habe und wie das System funktioniert erfährst du in diesem Artikel.


## Das Grundgerüst
Um zu verstehen, wie der Chatbot funktioniert, müssen wir uns zunächst das zugrunde liegende System anschauen. Dieses lässt sich in drei Bereiche unterteilen:
1. Daten und Indexierung
2. Datenbank und Ranking
3. Die Hochzeit

---

## Daten und Indexierung
   >[!info]-  Diagramm
   >
   > ![diagram](notes/images/OMRChat-diagram.png)

Die erste Herausforderung bestand darin, alle relevanten Daten von Webseiten wie omr.com zu extrahieren.
Dafür habe ich einen eigenen OMRCrawler entwickelt, der einen Browser imitiert und versteht, wie der Content richtig abgerufen werden kann – sprich: die richtigen Buttons klicken, um Popups anzuzeigen um dann den wichtigen Text zu extrahieren.

Anschließend wurden die Textinhalte aufbereitet und in einer Vektor-Datenbank gespeichert.

---

## Vektor-Datenbank
Vektoren werden verwendet, um möglichst relevante Inhalte aus der Datenbank zu identifizieren – [ein Ansatz](https://code.google.com/archive/p/word2vec/), den auch Google nutzt, um semantische Beziehungen zwischen Wörtern zu verstehen und Suchanfragen mit relevanten Inhalten auf Websites abzugleichen.

Ein Text-Vektor ist eine Liste an Zahlen, die einen bestimmten Text Repräsentiert. Anders wie bei Buchstaben und Wörtern, können wir mit Zahlen die thematische Relevanz durch die Entfernung der Zahlen zueinander Berechnen. So können wir über eine reine Keyword-Suche hinausgehen und thematisch passende Inhalte aus dem OMR-Kosmos finden.

>[!faq]- Vektor-Embeddings einem Kind erklärt
>
> Stell dir Vektor Embeddings wie eine Schatzkarte vor, auf der jeder Schatz (Wort) einen bestimmten Platz hat. Jeder Schatz hat seine eigene Position, die durch Zahlen (Koordinaten) beschrieben wird. Ähnliche Schätze sind nah beieinander, während unterschiedliche Schätze weiter voneinander entfernt sind. Zum Beispiel könnten die Kategorien für Tiere ["Größe", "Geschwindigkeit" und "Freundlichkeit"] sein. Ein Hund hätte dann einen Vektor wie [5, 3, 7], weil er mittelgroß ist, nicht so schnell wie ein Gepard, aber sehr freundlich.

Wir verwenden hier die OpenAI Embeddings (text-embedding-ada-002), die in 1536 Dimensionen die Bedeutung eines Texts zuordnen Können.

---

##  Die Suchanfrage
Nachdem wir nun unsere Texte und deren Vektoren in einer Datenbank gespeichert haben, geht es darum, sie abzurufen.
Das war einer der größten Pains. Auch wenn die Ergebnisse besser wurden, gab es immer wieder Antworten auf einfachen Fragen die für einige Nutzer nicht beantwortet wurden. 
Um besser zu verstehen, welche Komponenten zu so einem Fehlverhalten führen, müssen wir diesen Abschnitt genauer verstehen.

In einem Chat kann es vorkommen, dass sich eine Folgefrage auf den Chatverlauf bezieht. Ein Beispiel:

> Frage 1: Wann sind die Doppelgänger zu sehen?
> 
>Antwort 1: Die Doppelgänger treten am 10/5/2023 von 16:20 bis 17:00 auf.
---
>Frage 2: Auf welcher Stage?
>
> Antwort 2: Der Podcast findet auf der Red Stage statt.

Würden wir immer die letzte Frage für die Suche nach passenden Inhalten verwenden, würden wir bei diesem Beispiel keine gute Antwort erhalten.

Die beste Lösung war die AI nach einer guten Datenbank Anfrage zu fragen. Also habe ich den Chatverlauf mit einigen Anweisungen an GPT-4 gesendet und zurück kam ein kurzer Satz, der für die Datenbank Abfrage genutzt werden kann.
Dieser Satz wird anschließend wie die Texte der Website in einen Vektor verwandelt – und voilà: Wir erhalten thematisch relevante Inhalte aus unserer Datenbank.

>[!info]- Hier ein Beispiel aus einem fiktiven Chatverlauf:
>
> ![inquiry template](notes/images/Inquiry-Template.png)

**Wer mehr über die Suchanfrage lernen möchte, kann unter [[notes/OMRChat/Inquiry Template]] die aktuelle und alte Version ansehen.**

>[!tip]- Für Nerds
>
>Durch die Berechnung mittels Cosine Similarity wurden die 1914 Inhalte gerankt und die Top 3 Ergebnisse zur Beantwortung der Frage herangezogen.

---


## Die Hochzeit
Im letzten Schritt senden wir die Textblöcke aus der Datenbank zusammen mit dem Chatverlauf des Nutzers und einer Anweisung an GPT-4. Als Ergebnis erhalten wir eine finale Antwort, die wir dem Nutzer anzeigen können.
**Eine Detailierte Beschreibung des Finalen Prompts findest du unter: [[notes/OMRChat/Final Prompt]]**

## Bonus: Prompt Engineering
Jetzt, wo wir Beispiele für die Arbeit mit Prompts gesehen haben, wollen wir über das Prompt-Engineering sprechen - eine neu entstehende Fähigkeit an sich.

Prompt-Engineering ist der Prozess, Eingabeabfragen oder Aufgaben sorgfältig zu gestalten, um die genauesten und nützlichsten Antworten von AIs zu erhalten. Obwohl diese Modelle unglaublich leistungsstark und vielseitig sind, brauchen sie ein wenig Anleitung, um die Arbeit wirklich korrekt zu erledigen.

Prompt-Engineering besteht aus drei Hauptkomponenten:

1.  **Formulierung**: Wir müssen mit verschiedenen Möglichkeiten experimentieren, unsere Eingabeabfragen zu präsentieren. Ziel ist es, die perfekte Balance zwischen Klarheit und Spezifität zu finden, um sicherzustellen, dass die AI genau versteht, wonach wir suchen.
2.  **Kontext**: Wir müssen unseren Aufforderungen Kontext hinzufügen, um der AI zu helfen, das größere Bild zu "verstehen". Dies kann beinhalten, Hintergrundinformationen bereitzustellen oder das Modell sogar sanft in eine bestimmte Denkrichtung zu lenken. Wie wir vorher gesehen haben, tun wir dies, um eine Anfrage zu erstellen, die auf Basis des Chatverlaufs des Benutzers relevant ist.
3.  **Anweisungen**: Wir müssen der AI klare und präzise Anweisungen geben. Wir müssen das gewünschte Format für die Antwort angeben oder wichtige Punkte hervorheben, die das Modell berücksichtigen soll. Wie du vorher gesehen hast, haben wir das getan, indem wir eine Liste von Anweisungen definiert haben, die genau zeigen, wie die Anfrage interpretiert werden soll und wie sie mit der vom Benutzer erhaltenen Aufforderung kombiniert werden soll.

Beim Prompt-Engineering geht es um trial and error, ein Wechselspiel aus Iteration und Optimierung. Indem wir unsere Prompts verfeinern, entwickeln wir ein tieferes Verständnis dafür, wie wir effektiv mit der AI kommunizieren können und verwandeln sie in ein zuverlässigeres und effizienteres Werkzeug.

## Wie gehts weiter
-> Zusammenfassungen
