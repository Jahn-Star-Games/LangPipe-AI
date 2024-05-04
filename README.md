# LangPipe AI

Introducing LangPipe, a new software framework for integrating Large Language Models (LLMs) into C# projects, including Unity. It is a user-friendly and basic framework developed as an alternative to LangChain for C#. LangPipe is currently in its early development stages and offers a simple framework without complex language features. However, it is optimized for Unity and can be used in various .NET projects. LangPipe supports multiple AI APIs, such as OpenAI, Gemini AI, and Vertex AI.

## Current and Planned Features:

- [x] Prompt Templates
- [x] LLMs: **Open AI, Google Vertex AI and Gemini AI models**
- [X] Vision: *Currently only available with Gemini Vision models*
- [x] Memory: *Buffer Memory, Summary Memory*
- [ ] Document Loaders
- [ ] Text Splitters
- [ ] Tools 
- [ ] AI Agents
- [ ] Embeddings
- [ ] Moderation
- [ ] Retrievers
- [ ] Output Parsers

### Installation

You can install LangPipe directly from our GitHub repository. Simply download the plugin and manually insert it into the Packages directory of your Unity project, or integrate via UPM with a Git reference as shown below:

##### via Git URL

1. Open the `Packages/manifest.json` file in your Unity project with a text editor.
2. Add the following line to the `dependencies` block of your `manifest.json`:
```json
{
  "dependencies": {
    "com.jahnstargames.langpipe": "https://github.com/Jahn-Star-Games/LangPipe-AI.git"
  }
}
```

## Getting Started

#### Examples

```csharp
using JahnStarGames.Langpipe;
using JahnStarGames.Langpipe.Providers.GoogleApis;

public class ChatSummaryMemoryChainExample : MonoBehaviour
{
    public bool newChat = true; // Test memory
    public string systemPromptTemplate = "Your name is {npc_name}. You are my best friend. And I am {user_name}.";
    public List<FormatPromptValue> systemPromptFormat = new() { new("npc_name", "Jessica"), new("user_name", "John") };
    [Space]
    public string userPrefix = "John";
    public string aiPrefix = "Jessica";
    public string userPrompt = "What's my name?";
    [Space]
    public List<Message> messages = new();
    [TextArea(3, 10)]
    public string result;
    private Pipeline pipeline;
    
    IChatModel model;
    private async Task Start()
    {
        // OpenAI chat model
        // model = new ChatOpenAIModel("sk-your-open-ai-api-key", new ChatOpenAIRequest { Model = "gpt-3.5-turbo", Temperature = 0.5f });
        // Prompt template example with basic call
        // result = await model.CallAsync(new PromptTemplate("What is the capital of {country}?", new() { new() { key = "country", value = "{Turkey}" } }).Format());

        // Google Gemini AI model
        // model = new ChatGeminiAIModel("your-gemini-ai-api-key", new ChatGeminiAIRequest { Model = "gemini-1.0-pro-vision", Temperature = 0.5f }, verbose: new(true));

        // Gemini vision call example
        // result = await model.CallAsync("Do you see any picture? If so, what do you see?", ContentDeclaration.ImageFromBase64("iVBORw0KGgoAAAANSUhEUgAAABQAAAAUCAYAAACNiR0NAAAAKElEQVQ4jWNgYGD4Twzu6FhFFGYYNXDUwGFpIAk2E4dHDRw1cDgaCAASFOffhEIO3gAAAABJRU5ErkJggg=="));

        // Google Vertex AI setup
        string vertex_ai_access_token = await GoogleQauth.GetAccessTokenFromJSONKeyAsync("Assets/your-key-file.json");
        model = new ChatGeminiAIModel(vertex_ai_access_token, "your-project-name", new ChatGeminiAIRequest { Model = "gemini-1.0-pro-vision", Temperature = 0.5f }, verbose: new(true));

        // Create a memory
        var memory = new SummaryMemory((model, (userPrefix, aiPrefix), 200), "history");

        pipeline = new(model, memory, verbose: new(debugHighVerbose: true));

        if (newChat)
        {
            pipeline.AddSystemMessage(systemPromptTemplate, systemPromptFormat)
                .AddUserMessage(userPrompt)
                .SaveMemory();
        }
        else pipeline.LoadMemory().AddUserMessage(userPrompt);

        result = await pipeline.RunAsync(); // Output: Your name is John.

        // for debugging purposes
        messages = pipeline.GetMessages();
    }

    [ContextMenu("Auth Conversation")]
    public async void AutoConversation()
    {
        // Basic call example:
        userPrompt = await model.CallAsync($"[Don't use prefix, be yourself. Don't be obsessive, be natural.] You are {userPrefix}, answer as {userPrefix}. You are having conversation with {aiPrefix}. \nChat: {string.Join("\n", messages.Select(m => (m.Role == Role.user ? userPrefix : aiPrefix) + ": " + m.Content))}");
        await Task.Delay(1000);
        await ConversationLoop();
    }
    
    [ContextMenu("Continue")]
    public async Task ConversationLoop()
    {
        pipeline.AddUserMessage(userPrompt)
        .SaveMemory();
        result = await pipeline.RunAsync();
        // debug
        messages = pipeline.GetMessages();
    }
}
```

*More examples are provided within the package to help you understand and utilize the full capabilities of LangPipe. 
You can find these [examples](https://github.com/Jahn-Star-Games/LangPipe-AI/tree/master/Runtime/Samples)*

## Supported LLM Providers

- **Open AI**: Utilizes models like GPT-3.5 for text processing.
- **Gemini AI**: Offers capabilities for both text and vision models.
- **Vertex AI**: Google's AI platform for llm and vision models.

## Contribution

LangPipe was developed out of a personal need and is tailored to my use as the developer. I’m not an expert on LangChain or any particular AI models, so contributions that can help improve LangPipe or extend its functionalities are warmly welcomed.

## License

LangPipe is released under the MIT License. See the [LICENSE](LICENSE) file for details.
