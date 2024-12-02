# Health and Wellness Agent Tutorial

## Introduction

In this tutorial, we will build a health and wellness agent that provides users with information and advice on various health topics. The agent will be implemented as a chatbot using natural language processing (NLP) techniques to understand user queries and provide relevant responses. We will also integrate the agent with a database of health information to ensure accurate and up-to-date information.

## Prerequisites

Before starting this tutorial, you should have a basic understanding of Python programming and some familiarity with NLP concepts. Additionally, you will need to have the following libraries installed:

- `nltk`
- `spacy`
- `pandas`
- `flask`
- `sqlalchemy`

You can install these libraries using pip:

```bash
pip install nltk spacy pandas flask sqlalchemy
```

## Step 1: Setting Up the Project

First, create a new directory for your project and navigate to it:

```bash
mkdir health_wellness_agent
cd health_wellness_agent
```

Next, create a virtual environment to manage your project dependencies:

```bash
python -m venv venv
source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
```

## Step 2: Creating the Database

We will use SQLite as our database to store health information. Create a new file called `database.py` and add the following code to set up the database:

```python
from sqlalchemy import create_engine, Column, Integer, String, Text
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

Base = declarative_base()

class HealthInfo(Base):
    __tablename__ = 'health_info'
    id = Column(Integer, primary_key=True)
    topic = Column(String, nullable=False)
    content = Column(Text, nullable=False)

engine = create_engine('sqlite:///health_wellness.db')
Base.metadata.create_all(engine)
Session = sessionmaker(bind=engine)
session = Session()
```

## Step 3: Populating the Database

Next, we need to populate the database with health information. Create a new file called `populate_db.py` and add the following code:

```python
from database import session, HealthInfo

health_data = [
    {"topic": "hydration", "content": "Drinking enough water is essential for maintaining good health. Aim to drink at least 8 cups of water a day."},
    {"topic": "exercise", "content": "Regular physical activity can help improve your overall health and fitness. Aim for at least 30 minutes of moderate exercise most days of the week."},
    {"topic": "nutrition", "content": "Eating a balanced diet with a variety of nutrients is important for maintaining good health. Include plenty of fruits, vegetables, whole grains, and lean proteins in your diet."},
    {"topic": "sleep", "content": "Getting enough sleep is crucial for your overall health and well-being. Aim for 7-9 hours of sleep each night."},
    {"topic": "stress management", "content": "Managing stress is important for maintaining good mental and physical health. Practice relaxation techniques such as deep breathing, meditation, or yoga."}
]

for data in health_data:
    health_info = HealthInfo(topic=data["topic"], content=data["content"])
    session.add(health_info)

session.commit()
```

Run the script to populate the database:

```bash
python populate_db.py
```

## Step 4: Building the Chatbot

Now, let's build the chatbot. Create a new file called `chatbot.py` and add the following code:

```python
import random
from flask import Flask, request, jsonify
from database import session, HealthInfo
import spacy

nlp = spacy.load("en_core_web_sm")
app = Flask(__name__)

def get_health_info(topic):
    health_info = session.query(HealthInfo).filter_by(topic=topic).first()
    if health_info:
        return health_info.content
    else:
        return "I'm sorry, I don't have information on that topic."

@app.route("/chat", methods=["POST"])
def chat():
    user_input = request.json.get("message")
    doc = nlp(user_input)
    topics = [ent.text.lower() for ent in doc.ents if ent.label_ == "TOPIC"]
    if topics:
        response = get_health_info(topics[0])
    else:
        response = "I'm sorry, I didn't understand your question. Please ask about a specific health topic."
    return jsonify({"response": response})

if __name__ == "__main__":
    app.run(debug=True)
```

## Step 5: Testing the Chatbot

To test the chatbot, run the following command:

```bash
python chatbot.py
```

The chatbot will start running on `http://127.0.0.1:5000`. You can use a tool like Postman to send a POST request to the `/chat` endpoint with a JSON payload containing a message. For example:

```json
{
    "message": "Tell me about hydration."
}
```

The chatbot should respond with information about hydration.

## Additional Resources and References

- [Natural Language Toolkit (nltk)](https://www.nltk.org/)
- [spaCy](https://spacy.io/)
- [Flask](https://flask.palletsprojects.com/)
- [SQLAlchemy](https://www.sqlalchemy.org/)

