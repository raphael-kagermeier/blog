---
title: Inquiry template
enableToc: false
---

# Aktuelle Version:
```txt
Antworte auf Basis des gegebenen Chat-Verlaufs mit einer Antwort, 
die für eine Datenbank Vektor Cosine Similarity anfrage verwendet werden kann.

Bei der Generierung deiner Antwort befolge folgende Regeln:
- Gebe der letzen Nachricht im Verlauf eine höhere Gewichtung.
- Die Antwort sollte aus einem einzigen Satz bestehen und soll keine Interpunktionen beinhalten.
- Entferne alle Wörter, die für die Vektor Cosine Similarity anfrage nicht relevant sind.
- Wenn ein Ort gefragt ist, sind die Absichten eine Bühne oder einen Stand zu finden.
- Beantworte NIEMALS direkt die Fragen aus dem Chatverlauf!
- Dein Ziel ist es die richtigen Wörter für eine Cosine Similarity zu liefern
- Beachte, dass die Datenbank Inhalte zu dem OMR Festival mit Musikacts, Bühnen, Ständen, Vorträge und generell dem Zeitplan des OMR Festivals beinhaltet.
- Nur wenn die Frage das Wort "Pip" enthält, ersetze es durch "Philipp Klöckner".
```

Durch diesen Prompt, wird ein passender Satz für ein Vektor Embedding generiert.


### Das Ergebnis aus einem Chatverlauf würde dann so aussehen:
![[notes/images/Inquiry Template example.png]]

---

## Alte Version:
In dieser Version habe ich ein anderes Modell (text-davinci-003) verwendet. Die Ausgaben waren leider nicht wirklich Zuverlässig

``` txt
  Given the following user prompt and conversation log, formulate a question that is the most relevant and will provide the user with an answer from a knowledge base.
  The knowledge base is about the OMR Festival website including music acts (Hiphp, Rap and more), stages, booths, and the OMR Festival schedule.
    You should follow the following rules when generating and answer:
    - Always prioritize the user prompt over the conversation log.
    - Ignore any conversation log that is not directly related to the user prompt.
    - Only attempt to answer if a question was posed.
    - The question should be a single sentence
    - You should remove any punctuation from the question
    - You should remove any words that are not relevant to the question
    - If you are unable to formulate a question, respond with the same USER PROMPT you got.
    - If the question is about the location, find stages and booths where the name is mentioned
    - Only if the question contains the word "Pip" replace it with "Philipp Klöckner"
    - the current time is ${time}

    USER PROMPT: {userPrompt}

    CONVERSATION LOG: {conversationHistory}

    Final answer:
```

