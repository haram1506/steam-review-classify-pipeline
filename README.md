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

* How does it work? It goes through the text and gives it tags based on specific trigger words (i.e. "crash" becomes "Bug Report", "wish" becomes "Feature Request")
* How is it Naive? It classifies text strictly based on the trigger word, and does not care about context. It therefore cannot handle negation or any implied intentions. ("No bugs" becomes "Bug Report" even if it is not. "Fell through the floor" does not get caught as a "Bug Report" even if it is.)

#### AI Pipeline
In order to overcome the limitations with simply keyword matching, I implemented a Zero-Shot classification pipeline using the Hugging Face `transformers` library. This approach ensures that reviews can be classified with context in mind, going from simple pattern matching to a natural language inference task. 

1. Model Selection: I utilized `facebook/bart-large-mnli`, a BART model fine-tuned on the MNLI dataset. This model should be able to determine the logical relationships between two sentences, allowing it to classify text that it has never been trained on explicitly
2. Representation: The labels are converted into hypothesis sentences for the model so that they can have a better understanding of what this label should represent. (For example, "Feature Request (complaints about lack of content, suggestions, or missing mods)" becomes "Bug Report")
3. Post Processing: A probability score is given for each label independently. Steam reviews are often multi-dimensional, therefore, it makes sense to give multiple labels to a review then simply placing it under a single category. 
* Confidence threshold chosen: 0.3 
* This threshold was found after testing. Scores below this threshold lead to random, unrelated tags being placed, while higher thresholds resulted in too many reviews with no tags at all. 

#### Evaluation and Metrics
The AI pipeline was compared to the Naive baseline using precision, recall, and F1-Score. Since there can be multiple labels per review, the `sklearn.metrics` classification report with a weighted average was utilized to account for class imbalance. 

| Metric | Naive Baseline | AI Pipeline | Improvement |
| :--- | :--- | :--- | :--- |
| **Precision (Weighted)** | 0.69 | **0.81** | +12% |
| **Recall (Weighted)** | 0.54 | **0.58** | +4% |
| **F1-Score (Weighted)** | 0.59 | **0.67** | +8% |

The AI Pipeline shows a significant improvement in *precision* (0.81 vs. 0.69) which indicates taht when the AI applied a tag, it was much more likely to be correct compared to the Naive baseline, which frequently applied tags incorrectly due to the strict keyword triggers. The Recall improvement was minimal, mainly because the AI was conservative and sometimes returned no tags for reviews that tended to be more shorter or ambiguous. 

The following examples highlight where the AI Piepline's semantic and contextual understanding allowed it to outperform the strict logic of the baseline, but also its limitations. 

Case 1: Positive review with a word usually related to bugs. 
* Text: "Duskers is a game worth **breaking** your monitor for." 
* Baseline: "Bug" because of the word "break"
* AI Pipeline: "Praise"
* Discussion: The AI was able to properly understand that 

Case 2: Abstract Concepts 
* Text: "It's like they programmed **anxiety**"
* Baseline: Nothing because there were no keywords. 
* AI Pipeline: Nothing
* Discussion: Both models failed here. The true tag should be 'Praise' because this is a horror game, and 'anxiety' is a good thing. However, the AI likely gave 'anxiety' a low score for praise since in general training data, this is a negative emotion. 

### Reflection and Limitations 
* What worked better than expected: The Zero-Shot model's ability to handle idioms and sarcasm was impressive. Steam reviews are often filled with in-jokes and sarcasm, therefore being able to properly understand it is crucial. The 'monitor breaking' example validated the hypothesis that NL models can be much better at understanding context then a strict keyword search. Remember, the training did not have specific training on *Duskers* reviews, yet it was able to distinguish that "Monitor-breaking experience" is a positive thing.
* What failed or was harder than expected: The model struggled significantly in regards to making confident decisions. A major limitation was that a lot of reviews returned with empty brackets. This could not be easily fixed with a lower threshold, or even forcing at least one tag to be picked in the post processing, as that resulted in random tags. Additionally, a lot of game-specific terms such as "roomba" as a funny way of saying "drones" was likely not catched by the model. 
* Future Improvements: I would likely find a way to give context of the *game itself* to the model, so that it understands things like 'anxiety' and 'terrifying' is good, since this is a horror game. One way to do this is through Few-Shot Learning. Instead of just relying on label names, I would provide the prompt with 3-4 examples of labeled reviews. This can show explicitly that 'anxiety = praise' for this specific game. 



