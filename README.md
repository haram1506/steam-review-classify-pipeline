## Project: Intelligent Steam Review Analyzer

### 1. Introduction

#### Task Description

Listening to player feedback is essential for game developers. When the game is still small, developers might be able to afford going through each review to find feedback and bugs that require fixing. As the game grows, however, this becomes impractical. Not to mention that players often write long reviews which have critical bug reports burried inside them. 

This project aims to automate this process of classifying user reviews for video games. The goal is to clsasify raw text reviews into multiple semantic categories: Bug Reports, Feature Requests, and Analysis. Since a review can fall under multiple categories, they are applied as 'tags' where a single review can have multiple.  

#### Motivation

As an indie game developer currently working on a game planned for steam release, managing community feedback is critical. However, I have noticed that players often like to bury feature requests and critical bug reports inside long, rambling reviews. An autoamated pipeline that is able to distinguish between a player stating ' This game is trash ' (Sentiment) and ' This game crashes when I do this '(Bug Report) would be a valuable tool for game devs.

#### Input and Output 
```
Input: Raw text string (" The game is great, but the sfx could be improved. ")

Output: A list of relavant tags ("Praise", "Bug report")

Success Criteria: Correct identifications of what constitutes a 'bug report' even if the users do not specify with the word 'bug'. Players might say 'not a bug' 
```
### 2. Dataset
#### Source and Collection

The current game I am developing is similar in style to the indie roguelike game *Duskers*, therefore, I created a custom dataset by manually collecting *Duskers* reviews from its steam page. This can serve as a proxy for the type of feedback I might expect to recieve in my own game. 

#### Examples 
24 reviews from the *Duskers* Steam Page.

#### Data split 
One small, customized dataset was used to maximize variety of test cases. 

#### Preprocessing: 
Each review was given tags based on its contents. All text was converted to lowercase, and non-text elements were removed. 

### 3. Methodology
#### Naive Baseline

The baseline approach uses a rule-based keyword search.

* How it works: It goes through the text and gives it tags based on specific trigger words (i.e. "crash" > "Bug Report", "wish" > "Feature Request")
* Why it is Naive: It classifies text strictly based on the trigger word, and does not care about context. It therefore cannot handle negation or any implied intentions. ("No bugs" > "Bug Report" even if it is not. "Fell through the floor" does not get caught as a "Bug Report" even if it is.)

#### AI Pipeline



