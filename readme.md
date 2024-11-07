
# Meeting Management Tool

## Demo

[![Watch the video](https://github.com/user-attachments/assets/35a9cf1f-fd78-4b34-9ff0-1ee154627dd1)](https://drive.google.com/file/d/18Hp5lJNZYLY2MWipr_S9sqiXsm3fM_ay/view?usp=sharing)

## Features

### 1. Pre-Meeting Document Management
- **Upload Documents**: Users can upload relevant documents, which are stored in the local server with document paths stored in MongoDB Atlas.
- **Add Discussion Points**: Participants can contribute discussion points before the meeting, which are then fed into Gemini AI to be organized into a coherent agenda.

### 2. AI-Powered Agenda Creation
- **Automatic Agenda Generation**: Gemini AI analyzes the discussion points added by participants, organizing them into a structured agenda. It links related topics to streamline meeting flow, enabling a seamless pre-meeting organization process.

### 3. Meeting Recording and Tracking
- **Video Transcription**: After the meeting, users can submit a video recording, which is processed using AssemblyAI for transcription. The transcription helps the system track which discussion points were covered and flags unresolved issues. (Requires ffmpeg to convert video to audio first.)
- **Unresolved Issues**: The tool highlights any unresolved discussion points and tracks them for future follow-up.

### 4. Post-Meeting Summary Generation
- **Comprehensive Summaries**: After the meeting, a detailed summary is generated using Gemini AI. This summary includes:
  - Topics discussed.
  - Key decisions made.
  - Action items assigned, along with the responsible participants.
- **RAG Implementation**: Summary generation is enhanced through Retrieval-Augmented Generation (RAG), leveraging the Pinecone vector database to retrieve relevant meeting data from past sessions, making the summary more context-aware.

## System Architecture

### Frontend (React)
- The frontend is built using React to provide an intuitive and user-friendly interface for managing meetings. Users can:
  - Upload documents and videos.
  - Add discussion points.
  - View agendas and summaries.
  - View tracing points.
  - View video and download documents.

### Backend (Node.js, Express)
- The backend API handles all meeting-related operations, including document uploads, video submissions, and interaction with AI models and the database.

### Database (MongoDB Atlas)
- MongoDB Atlas is used to store:
  - User and meeting data.
  - Paths of uploaded documents and videos.
  - Discussion points.
  - Agendas and summaries.

## AI Integration

- **Gemini AI**:
  - **Pre-Meeting**: Used to organize discussion points into a structured agenda and link related topics uploaded by the user.
  - **Post-Meeting**: Generates meeting summaries based on transcription data and retrieved information from past meetings (RAG).
  - **Post-Meeting**: Tracks which points have been discussed and flags any unresolved issues.
- **AssemblyAI**:
  - Processes video recordings to generate transcriptions, which help track discussed points and flag unresolved topics.

### Vector Database (Pinecone)
- **RAG Process**: Meeting data is stored in Pinecone as embeddings, which are retrieved during post-meeting summary generation. This ensures context from prior meetings is utilized to enhance relevance and accuracy in AI-generated summaries.

## Installation Guide

### System Requirements
- Node.js
- MongoDB Atlas
- Pinecone API keys
- Gemini AI and AssemblyAI API keys
- Ffmpeg (To convert video to audio)

### .env File
- `PORT`
- `MONGOURL`
- `JWT_SEC`
- `ASSEMBLYAI_API_KEY`
- `GEMINI_API_KEY`
- `PINECONE_API_KEY`
- `PINECONE_ENVIRONMENT`
- `PINECONE_INDEX_NAME`

## Design Decisions and Rationale

### 1. LLM and AI Selection
- **Gemini AI** was selected for its robust natural language processing capabilities, ideal for generating structured agendas and post-meeting summaries.
- **AssemblyAI** provides high-quality transcription services, enabling the tool to accurately track which discussion points were addressed during the meeting.

### 2. Retrieval-Augmented Generation (RAG)
In this tool, embeddings store relevant meeting data, which can be retrieved later using Pinecone. The Gemini AI embedding-001 model generates embeddings for meeting data, such as discussion points and summaries. These embeddings are stored in Pinecone to enable RAG, enhancing the relevance of generated content by pulling information from previous meetings.

### Embedding Generation Process
The function below generates embeddings for any given text input, like meeting summaries, to store and retrieve relevant data.

```javascript
async function generateEmbedding(text) {
  try {
    const model = genAI.getGenerativeModel({ model: "embedding-001" });
    const result = await model.embedContent(text);
    const embedding = result.embedding.values;
    return embedding;
  } catch (error) {
    console.error("Error generating embedding:", error);
    throw error;
  }
}
```

**How It Works**:
- The `generateEmbedding(text)` function takes a text input (e.g., meeting summary or discussion point).
- It then calls the Gemini AI model using the embedding-001 model type.
- The embedding is generated by invoking the model's `embedContent` method on the text input.
- The resulting embedding values are returned and can be stored in the Pinecone vector database for future retrieval.

These embeddings enable RAG within the tool, allowing retrieval of contextually relevant information from past meetings, making AI-generated outputs (like agendas and summaries) more accurate and informed.

### Embedding Usage in RAG
Once generated, embeddings are stored in Pinecone as vectorized representations of meeting content. During post-meeting summary generation, these vectors are queried, allowing the AI to access past meeting data and provide more relevant and accurate summaries and insights.

## Prompts Used

1. **To generate Agenda**:
    ```plaintext
    Given the following discussion points provided by participants, organize them into a coherent agenda. The agenda should group related topics together to streamline the meeting flow. Ensure that similar or related discussion points are linked and presented in a logical sequence. The final output should include:
    1. An organized agenda with topics grouped together.
    2. Clear headings for each section or group.
    3. Subtopics under each heading if necessary, to show how related points are connected.
    4. Do not include (**Time:**, **Location:**, **Attendees:**).
    Here are the discussion points:
    
${JSON.stringify(discussionPoints, null, 2)}
    ```

2. **To Track the Points**:
    ```plaintext
    Analyze the following discussion points and transcript:
    Discussion Points:
    ${discussionPoints.map((point, index) => `${index + 1}. ${point.points.join(", ")}`).join("
")}
    Transcript:
    ${transcript}
    Please provide the following analysis:
    1. For each discussion point, indicate whether it has been discussed in the transcript and do the strict checking if point is discussed or not.
    2. Flag any unresolved points or topics that need further discussion.
    3. Summarize the key decisions or action items from the discussion.
    Format your response as JSON with the following structure:
    {
      "analyzedPoints": [
        {
          "point": "Discussion point text",
          "discussed": true/false,
          "resolution": "Brief note on resolution or why it's unresolved"
        }
      ],
      "unresolvedTopics": ["List of unresolved topics"],
      "keySummary": "Brief summary of key decisions and action items"
    }
    Return only the JSON object without any additional text or formatting.
    ```

3. **To Generate Summary**:
    ```plaintext
    Based on the following transcript, generate a detailed summary of the meeting.
    The summary should include:
    * What was discussed.
    * Key decisions made during the meeting. If no key decisions were made, reply as "No specific decisions were made during this meeting."
    * Assigned action items and responsible participants. If no action assigned, reply as "No action items were assigned" and if no specific participants were mentioned, reply as "No specific participants were mentioned."
    * Do not include unnecessary fields at the top (Date of meeting, Attendees).
    * Heading must be based on summary.
    Transcript:
    ${transcript}
    Please provide a structured and detailed summary.
    ```

4. **To Implement RAG**:
    ```plaintext
    Based on the following meeting summary, answer this question: "${question}"
    Meeting Summary:
    ${relevantSummary}
    Please provide a concise and accurate answer based on the information in the summary.
    ```

## Future Enhancements
- **UI Improvements**: Enhance the user interface for better real-time tracking during meetings.
- **Extended AI Functionality**: Add more complex NLP features, such as topic modeling for improved agenda optimization.
- **Cross-Meeting Analytics**: Provide insights on recurring unresolved issues across multiple meetings.
