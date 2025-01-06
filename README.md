# AI-TUTOR-SYSTEM
import os
from flask import Flask, render_template, request, jsonify
from langchain_groq import ChatGroq
from langchain_core.prompts import ChatPromptTemplate
#from tools.wikipedia import get_wikipedia_content
from langchain.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper

# Create Flask app
app = Flask(__name__)

# Initialize Wikipedia API wrapper
wikipedia = WikipediaQueryRun(api_wrapper=WikipediaAPIWrapper())

# Function to get Wikipedia content
def get_wikipedia_content(topic):
    try:
        content = wikipedia.run(topic)
        return content
    except Exception as e:
        return f"Could not retrieve content from Wikipedia for the topic '{topic}'. Error: {e}"


# Temporary storage for user details (simulating data from a database)
TEMP_USER_STORAGE = [
    {
        'first_name': 'John',
        'last_name': 'Doe',
        'email': 'john.doe@example.com',
        'basic_computer_knowledge': 'Yes',
        'coding_knowledge': 7,
        'interests': 'AI, Python, Machine Learning'
    },
    {
        'first_name': 'Jane',
        'last_name': 'Smith',
        'email': 'jane.smith@example.com',
        'basic_computer_knowledge': 'No',
        'coding_knowledge': 4,
        'interests': 'Data Science, Deep Learning'
    }
]

# Function to create the mentor bot
def create_mentor_bot(groq_chat, user, wiki_content):
    system = """You are an expert in Python and AI with 20 years of experience. Your goal is to provide detailed, personalized responses to users. You must incorporate user preferences and relevant information from Wikipedia into your explanations. Be clear, concise, and context-aware, ensuring your responses are tailored to the user's background and interests. Adjust the difficulty and tone based on the user's coding knowledge and interests. End with a fun fact related to the topic to keep the user engaged."""

    def respond_to_query(query):
        human_template = """Explain the following content by heavily relying on information from Wikipedia, and tailor it to the user's preferences. Adjust the difficulty and tone based on the user's coding knowledge and interests. Use specific examples that align with the user's interests:
        User query: {query}
        Wikipedia Content: {wiki_content}
        User details:
        - First Name: {first_name}
        - Last Name: {last_name}
        - Basic Computer Knowledge: {basic_computer_knowledge}
        - Coding Knowledge Score: {coding_knowledge}
        - Interests: {interests}
        Provide a thorough, personalized response that reflects your expertise in Python and AI, incorporating examples and explanations related to the user's interests. End with a fun fact related to the topic."""

        human = human_template.format(
            query=query,
            wiki_content=wiki_content,
            first_name=user['first_name'],
            last_name=user['last_name'],
            basic_computer_knowledge=user['basic_computer_knowledge'],
            coding_knowledge=user['coding_knowledge'],
            interests=user['interests']
        )

        prompt = ChatPromptTemplate.from_messages([("system", system), ("human", human)])
        chain = prompt | groq_chat
        result = chain.invoke({"query": query})
        return result

    return respond_to_query


# Initialize Groq model
groq_chat = ChatGroq(groq_api_key='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', model_name='llama3-8b-8192')

# Flask route to handle user queries
@app.route('/', methods=['GET', 'POST'])
def ai_mentor():
    if request.method == 'POST':
        user_id = int(request.form['user'])  # Get user selection
        query = request.form['query']  # User query
        wiki_concept = request.form['wiki_concept']  # Wikipedia search term
        
        # Get selected user from temporary storage
        user = TEMP_USER_STORAGE[user_id]
        
        # Get Wikipedia content
        wikipedia_content = get_wikipedia_content(wiki_concept)

        # Create mentor bot for the user
        mentor_bot = create_mentor_bot(groq_chat, user, wikipedia_content)

        # Generate response
        response = mentor_bot(query)

        content = response.content

        # Print or use the cleaned content
        print(content)

                
        return jsonify({'response': str(content)})

    return render_template('mentor.html', users=TEMP_USER_STORAGE)


if __name__ == "__main__":
    app.run(debug=True)
