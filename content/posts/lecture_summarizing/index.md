+++
title = 'Summarizing University Lectures with OpenAI'
date = 2024-04-14T19:25:00-00:00
tags = ['data science', 'LLM', 'python', 'langchain', 'openai', 'whisper']
+++

# Introduction

Lectures suck.
You can always find better information for a topic these days, but what if you're actually taking a class for a grade?
You have to invest time into watching a teacher terribly explain something or briefly mention something that is actually rather important for you to study.

We can leverage OpenAI's technology to help us extract the information within any lecture.
This will allow anyone to venture out and learn through their preferred avenue of learning, while allowing the student to be aware of course specific information for a test.

Microsoft released a feature back in 2023 called [Intelligent Recap](https://techcommunity.microsoft.com/t5/microsoft-teams-blog/intelligent-meeting-recap-in-teams-premium-now-available/ba-p/3832541) which takes a **Microsoft Teams** meeting and summarizes it's contents. 
It leverages an speech-to-text and an LLM (probably OpenAI tools).
It would be interesting to stream a lecture through the tool to see how useful it is, given it's timestamp features. 
However, I (nor most students) don't have access to this tool, as it requires some sort of business account.

There are a couple of drawbacks to doing this with just a summary: no visual explanations and no timestamps to follow (was attempted but out of scope to do in a useful way).
In this article, I will explore the use of OpenAI's [ChatGPT](https://chat.openai.com/) and [Whisper](https://openai.com/research/whisper), along with [langchain](https://www.langchain.com/) in order to summarize a lecture.
Ways to fill in the gaps of visual aids that are specific to the class will be addressed in the conclusion.



# Procedure

The procedure for this experiment goes as follows:

1. Download a lecture's audio
2. Whisper Transcription
3. ChatGPT Summary

The three steps are simple in principle, but each provide their own set of challenges to overcome.


## 1. Download a lecture's audio

I used the MIT recorded lecture for Quantum Mechanics titled ["Introduction to Superposition"](https://www.youtube.com/watch?v=lZ3bPUKo5zc) by Professor Allan Adams.
The video was downloaded, then converted to audio using the snippet of code below.


```python
from moviepy.editor import VideoFileClip

def vid2aud(video_path, audio_path, overwrite=False):
    if not video_path.endswith(".mp4"):
        raise ValueError("Only mp4 files are supported")
    if not audio_path.endswith(".mp3"):
        raise ValueError("Only mp3 files are supported")
    if not os.path.isfile(video_path):
        raise FileNotFoundError("Video file not found")
    if os.path.isfile(audio_path) and not overwrite:
        raise FileExistsError("Audio file already exists")
    
    video = VideoFileClip(video_path)
    audio = video.audio
    # Set the desired bitrate (e.g., '64k')
    audio.write_audiofile(audio_path, bitrate='42k')  

video_path = "..\data\Lecture 1 Introduction to Superposition.mp4"
audio_path = "..\data\output_audio.mp3"
vid2aud(video_path, audio_path, overwrite=True)

```
    


## 2. Whisper Transcription

Details on interacting with OpenAI's API can be found [here](https://platform.openai.com/docs/api-reference).
This step is pretty cut and dry but is unfortunately not available through the langchain package for python.


```python
from dotenv import load_dotenv, find_dotenv
import os

load_dotenv(find_dotenv())


from openai import OpenAI
client = OpenAI()

audio_file= open(audio_path, "rb")
transcription = client.audio.transcriptions.create(
  model="whisper-1", 
  file=audio_file
)

transcription_file = "..\data\output_transcription.txt"

with open(transcription_file, 'w') as file:
    file.write(transcription.text)

```

Here's an example of the output:

> The following content is provided under a Creative Commons license. Your support will help MIT OpenCourseWare continue to offer high quality educational resources for free. To make a donation or to view additional materials from hundreds of MIT courses, visit MIT OpenCourseWare at ocw.mit.edu. Hi, everyone. Welcome to 8.04 for spring 2013. This is the fourth and presumably final time that I will be teaching this class, so I'm pretty excited about it. So my name is Alan Adams. I'll be lecturing the course. I'm an assistant professor in course eight. I study string theory and its applications to gravity, quantum gravity, and condensed matter physics. Quantum mechanics, this is a course in quantum mechanics. Quantum mechanics is my daily language. Quantum mechanics is my old friend. I met quantum mechanics 20 years ago. I just realized that last night. It was kind of depressing. So old friend. It's also my most powerful tool. So I'm pretty psyched about it. Our recitation instructors are Barton Zwiebach, and Matt Evans. Matt's new to the department, so welcome him. Hi. So he just started his faculty position, which is pretty awesome. And our TA is Paolo Glorioso. Paolo, are you here? Hey, there you are. OK, so he's a person to send all complaints to. So just out of curiosity, how many of you all are course eight? Oh, awesome. How many of you all are, I don't know, 18? Solid. Six?...


## 3. ChatGPT Summary

In the process of writing a good example for this step, I came across problems with simple prompts to ChatGPT.
The most important rule for a consistent output is a very specific request (content, formatting, etc.).
In the following blocks of code, I settled on this specific prompt to keep the article short and concise.
Even with a very specific request, you will see that content of the response is varied.
I will demonstrate this by prompting a couple of times with the same input and showing the difference in ChatGPT's outputs.


```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
```


```python
llm = ChatOpenAI(model_name="gpt-3.5-turbo")

output_parser = StrOutputParser()
prompt = ChatPromptTemplate.from_messages([
    ("system", """You are world class note taker and will 
summarize the following text into a numbered list of 
unique concepts, and their explanations as worded by 
the lecturer, for us to review later."""),
    ("user", "{input}")
])

chain = prompt | llm | output_parser

for n in range(2):
    output = chain.invoke({"input": transcription_text})
```

The raw form of the two responses are as follows:

#### Response 1

> 1. \*\*Introduction to Quantum Mechanics**: Quantum mechanics involves understanding the behavior of particles like electrons in a way that defies classical intuition.
> 2. \*\*Color and Hardness Boxes**: Experimental setup involving color and hardness boxes reveal that electrons can be in a superposition of states such as black or white, and hard or soft.
> 3. \*\*Results of Superposition Experiments**: Electrons in a superposition of states can behave unexpectedly, leading to phenomena like always coming out white in certain experiments.
> 4. \*\*Implications of Superposition**: The concept of superposition challenges classical notions of particles having fixed properties, leading to the need for new language and understanding in quantum mechanics.
    
#### Response 2   
    
> 1. Quantum Mechanics: Quantum mechanics is a framework that describes the behavior of particles at the atomic and subatomic levels. It includes principles like superposition, where particles can exist in multiple states simultaneously.
> 2. Superposition: An electron inside an apparatus can be in a superposition state, meaning it is neither definitively hard nor soft, but rather exists as a combination of both states. This concept challenges classical intuition and is a fundamental aspect of quantum mechanics.
> 3. Experimental Results: Experiments involving electrons passing through color and hardness boxes revealed that electrons exhibit behaviors that cannot be explained by classical physics. They can be in superposition states, leading to unpredictable outcomes that defy classical logic.
> 4. Implications of Superposition: Superposition implies that particles like electrons can exist in multiple states at the same time until measured, at which point they collapse into a single state. This idea forms the basis of quantum mechanics and challenges traditional notions of particle behavior.
> 5. Development of Intuition: Understanding and developing intuition for quantum phenomena, such as superposition, is a key aspect of learning quantum mechanics. It requires a shift from classical thinking to embrace the unique behaviors of particles at the quantum level.

    
    
    

This course is about quantum mechanics, yet there are some concepts in our output that pertain to unique concepts of the *classroom setting itself*. 
We shouldn't have any issue with this, as we can just pick the course specific material out ourselves.

Focusing on just the material pertaining to quantum mechanics, I'll mark off concepts that I know are in this lecture, and what these two outputs actually pick up.

| Concepts             | Response 1  | Response2
|---------             |------------|--------  
| Quantum Mechanics    | X          | X        
| Superposition        | X          | X        
| Uncertainty Principle|            |          
| Determinism          |            |          
| Locality             |            |          


For a long lecture, there are really only a few quantum mechanic specific concepts that are elaborated on and there may be some that I missed. 
Other definitions within this topic such as electrons, molecules, and detectors are assumed to be known anyways.
Also, their formatting is not consistent, which we will see later does come down to prompt engineering.
You can see both have picked up the same concepts but have missed these other definitions that are particularly important.
Resolving this could also come down to prompt engineering, but I propose something else that will give us more fine grained control over the process of parsing the knowledge in the lecture.

Let's revisit both the transcription step (step 2) and summarizing (step 3).

## 2. (improved) Whisper Transcription

Here we will leverage another feature of Whisper. Time Stamps.
Segments are delimited by each sentence (I believe).
Now we can simply summarize groups of sentences to give us more fine grained information.




```python
from openai import OpenAI
client = OpenAI()

audio_path = "..\data\output_audio.mp3"

audio_file= open(audio_path, "rb")
transcription = client.audio.transcriptions.create(
  model="whisper-1", 
  response_format="verbose_json",
  timestamp_granularities=["segment"],
  file=audio_file
)
print(transcription.text)


transcription_file = "..\data\output_transcription.json"

import json
with open(transcription_file, 'w') as file:
    json.dump(transcription.segments, file)


```

    

## 3. (improved) ChatGPT Summary


I will invoke the following strategy. 

1. Reformat the transcription to reduce token usage but keep timestamp information.
2. Break the video up into approximately 10 separate chunks.
3. Ask ChatGPT to summarize each chunk into valuable notes.
4. Send ChatGPT it's collective summarize and ask it to combine them.

Step 3 gives increased granularity and visibility into our process of breaking up our information. 
The model missing two important concepts in our prompts (locality and uncertainty principle) could be due to the fact that they're simply not mentioned as much 
as the terms **superposition** or **quantum mechanics**.
This appears necessary given the model's inconsistency in dealing with all of the text at one time. 
We can also break this up further to have some sort of note taken transcript that more closely follows how we would take notes on our own.
In general, grabbing a full summary of a lecture is how we end up remembering the information in our minds.
However, the visual portion of this video is still important (equations, graphs, demonstrations, etc.) to solidify those memories more easily.
This won't be fixed in this video but will be addressed later in the article's conclusion.



```python

# Load the transcription segments from the file

transcription_file = "..\data\output_transcription.json"

with open(transcription_file, 'r') as file:
    transcription_segments = json.load(file)

```


```python
from pprint import pprint 

time_stamped_transcription = ""
for segment in transcription_segments:
    hour = int(segment['start']//3600)
    minute = int((segment['start']%3600)//60)
    seconds = int(segment['start']%60)
    time_stamped_transcription += f"[{hour}:{minute}:{seconds}] {segment['text']} \n"
    
pprint(time_stamped_transcription[0:1000])
```

    ('[0:0:0]  The following content is provided under a Creative \n'
     '[0:0:2]  Commons license. \n'
     '[0:0:3]  Your support will help MIT OpenCourseWare \n'
     '[0:0:6]  continue to offer high quality educational resources for free. \n'
     '[0:0:10]  To make a donation or to view additional materials \n'
     '[0:0:12]  from hundreds of MIT courses, visit MIT OpenCourseWare \n'
     '[0:0:16]  at ocw.mit.edu. \n'
     '[0:0:21]  Hi, everyone. \n'
     '[0:0:24]  Welcome to 8.04 for spring 2013. \n'
     '[0:0:27]  This is the fourth and presumably final time \n'
     "[0:0:30]  that I will be teaching this class, so I'm pretty excited about "
     'it. \n'
     '[0:0:33]  So my name is Alan Adams. \n'
     "[0:0:34]  I'll be lecturing the course. \n"
     "[0:0:37]  I'm an assistant professor in course eight. \n"
     '[0:0:40]  I study string theory and its applications \n'
     '[0:0:43]  to gravity, quantum gravity, and condensed matter physics. \n'
     '[0:0:48]  Quantum mechanics, this is a course in quantum mechanics. \n'
     '[0:0:52]  Quantum mechanics is my daily language. \n'
     '[0:0:54]  Quantum mechanics is my old friend. \n'
     ...

```python
import math

llm = ChatOpenAI(model_name="gpt-3.5-turbo")

# Calculate the number of segments in each 5 minute group
num_segments = len(transcription.segments)
segments_per_group = math.ceil(num_segments/20)

# Create a list to store the grouped segments
segment_groups = []

# Iterate over the segments and group them
for i in range(0, num_segments, segments_per_group):
    group = transcription.segments[i:i+segments_per_group]
    segment_groups.append(group)
```


```python

output_parser = StrOutputParser()
prompt = ChatPromptTemplate.from_messages([
    ("system", """
     You are world class note taker and will summarize the following transcript from a lecture. 
     Summarize into as many key points as possible and definitions.
     Format it as such.
     Key points:
     [key points]
     
     Definitions:
     [definitions]
     """),
    ("user", "{input}")
])

chain = prompt | llm | output_parser


output_summaries = []
for group in segment_groups:
    output = chain.invoke({"input": group})
    output_summaries.append(output)
```

Here's an example of a chunk's summary:

> Key points:
> - Physicists have a term for a new mode of being called "superposition."
> - Superposition refers to an object being in a state of two or more contradictory properties simultaneously.
> - Electrons can exist in a superposition of being hard and soft.
> - Quantum mechanics provides a language to understand superposition in the realm of atoms and molecules.
> - Developing an intuition for superposition is essential in quantum mechanics.
> - Electrons exhibit behavior in superposition, which may defy traditional intuition.
> - The course aims to expand understanding beyond daily experiences to grasp the concept of superposition.
> 
> Definitions:
> - Superposition: The state in which an object simultaneously exists in multiple contradictory states or properties.
> - Quantum mechanics: The branch of physics that deals with the behavior of particles at the atomic and subatomic levels.
    

Now I'll compile all of these notes into a giant query and send it for a meta-summarization.


```python
# combine the text summaries into a single string
text_summary = ""
for summary in output_summaries:
    text_summary += summary + "\n\n"

    
# create a meta chain
meta_prompt = ChatPromptTemplate.from_messages([
    ("system", """
    Compile and organize the following notes together into two categories: key points and definitions.
     """),
    ("user", "{input}")
])


meta_chain = meta_prompt | llm | output_parser

note_output = meta_chain.invoke({"input": text_summary})
```

And our final summarized output:

>### Key Points:
>- The lecture focuses on teaching quantum mechanics with an emphasis on developing intuition rather than just calculations.
>- Active participation, problem-solving, and collaboration with other students are encouraged to enhance understanding.
>- Course materials are available on the MIT OpenCourseWare website.
>- Clickers are used for non-graded activities like concept questions and quizzes.
>- Two midterms and one final exam are part of the assessment.
>- Recommended textbooks are suggested for further reading.
>- Asking questions, attending office hours, and recitations are advised for better understanding.
>- The lowest problem set score will be dropped without questions.
>- Importance of group work to efficiently cover all recommended textbooks.
>- The unpredictability and randomness in quantum mechanics are highlighted.
>- The limitations of building specific boxes due to fundamental principles are discussed.
>- The uncertainty principle in physics is introduced.
>- Experiments are conducted to observe the behavior of electrons and photons.
>- The concept of superposition in quantum mechanics is explained.
>
>### Definitions:
>- **Quantum mechanics:** A branch of physics that deals with the behavior of particles at the atomic and subatomic levels.
>- **MIT OpenCourseWare:** An initiative by MIT to provide free access to course materials from MIT classes.
>- **Problem sets:** Assignments given to students to solve and practice the concepts taught in class.
>- **Clickers:** Electronic devices used for interactive participation and feedback in lectures.
>- **Midterms:** Examinations held in the middle of the semester to assess students' understanding of the course material.
>- **Final exam:** Comprehensive exam held at the end of the semester to evaluate students' overall knowledge of the course.
>- **Collaboration:** Working together with other students to achieve common goals.
>- **Textbooks:** Academic books containing information and explanations on a particular subject.
>- **Office hours:** Designated times outside of class where students can meet with the professor for additional help or clarification.
>- **Recitations:** Small group sessions led by teaching assistants to review course material and work on problem-solving.
>- **Lecture series:** A series of lectures on a particular topic or subject matter.
>- **Reading assignments:** Required readings for a course to supplement the material covered in lectures.
>- **Color boxes:** Containers where electrons are sent in and may come out either white or black.
>- **Hardness boxes:** Containers where electrons are sent in and may come out either soft or hard.
>- **Correlation:** A relationship or connection between two or more variables, indicating they may influence each other.
>- **Probability:** The likelihood of a specific outcome occurring in an event or experiment.
>- **Determinism:** The philosophical concept that all events are determined by external causes.
>- **Compression ratio:** The level of compression applied to speech data.
>- **No speech probability:** The likelihood of non-speech elements in the audio.
>- **Aperture:** An opening or hole through which particles can pass.
>- **Photons:** Particles of light that carry electromagnetic force.
>- **Laser:** Device that emits light through optical amplification.
>- **Apparatus:** Equipment or tools used for a specific purpose.
>- **Empirical:** Based on observation or experience rather than theory.
>- **Superposition:** The state in which an object simultaneously exists in multiple contradictory states or properties.
>
>By compiling and categorizing the content, the key points and definitions are presented in a structured and organized manner for better understanding.
    

This is an improvement in the shear amount of information created.
And although we did pick up definitions on the **uncertainty principle** (found in key concepts not definitions) and **determinism**,
we did not find **locality**.
Again, this can come down to prompt engineering or how well we break up the incoming information.
From this point of view I have a "good enough" solution that at least mentions 80% of the new class specific definitions (locality is mentioned once with a vague definition, it could come up in future lectures with more emphasis).
The video is still important to watch, but if we wanted to follow the class and say, go to third party material for those specific definitions, this is a great way to compile a list of where to start.


# Conclusion

I generally try to produce projects that I would use in the future, but the amount of work required to make this particularly useful to me requires more experience in other domains and a lot more time.
A good application to this process would be some sort of web plugin or video player plugin, that pre-generates the notes and gives us real time context and maybe definitions as we go.

If you pay attention closely to the original lecture, many of the definitions provided by our final prompt were not explicitly stated by the lecturer.
There are pros and cons to this approach. On one hand, we get very basic definitions that are assumed to be known, such as the definition of "trivial" or "probability". 
This is great because in practice, not everyone comes in with all of the basics they need to succeed. 
This is also bad because part of the learning process is developing a vocabulary for the subject matter by reading context clues surrounding the jargon.
Too much learning could be potentially taken away from the student.
Another con is that I am not absolutely certain that the definitions in the summary ARE the definitions defined within the lecture.
ChatGPT may be using it's own definition.
For simple subjects, this is probably ok, but for more complex topics, ChatGPT may get some nuances wrong.

This brings me to my ideal use case for this type of note summarization.
It would be either an application itself, or a plugin to a popular video player such as VLC.
The application would alter the lecture process as such.

1. A video player that contains some sort of interaction box below it for students to follow along with.
2. When something not basic is said, such as a definition that is not immediately defined, it will appear either with the definitions or for the student to click on.
3. Periodically throughout the video, your definition and problem solving ability will be tested with a brief quiz (creating the quiz could be an application of an LLM that is curated by a human).
4. For classes that you are attending, relevant information such as test and quiz dates could be parsed and handled, and available to access at any time.
5. Going even further, maybe there is some sort of account management where you can share notes with other students, corrections to syllabus details, etc.

Lectures, in my eyes, are one of the lowest forms of learning. They require us to be active listeners to gain anything of value, but do not coerce us to do anything beyond simply tagging along.
Adding periodic check-ins and real-time interactions would make it easier to stay engaged and pay attention.
All these little moments of learning add up and can prevent students from cramming, which is well known for producing high amounts of stress, and poor short/long term results.
LLM's have great potential but require much creativity to help us learn about different subjects and improve our understanding in a classroom setting.

