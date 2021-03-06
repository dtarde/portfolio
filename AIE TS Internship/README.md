# Health Check-up Chatbot

This here is a simple example of how a symptom-checking chatbot may be built using Infermedica API and 300 lines
of Python code.

The project is merely a tutorial. It's not intended to be a production-ready solution.
The entire interaction takes place in console. No intent recognition is performed and the bot is pretty ignorant
when it comes to understanding user's actions other than blindly following the instructions.

## How to run the example

You'll need an Infermedica account. If you don't have one, please register at https://developer.infermedica.com.

The code is written in Python 3. First, install the requirements (preferably within a dedicated virtualenv):

```
pip install -r requirements.txt
```

To run the demo:

```
python chat.py 1cda2064:e7110ead980d529eea75c0f8c5bb2ebc
```

Please use the `APP_ID` and `APP_KEY` values from your Infermedica account.

The demo starts by asking for patient's age and sex. Then you're expected to provide complaints.
You can provide complaints in one or multiple messages. You need to reply with an empty string to let the bot know
you're finished with the complaints. A real-life bot should understand replies such as _I don't know_, _that's it_, etc.

The complaints are analysed by using the `/parse` endpoint of the Infermedica API.

The bot will start asking diagnostic questions obtained from the `/diagnosis` endpoint of the Infermedica API.
At this stage, answers of `yes` (or `y`), `no` (`n`) or `dont know` (or `skip`) are expected.
The interview will finish when the diagnostic API responds with a `should_stop` flag.
This flag is raised when the engine reaches conclusion that enough is known or there have already been too many
questions to bother the user further.

See [an example session](example_session.txt).

## What is not covered here

 1. **Multiple users and event-driven logic**.
 A proper chatbot (such as https://symptomate.com/chatbot/) needs to handle multiple users.
 You need to have a database where you'd keep the state of conversation with each user and have it updated
 each time an interaction takes place. Also, a real chatbot will need to handle events coming from external components
 (e.g. your own chatbot front-end or third-party platform such as Google Assistant).
 So, the bot back-end should probably be a REST app (e.g. made with Flask or UWSGI).
 In such a setting, this would be an event-driven program. The application would expose a _handle message_ endpoint
 that would be called with user id, user message text and possibly some settings.
 The endpoint would be responsible for retrieving the state of conversation with the requesting user,
 handling the user's message, altering the state, storing the altered state back in the database and returning response
 messages.

 2. **Intent recognition**. The users don't always do what they're told. Also, language is flexible.
 Some basic intent recognition will be useful to know when the user wants to proceed to next stage, to quit the
 conversation, perhaps restart or request for help.
 In the most crude form this could be handled by a list of most likely answers or regular expressions;
 but in the long run you should consider using open-source tools such as _Snips NLU_ or _RASA NLU_.
 This would also allow you to understand parameters such as age and sex (via _slots_ of _intents_).

 3. **Custom flow**. You must know the user's age and sex before calling `/diagnosis`.
 But you're free to choose if you want to read complaints first or age and sex.
 Any case of age below 12 years old should be rejected as the Infermedica's engine does not cover paediatrics yet
 (at the moment of writing).
 For legal reasons you may need to use higher age threshold, though.
 Also, you can consider an option to allow the user to add complaints later on during the interview (if the user
 explicitly wants to add something instead of responding to the last question).
 
 4. **Group questions**. If you're developing a voice app or simple text-based chat, it's hard to handle medical
 questions other than the simple ones (yes, no, don't-know). For such scenarios the `disable_groups` mode is intended
 (see [apiaccess.py](apiaccess.py)). If your chatbot is more of a rich conversational UI, then you may as well support
 all of the question types and not need this mode.
