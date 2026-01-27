I want to make something that uses sentiment analysis through LLMs. 

***

YouTube video/podcast sentiment analysis.

I'm think of running it on something like a Joe Rogan podcast or maybe a Mr. Beast video and comparing why a video did based on the sentiment or another thing in the video vs. 

- `pip install youtube-transcript-api` 
    - to get transcripts from YouTube.
- ChromaDB
    - https://www.geeksforgeeks.org/nlp/introduction-to-chromadb/
- https://github.com/clips/pattern

sentiment analysis of different types of YouTube videos.

[What Sentiment Analysis Is](https://www.geeksforgeeks.org/machine-learning/what-is-sentiment-analysis/)

I want to grab the YouTube video's transcript, then I want to analyze it to try and look for whether it's an overall positive or negative video

***

I don't think the speed dating dataset is going to work. It's not seeming easy to find transcripts of people just going on a speed date.

Now I need to find a dataset to run the sentiment analysis on. I'm thinking of trying to find the speed dating dataset from my TALK book.

[some data on github](https://github.com/datasets/speed-dating)
[example of someone analyzing a speed dating dataset using conventional methods](https://rstudio-pubs-static.s3.amazonaws.com/272379_8f1a6fc351bf4550825f611ddd9a331b.html)
***
Related tools:
- [[LangChain]]
- [[ChromaDB]]
- Ollama
- [Streamlit](https://streamlit.io/) (a good simple UI to use)

Example of this being implemented: https://github.com/solilei/PDF-RAG-System

Instead of Ollama, I wonder if the school provides access to an API so that I don't have to download any large models.

<<<<<<< HEAD
=======
- [ ] Do we have access to an LLM API through the school?
- [ ] Using Google Cloud credits?

>>>>>>> origin/main
