
# Welcome to DracoArts

![Logo](https://dracoarts-logo.s3.eu-north-1.amazonaws.com/DracoArts.png)



# OpenAI Integration in Unity Example
OpenAI's suite of artificial intelligence models offers game developers unprecedented opportunities to create dynamic, intelligent, and immersive experiences in Unity. By integrating OpenAI's API, developers can harness cutting-edge AI capabilities to revolutionize gameplay mechanics, automate content creation, and deliver next-generation interactive storytelling. This integration bridges the gap between traditional game design and AI-powered innovation, enabling features that were previously impossible or extremely difficult to implement.

# Deep Dive into OpenAI's Capabilities for Unity

## 1. Natural Language Processing for Enhanced Interactions
- OpenAI's language models (GPT-3.5, GPT-4) bring human-like conversational abilities to Unity projects:

### Dynamic NPC Dialogues:
 - Non-player characters can engage in meaningful, context-aware conversations that adapt to player input in real-time, moving beyond pre-scripted dialogue trees.

### Procedural Story Generation:
 - Games can generate quests, lore, and narrative arcs on-the-fly, creating unique experiences for each player while maintaining coherent storytelling.

### Intelligent Game Help Systems: 
- AI can analyze player behavior and provide contextual hints or tutorials in natural language, improving the learning curve without breaking immersion.

## 2. AI-Generated Visual Content with DALL·E
 - The DALL·E image generation model opens new possibilities for visual content creation:

### Procedural Asset Generation: 
- Generate unique textures, item icons, or environmental assets based on game context or player input.

### Dynamic World Building:
 - Create evolving in-game art displays, posters, or magical portals that change based on narrative progression.

### Concept Art Acceleration: 
- Rapidly prototype character designs, environments, and UI elements during development.

## 3. Speech Recognition and Synthesis
- Combining Whisper (speech-to-text) and TTS (text-to-speech) technologies enables:

### Voice-Controlled Gameplay:
 - Players can issue verbal commands, creating more immersive RPG or strategy experiences.

### Accessibility Features:
 - Voice navigation for players with mobility challenges.

### Real-Time Voice AI:
 - NPCs that can listen and respond to player speech with appropriate lip-syncing.

 ## 4. AI-Assisted Game Design and Balancing
Beyond player-facing features, OpenAI can assist in development:

### Automated Playtesting Analysis: 
- AI can process playtest feedback at scale, identifying common pain points.

### Difficulty Adjustment: 
- Machine learning can dynamically tune challenge levels based on player performance metrics.

### Bug Detection:
 - Natural language processing can help analyze error logs and suggest fixes.

 ## Architecture of OpenAI Integration
 The typical integration follows a client-server model:

## Unity Client: 
- Handles player input and rendering

## API Bridge:
 - Manages secure communication with OpenAI's servers

## Response Processing: 
- Interprets AI outputs for game integration

# Data Flow Process
- Player triggers an AI interaction (text input, voice command, etc.)

- Unity packages the request with appropriate context

- Request is securely transmitted to OpenAI's API

- AI processes the input and generates a response

- Unity receives and parses the response

- Game systems implement the AI's output (display text, generate image, trigger gameplay event)

# Key Components for OpenAI in Unity

## 1. OpenAI API Access

- Requires an API key from [OpenAI’s platform](https://platform.openai.com/docs/overview)
### Supports multiple AI models:
####  GPT-4 / GPT-3.5
- Text generation and conversation.

#### DALL·E
 - AI-generated images.

#### Whisper 
-  Speech-to-text.

#### Embeddings 
 - Semantic search and recommendations.

## 2. Unity’s Networking System
- OpenAI integration relies on HTTP requests (REST API).

### Unity supports API calls via:

- UnityWebRequest (Built-in networking).

- Third-party libraries (REST clients, WebSocket solutions).

## 3. Data Handling
- JSON Parsing – Required for processing OpenAI responses.

- Error Handling – Managing API limits, network issues, and invalid responses.


# Prerequisites
### Importing the Package
- Unity Version: 2019.4 LTS or newer (recommended)

- Go to Window > Package Manager
- Click the + button and select Add package from git URL
Paste the repository URL 

                  https://github.com/srcnalt/OpenAI-Unity.git 
                  
and click Add
### Setting Up Your OpenAI Account
- To use the OpenAI API, you need to have an OpenAI account. Follow these steps to create an account and generate an API key:

- Go to https://openai.com/api and sign up for an account
- Once you have created an account, go to https://beta.openai.com/account/api-keys
- Create a new secret key and save it
- Saving Your Credentials
- To make requests to the OpenAI API, you need to use your API key and organization name (if applicable). To avoid exposing your API key in your Unity project, you can save it in your device's local storage.

### To do this, follow these steps:

Create a folder called .openai in your home directory (e.g. C:User\UserName\ for Windows or ~\ for Linux or Mac)
Create a file called auth.json in the .openai folder
Add an api_key field and a organization field (if applicable) to the auth.json file and save it
Here is an example of what your auth.json file should look like:

              {
    "api_key": "sk-...W6yi",
    "organization": "org-...L7W"
     }
You can also pass your API key into OpenAIApi ctor when creating an instance of it but again, this is not recommended!

     var openai = new OpenAIApi("sk-Me8...6yi");
## Usage/Examples
    
        [SerializeField] private InputField inputField;
        [SerializeField] private Button button;
        [SerializeField] private ScrollRect scroll;
        
        [SerializeField] private RectTransform sent;
        [SerializeField] private RectTransform received;

        private float height;
        private OpenAIApi openai = new OpenAIApi();

        private List<ChatMessage> messages = new List<ChatMessage>();
        private string prompt = "Act as a random stranger in a chat room and reply to the questions. Don't break character. Don't ever mention that you are an AI model.";

        private void Start()
        {
            button.onClick.AddListener(SendReply);
        }

        private void AppendMessage(ChatMessage message)
        {
            scroll.content.SetSizeWithCurrentAnchors(RectTransform.Axis.Vertical, 0);

            var item = Instantiate(message.Role == "user" ? sent : received, scroll.content);
            item.GetChild(0).GetChild(0).GetComponent<Text>().text = message.Content;
            item.anchoredPosition = new Vector2(0, -height);
            LayoutRebuilder.ForceRebuildLayoutImmediate(item);
            height += item.sizeDelta.y;
            scroll.content.SetSizeWithCurrentAnchors(RectTransform.Axis.Vertical, height);
            scroll.verticalNormalizedPosition = 0;
        }

        private async void SendReply()
        {
            var newMessage = new ChatMessage()
            {
                Role = "user",
                Content = inputField.text
            };
            
            AppendMessage(newMessage);

            if (messages.Count == 0) newMessage.Content = prompt + "\n" + inputField.text; 
            
            messages.Add(newMessage);
            
            button.enabled = false;
            inputField.text = "";
            inputField.enabled = false;
            
            // Complete the instruction
            var completionResponse = await openai.CreateChatCompletion(new CreateChatCompletionRequest()
            {
                Model = "gpt-4o-mini",
                Messages = messages
            });

            if (completionResponse.Choices != null && completionResponse.Choices.Count > 0)
            {
                var message = completionResponse.Choices[0].Message;
                message.Content = message.Content.Trim();
                
                messages.Add(message);
                AppendMessage(message);
            }
            else
            {
                Debug.LogWarning("No text was generated from this prompt.");
            }

            button.enabled = true;
            inputField.enabled = true;
        }
## Images

Text Chat

![](https://github.com/AzharKhemta/Gif-File-images/blob/main/OpenAi%20Unity%20Part%201.gif?raw=true)

Image Generate

![](https://github.com/AzharKhemta/Gif-File-images/blob/main/OpenAi%20%20Unity%20Part%202.gif?raw=true)

## Authors

- [@MirHamzaHasan](https://github.com/MirHamzaHasan)
- [@WebSite](https://mirhamzahasan.com)


## 🔗 Links

[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/company/mir-hamza-hasan/posts/?feedView=all/)
## Documentation

[Open Ai](https://platform.openai.com/docs/api-reference/introduction)




## Tech Stack
**Client:** Unity,C#

**Plugin:**  OpenAi-Unity 



