# Chatbot-using-RASA-and-AWS
build a chatbot initial setup for me using RASA and AWS
=========
To build a chatbot using Rasa and deploy it on AWS, you'll need to follow a structured approach. Here's a step-by-step guide and Python code to set up the initial environment and deploy the chatbot on AWS.
Step 1: Install Rasa and Dependencies

First, you need to install Rasa and its dependencies. This can be done using pip. Create a virtual environment and install Rasa.

# Create a virtual environment (optional but recommended)
python3 -m venv rasa_env
source rasa_env/bin/activate  # On Windows, use `rasa_env\Scripts\activate`

# Install Rasa
pip install rasa

Step 2: Create a New Rasa Project

Once Rasa is installed, create a new Rasa project using the following command:

rasa init --no-prompt

This will create a project directory with a predefined structure:

.
├── actions.py
├── config.yml
├── credentials.yml
├── domain.yml
├── endpoints.yml
├── models/
├── data/
│   ├── nlu.yml
│   └── stories.yml
├── tests/
└── rasa/

Step 3: Configure Rasa

Now, let’s configure the core parts of the bot, like intents, entities, and actions.

    Define Intents: In the data/nlu.yml, add different intents for your chatbot. For example:

version: "2.0"
nlu:
  - intent: greet
    examples: |
      - hello
      - hi
      - hey

  - intent: goodbye
    examples: |
      - bye
      - goodbye
      - see you

  - intent: inform
    examples: |
      - I need help with my order
      - Can you help me with billing?

    Define Responses: In domain.yml, define the responses the bot should return when certain intents are detected:

version: "2.0"

intents:
  - greet
  - goodbye
  - inform

responses:
  utter_greet:
    - text: "Hello! How can I assist you today?"
  
  utter_goodbye:
    - text: "Goodbye! Have a great day."

  utter_inform:
    - text: "Sure! What information do you need help with?"

    Create Stories: In data/stories.yml, define stories which are sequences of user inputs and bot actions:

version: "2.0"
stories:
  - story: user greets
    steps:
      - intent: greet
      - action: utter_greet

  - story: user says goodbye
    steps:
      - intent: goodbye
      - action: utter_goodbye

  - story: user needs help
    steps:
      - intent: inform
      - action: utter_inform

    Train the Model: After setting up the intents, actions, and stories, you can train the model.

rasa train

Step 4: Test the Rasa Chatbot Locally

To test your bot locally:

rasa shell

This will run the bot in an interactive shell. You can talk to your bot and verify its responses.
Step 5: Deploying Rasa on AWS

Now that your bot is ready locally, let's deploy it on AWS.

    Set Up an EC2 Instance on AWS
        Log in to the AWS Management Console.
        Go to EC2 service and create a new EC2 instance.
        Choose an appropriate instance type (e.g., t2.micro for testing).
        Make sure to open ports 80, 443 (for HTTP/HTTPS), and 5005 (for Rasa).
        Select a key pair for SSH access.

    Install Dependencies on EC2

    SSH into your EC2 instance:

ssh -i <your-key-file>.pem ubuntu@<ec2-public-ip>

Install necessary packages:

# Update the package list
sudo apt-get update

# Install Python 3 and pip
sudo apt-get install python3-pip python3-dev

# Install Rasa
pip3 install rasa

# Install Docker (optional for containerization)
sudo apt-get install docker.io

    Copy Your Rasa Project to EC2

You can copy your local project directory to the EC2 instance using scp or a similar file transfer method:

scp -i <your-key-file>.pem -r ./my-rasa-bot ubuntu@<ec2-public-ip>:/home/ubuntu/

    Run the Rasa Server on EC2

Once the files are transferred, navigate to your Rasa bot directory on the EC2 instance:

cd my-rasa-bot

Run the Rasa action server (if any actions are defined):

rasa run actions

Run the Rasa server:

rasa run --enable-api --cors "*"

This will start the Rasa server and expose the bot’s API for handling requests.
Step 6: Expose the Bot’s API to the Web

Since AWS EC2 instances are behind a firewall, ensure you have the right security group settings to allow inbound traffic on port 5005.

    Modify Security Groups: In the AWS console, go to EC2 > Security Groups > Edit Inbound Rules. Add a rule for port 5005 (the Rasa default port).

    Access the Bot: Once the Rasa server is running, you can access it at http://<ec2-public-ip>:5005/webhooks/rest/webhook to send messages and get responses from your bot via a REST API.

Step 7: Optional - Set Up a Web Interface

To make your bot available to users via a web interface, you can use Flask or Django to create a frontend. Here’s a basic example of using Flask:

Install Flask:

pip install flask

Create a app.py file with Flask:

from flask import Flask, request, jsonify
import requests

app = Flask(__name__)

RASA_URL = "http://<ec2-public-ip>:5005/webhooks/rest/webhook"

@app.route("/chat", methods=["POST"])
def chat():
    user_message = request.json.get("message")
    
    response = requests.post(RASA_URL, json={"message": user_message})
    
    return jsonify(response.json())

if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0", port=8080)

Run the Flask app:

python app.py

This will expose a REST API on http://<your-ec2-ip>:8080/chat to send and receive messages from your Rasa chatbot.
Step 8: Set Up a Domain and SSL (Optional)

For production environments, it's recommended to set up a domain (via AWS Route 53 or other DNS providers) and use SSL (via Let's Encrypt or AWS ACM) to secure the connection.
Conclusion

By following these steps, you have:

    Set up a basic Rasa chatbot.
    Deployed it on AWS EC2.
    Exposed its API and tested it.
    Optionally set up a web interface using Flask.

You can now scale and improve the chatbot, integrate with other services (e.g., databases, external APIs), and add more complex functionalities such as user authentication, logging, etc.
