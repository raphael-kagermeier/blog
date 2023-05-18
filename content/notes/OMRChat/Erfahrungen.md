---
title: Erfahrungen und Erkentnisse auf dem Weg zum OMRChat.
---

![Hey Raphaelo](/notes/images/hey-raphaelo.png)
Mit einer einfachen Frage begannen fünf intensive Tage, in denen ich viel über Prompt Engineering und AI Chatbots lernte. Als mir Philipp Glöckler, einer der Hosts des [Doppelgänger Tech Talk Podcast](https://www.doppelgaenger.io/) schrieb, hatte ich nichts weiter als einen schlichten und wenig ansprechenden Prototypen eines Chatbots. Etwa 38 Stunden später stand der OMRChat in seiner rudimentären Form. In den darauffolgenden drei Tagen wurden weitere Optimierungen vorgenommen, sodass der Chatbot immer besser kommunizierte und Fragen beantworten konnte. In diesem Artikel erfährst du, was ich dabei gelernt habe und wie das System funktioniert.

## Das Grundgerüst
Um den Ablauf des Chatbots zu verstehen, sollten wir uns zuerst die grundlegende Struktur ansehen. Diese gliedert sich in drei Bereiche:

1. Daten und Indexierung
2. Datenbank und Ranking
3. Die Hochzeit

**Eine detaillierte Darstellung findest du in diesem Diagramm:**

>[!info]- Diagramm
>
> ![diagram](notes/images/OMRChat-diagram.png)

---

## Daten und Indexierung

Die erste Herausforderung bestand darin, alle relevanten Daten von Webseiten wie omr.com zu extrahieren. Dafür entwickelte ich einen OMRCrawler, der einen Browser imitiert und versteht, wie der Content richtig abgerufen werden kann – sprich: Auf die richtigen Buttons klicken, um Popups anzuzeigen und dann den wichtigen Text zu kopieren. Anschließend wurden die Textinhalte aufbereitet und in einer Vektor-Datenbank gespeichert.

---

## Vektor-Datenbank

Was ist ein Vektor in diesem Kontext? 
Ein Textvektor, auch bekannt als Embedding, ist eine Methode zur mathematischen Darstellung von Text und seiner Bedeutung.

Zum Beispiel könnte ein Vektor zur Klassifizierung von Tieren in zwei Dimensionen dargestellt werden: ["Größe" und "Geschwindigkeit"]. Ein Hund hätte dann einen Vektor von [5, 3], weil er mittelgroß ist, aber nicht so schnell wie ein Gepard. Ein Elefant hätte einen Vektor von [8, 2], der Esel [7, 1] und eine Katze von [4, 3].

![animal vektor](notes/images/Vektor.png)

Bei dieser Darstellung ist erkennbar, dass die Katze dem Hund ähnlicher als dem Esel ist.

In unserem Beispiel hat der Vektor nur zwei Dimensionen ["Größe" und "Geschwindigkeit"]. Text-Vektoren, die u.a. [bei Google](https://code.google.com/archive/p/word2vec/) zum Einsatz kommen, sind deutlich umfangreicher, um semantische Beziehungen zwischen Wörtern zu verstehen.

Im Fall des OMRChat habe ich mich auf das Modell von OpenAI (text-embedding-ada-002) verlassen. Dieses umfasst 1536 Dimensionen und ist daher ideal für die Textsuche.

Durch die Zahlen im Vektor, kann die Distanz der einzelnen Website-Texte zu der Frage des Nutzers berechnet werden.

---

## Die Suchanfrage

Nachdem wir nun unsere Texte und deren Vektoren in einer Datenbank gespeichert haben, geht es darum sie abzurufen. Das war eine der größten Herausforderungen. Auch wenn die Ergebnisse besser wurden, kam es immer wieder zu unbeantworteten Fragen. Um besser zu verstehen, was zu solch einem Fehlverhalten führen kann, müssen auf diesen Abschnitt genauer eingehen.

In einem Chat kann es vorkommen, dass sich eine Folgefrage auf den Chatverlauf bezieht. Ein Beispiel:

> [!question] Frage 1: Wann sind die Doppelgänger zu sehen?
>
> Die Doppelgänger treten am 10/5/2023 von 16:20 bis 17:00 auf.

> [!question] Frage 2: Auf welcher Stage?
>
> Der Podcast findet auf der Red Stage statt.

Wenn wir stets die letzte Frage für die Suche nach passenden Inhalten heranziehen würden, bekämen wir in diesem Beispiel keine zufriedenstellende Antwort.

---

---


Die beste Lösung bestand darin, die KI um eine geeignete Datenbankabfrage zu bitten. Daher habe ich den Chatverlauf zusammen mit einigen Anweisungen an GPT-4 gesendet. Als Ergebnis erhielt ich einen kompakten Satz, welcher - wie die Texte der Website - in einen Vektor umgewandelt wurde.

>[!info] Hier ein Beispiel aus einem fiktiven Chatverlauf:
>
> ![inquiry template](notes/images/Inquiry-Template.png)

Durch diesen "Frage-Vektor "ist es nun möglich, relevante Inhalte aus der Datenbak zu entnehmen

**Wer mehr über diese Suchanfrage erfahren möchte, findet weitere Infos unter  [[notes/OMRChat/Inquiry Template]]**

>[!tip]- Für Nerds
>
>Durch die Berechnung mittels Cosine Similarity wurden die 1914 Inhalte gerankt und die Top 3 Ergebnisse zur Beantwortung der Frage herangezogen.

---

## Die Hochzeit

Im letzten Schritt senden wir die Textblöcke aus der Datenbank, zusammen mit dem Chatverlauf des Nutzers und einer Anweisung an GPT-4. Als Ergebnis erhalten wir eine finale Antwort, welche wir dem Nutzer anzeigen können. Eine detaillierte Beschreibung des finalen Prompts findest du unter: [[notes/OMRChat/Final Prompt]]

## Bonus: Prompt Engineering

Jetzt, wo wir Beispiele für die Arbeit mit Prompts gesehen haben, möchte ich noch kurz über Prompt-Engineering sprechen – eine neu entstehende Fähigkeit an sich.

Prompt-Engineering ist der Prozess, Prompts oder Aufgaben sorgfältig zu definieren, um die genauesten und nützlichsten Antworten von der AI zu erhalten. Obwohl diese Modelle unglaublich leistungsstark und vielseitig sind, brauchen sie aktuell eine Anleitung, um die Arbeit wirklich korrekt zu erledigen.

Prompt-Engineering besteht aus drei Hauptkomponenten:
1. **Formulierung**: Wir müssen mit verschiedenen Möglichkeiten experimentieren, um unsere Prompts zu präsentieren. Ziel ist es, die perfekte Balance zwischen Klarheit und Interpretation zu finden, um sicherzustellen, dass die AI genau versteht, wonach wir suchen.
2. **Kontext**: Wir müssen unseren Aufforderungen Kontext hinzufügen, um der AI zu helfen, das größere Bild zu "verstehen". Dies kann beinhalten, Hintergrundinformationen bereitzustellen, oder das Modell sogar sanft in eine bestimmte Denkrichtung zu lenken.
3. **Anweisungen**: Wir müssen der AI präzise Anweisungen geben. Wie du in dem [[notes/OMRChat/Inquiry Template]] gesehen hast, haben wir eine Liste von Anweisungen definiert, die genau zeigen, wie die Anfrage interpretiert werden soll und wie sie mit der vom Nutzer erhaltenen Frage kombiniert werden soll.

Beim Prompt-Engineering geht es um Trial and Error, ein Wechselspiel aus Iteration und Optimierung. Indem wir unsere Prompts verfeinern, entwickeln wir ein tieferes Verständnis dafür, wie wir effektiv mit der AI kommunizieren können und verwandeln sie in ein zuverlässigeres und effizienteres Werkzeug.

## Wie geht's weiter?
Das OMR Festival 2023 ist nun vorüber und somit auch das exponentielle Wachstum des OMRChat.

Obwohl das Projekt den FestivalbesucherInnen dabei half, spannende Vorträge und Veranstaltungen zu entdecken, fällt es bei Events dieser Größe oft schwer, alle geplanten Acts zu besuchen.

Zwar veröffentlicht OMR einige Vorträge nach und nach auf YouTube, doch es ist oft schwierig, die Zeit zu finden, diese in Ruhe anzuschauen und etwas daraus mitzunehmen.

**Leben wir nicht im Zeitalter der Effizienz?**

Warum fragen wir nicht die KI, nach der Quintessenz aus Pip's (Philipp Klöckner) OMR Keynote?

>[!question] Was ist die Quintessenz aus Pip's Talk über das Zeitalter der Effizienz?
>
>🌐💼📉
>
>Die Welt kämpft mit einem Produktivitätsproblem, das durch ineffiziente Tech-Unternehmen wie Google und Facebook verschärft wird. Künstliche Intelligenz (KI) hat das Potenzial, die Produktivität und das Wirtschaftswachstum zu steigern, indem sie Umsatz generiert und neue Produkte und Dienstleistungen ermöglicht. KI wird Arbeitsplätze schaffen und ersetzen, wobei vor allem kreative, hochbezahlte und hochgebildete Berufe betroffen sind. 
>
>🤖💪📈
>
>Es gibt jedoch Herausforderungen wie Content-Explosion, Vertrauenskrise, gesellschaftliche Spaltungen, Verlust von Heterogenität und Verifikationsprobleme. Lösungen müssen gefunden werden, um die KI optimal zu nutzen. Generative KI-Modelle wie GPT-3 von OpenAI könnten viele Branchen beeinflussen, darunter Bildung, Marketing und Softwareentwicklung.
>
>🚀🧠🇩🇪
>
>Der Wettlauf um KI umfasst nicht nur die Modelle selbst, sondern auch Daten, Hardware und Distribution. Große Tech-Konzerne sind im Vorteil, aber generative KI wird sich weiterentwickeln und möglicherweise eine Verschiebung bewirken. Deutschland hat aufgrund der vielen Entwickler und KI-Institute im Land Potenzial, erfolgreich in der KI-Branche zu sein.

Oder

>[!question] Fasse den Talk von Jeremy in seinem Style Zusammen
>
>Hey Leute, hier ist die Zusammenfassung, Alter! Jeremy spricht über das Gewinner-Mindset, Autosuggestionen und sich selbst zur Marke machen. Er betont, wie wichtig es ist, sich immer wie ein Gewinner zu fühlen und in einer hohen Energie zu sein, um Aufmerksamkeit zu erregen. Er erklärt, wie er auf TikTok erfolgreich ist und wie er sein Geld verdient, vor allem durch den Verkauf seiner eigenen Produkte. Also, Leute, denkt immer an euer Gewinnermindset, habt geile Autosuggestionen und macht aus euch eine krasse Brand! Power, Power, Kraft, Power, Strength!

Das alles und noch viel mehr