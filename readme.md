# Meeting Management tool

The Meeting Management tool is a solution designed to simplify the organization and documentation of meetings. By leveraging advanced generative AI, it ensures that every meeting is well-structured, productive, and thoroughly documented.

## Installation/Setup

Use the node package manager [npm](https://www.npmjs.com/) to Setup project.

run this command in dhiwisefrontend

```bash
npm install
```

run this command in dhiwisebackend

```bash
npm install
```

must install if it dose not downloaded by it self while running npm install [ffmpeg](https://www.ffmpeg.org/) and set path if required.

## Run the tool

As this tool is built by MERN stack will run frontend and backend individually

run this command in dhiwisefrontend

```bash
npm start
```

This will run the react files.

run this command in dhiwisebackend

```bash
nodemon server.js
```

or

```bash
node server.js
```

To run the backendpart

## Usage

There are 2 functions with ai integration using Gemini APi

1. To Genrate Agenda based on discussion points
2. To Genrate summary based on transcript
3. To track dissection Points based on transcript

There is a function with ai integration using AssemblyAi APi

1. To Genrate Transcript from the video

- Used Persistent Vector Database Usage To store summary And implemented RAG to search queries from vector database.

This tool have many non-AI facilities like to

- To Host and join meetings
- To upload videos and files for the meetings.
- Participants can add dissection Points to meeting
- Participants can download Documents and view the videos
