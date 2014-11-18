---
---

Data Science in Industry
========================

Alon Kaufman PhD.

##RSA & EMC Data Science Activities

* over 8000 customers, 7y building proven ROI products based on ML
* In israel, team of 25 data scientists, 8PhD's focusing on:
  * Financial fraud detection (nline banking & ecommersce)
  * Cybercrime, security risk analysis
  * IT advanced solutions
  * General data science services for companies entering the field
* End to end business solutions

## What we know about big data
 - 4 'v's: variety, volume, velocity, value
 - It's **BIG**
  - 2.5 Petabytes generated daily
 - It's a buzz word (in 2012 anyway)
 - it's messy
 - It's unused
  - Only 23% estimated as valuable
  - but only 3% is actually tagged and .5% analyzed
 - but people say it's useful (See gartner magnitude index -- [here?](http://www.collaborative.com/blog/useful-models-for-indexing-big-datas-impact/))
 - How does it all make sense? *analyze it*
 
   Enaling big data is becoming central to enterprise strategy  
   Every member of the C-suite has an interest in a company's big data strategy.  
   Most companies have an opportunity to utilize big data to improve their strategy.
####Key Verticals -- Industries embracing Big Data
- retail
- financial services
  * fraud detection
- manufacturing
  - anomaly detection
- government
- Advertising and PR
- Media & Telecom
- Energy
- Healthcare & Life Sciences


##What are we going to cover here?

- Today: Introduction to data science, the business problems 
- January: 7 Use case scenarios
  - Real world business problems & data science solutions
  - Hands on
  
# Getting Started

## Making Sense out of data is not always Trivial

Example: Warplanes with a low survival rate. Some come back riddled with bullets, others don't come back
Where do you reinforce them?
**Answer:** Where there aren't bullet holes -- those are the fatal points on the plane when shot.

###Big Data In Action

Make better Decisions using Big Data

- Acquire, Organize, Decide, Analyze
  - Acquire all available data
  - Organize and distill big data using massie parallelism
  - Analyze all your data at once
  - Decide based on real-time Big Data
- We'll focus on Decision and Analysis


   Video example -- [MR. Kelly orders a pizza](https://www.aclu.org/ordering-pizza)
   
### The birth of a new role: Data Scientist

[Data science definition](http://en.wikipedia.org/wiki/Data_science)  
Gartner (March 2012)
 - Data Management
   - Understand data
 - Analytics modeling
   - Know analytics
 - Business analysis
   - Focus on the business
     -Goals
     - Constraints
     - Decisions
     - Communication of results

Data scientist categories
- Programming skills
- Technical platform knowledge
- Communication skills
- Domain knowledge
- Mathematical/Statistical skills

EMC definition of big data: the art of using (big) data to solve business problems.  
Site: [KDNuggets](http://www.kdnuggets.com/)
[languages](http://www.kdnuggets.com/polls/2013/languages-analytics-data-mining-data-science.html)  

####Why is Data Science different from "Old" statistics/data mining/machine learning?

- Type of Data
  - Before: structured
  - Today: Structured& unstructured
-Availability
  - Before: No data available, every research new collection
  - Today: Data already collected; available for analysis
- Volume
  - Before: Existing tools were able to process small samples
  - Today Good processing capabilities for terabytes of data
- Productization
  - Before: Analysis mainly for research purposes
  - Today: Ongoing decision making  based on analytics components
  
## _10 minute break_


### _Aaaaand we're back_

## The data Science Cycle

###Business Understanding
- Learn the subject, and drill down, to create:
  - Statement of *Business Objective/Problem*
  - Statement of *Data Mining Objective*
  - Statement of *Success Criteria*

Focus on understanding the project objectives/requirements from a business perspective and convert this knowledge to a data science problem.

#### Telco Churn example
- Churn is when a customer leaves a company in some way
  - Business problem: Impacts bottom line
  - Data Science problem: ???
- What business problems would lead to churn being an issue?
  - High customer acquisition costs
  - A goal of higher market share
  - long-term customers are more predictable and allow network optimization
  - Decrease expenditure on help desk
  - Decrease exposure to fraud and bad debt
  - Higher confidence of investors
  - Increased added-value sales to long-term customers
- In which of these cases would a model of churn be helpful?  
  how do they impact the model
-What is churn?
  - Absolute?
  - Per customer?
  - Per line?
  - Per service?
  - What if a customer leaves the country (the planet?)?
  - What if the customer required technology we can't provide?
  - Are we interested in a particular customer segment?

##### Business Understanding - Telco Churn
- Statement of business objective
  - Increase market share -- since recruiting new customers is more expensive than preserving customers, prevent churn by identifying ahead of time profitable customers that are about to leave and retain them
- Statement of DS Objective
  - For permanent customers predict abandonment 15 days before customer decision (Absolute Churn)
  - Estimate customer life-time-value
  
- Statement of Success Criteria
  - Missed it :frown:
  
##### Business Understanding - Common Mistakes
- Avoid technical descriptions - it's all about understanding the business problem
- Avoid solving lovely problems that don't help the business
- Start top-down, business problem usually are high level:
  - Profitability, Market Share, ROI, Renewals, Revenues usw...
- Have a clear & agreed measure for success!
- Remember - Data Science is yet another tool, not a solution for everything!

##### Data Understanding

What types of information do you want to know to predict churn?
- visits to competitor websites
- Usage statistics
- Spending
- Customer age (age of account)
- social group
- Personal data (age, demographic, etc.)

#####Data, Data, Data is KING!
- Data is typically a record on a customer/patient/product/entity
- For example, in retail a record may include:
  - Customer data (gender, age, location)
  - Behavioral data (last purchases, seasonal...)
  - External knowledge (weather, competition, state of the organization...)
- Data is mostly historical
  - Identify trends in the past and predict them in the future
- Other data available
  - Text video, images, audio
  - Web, social media
  
##### How to select appropriate data
incoming gorilla example -- actually it was a moonwalking bear but the contrast was too tough to see. re-watch it [here](https://www.youtube.com/watch?v=Ahg6qcgoay4)  

- Ideally data will be stored in a data warehouse - clean, accurate, and available
- Write down he wish list of data
  - Domain expert in light  of the business problem
  - What is available, what can we acquire?
  - Combine data repositories
- HOw much data do we need? ... Depends
  - more is usually better
    - seasonality is key. Probably want at least one year
  - How much history?
  - How many and which variables?
- Sample... at the end of the day the data MUST contain all possible outcomes
- Get to know your data!!!

##### Why is the data and Data Understanding so Important?

- DS needs data!
- GIGO: Garbage In Garbage Out
- Caution: Data can be misleading

#### Data Preparation

- Takes usually _over 90% of the time_
  - Collection
  - Data selection
  - ??? (Missed it)
  
##### Simple example why prepare data
- Data in the real world is dirty
  - incomplete (eg: occupation="") [:bulb:](http://thecodelesscode.com/case/31)
  - old     (eg: age)
  - noisy
  - inconsistent ([canadian zipcodes](http://en.wikipedia.org/wiki/Postal_codes_in_Canada))
  - redundant

"No business will say 'take all this money and go improve your data'."

##### Creating a Model Set
- Assembling customer signatures
  - Creating single united signatures
-Create a balanced sample
  - Balance different groups
- Create a model set and partitioning
  - Training, validation, and testing - Three different data sets to prevent over-fitting

#### Model
- The model is what provides the transformation from data to actionable knowledge
- Selecting of the modelling techniques is based upon the data science objective
- Modelling is an iterative process

##### Select and Build Models
- What type of data science modeling task do we need?
  - _**Directed** models_: We know in advance which analysis problem we want to solve - _**has a target variable**_
  - Undirected
  - Descriptive
  - Supervised learning
  - Unsupervised learning
  
#####Two main learning paradigms
- Supervised
  -Classification
  -estimation
- Unsupervised
  - Clustering
  - Association rules
- Both
  - Anomaly detection
  - Collaborative filtering
##### Data mining Tasks: Classification
- Predicts categorical class labels, very human imperative
- Example:
  - Classify credit applicants to low/mid/high risk
  - Place a new customer in a specific product program
  
##### Data Mining Tasks: Estimation
- Similar to classification but continuous outcome. Estimate valuses of variable from evolution of other continuous or nominal values

##### Data Mining Task: Clustering
- Clustering/segmentation: Clustering is the detection of groups of individuals. no predefined categories.
- Example
  - Find types (clusters) of telephone calls or cc purchases
  - Market segmentation

##### Data Mining Tasks: Association Rules
- Methods to study attributes or characteristics that "go together"- market based analysis
- An association between two attributes happens when the frequency of two specific values happen together is relatively high
Example:
  - What products are purchased together
  - Beer and diaper case (huh?)
  - etc. (missed)

##### Data Mining Task: Collaborative Filtering
- Method or making automatic predictions (filtering) about interests of users or entities

~~Short Break with funny formatting~~

####Evaluation

- How accurate is the model
- compared to na:ive models
- how much ocnfidence can we phave in the model's predictions?
- How understandable is the outcome
- HOw comprehensible is the model -- this sounds more like resiliency (thought he meant comprehensive); also addresses how the output is conveyed/supported when presented to the client.



#####Classification Evaluation: Accuracy Measures
- **TP** - True Positive (Classified Positive, Reality Positive)
- **TN** - True Negative (Classified Negative, Reality Negative)
- **FP** - False Positive (Classified Positive, Reality Negative)
- **FN** - False Negative (Classified Negative, Reality Positive)
- Total Accuracy = 
- Sensitivity, recall - Probability of correctly classifying a positive event ("Hit Rate") 
- Specificity - probability of correctly classifying a negative event
- Precision - Percentage of positive predictions that are correct
- False alarm rate (false positive rate) = 
- F-measure : 2*(recall*precision)/(recall+precision)
  **_Get the definitions and equations from the slides_**
  
##### Evaluation: Lift Chart
- **Lift** is a measure of the effectiveness of a predictive model calculated as the ratio between the results obtained with and without predictive model
- Customer base: 10m, Random 1% churn = 100K
- ideally a model should be well above the x=y line (random line)
- lift = model response minus random line for some x% of customer base/population

##### Deployment
- Moving from data science space to the actual application
- Determine how the results need to be utilized
- Who needs to use them
- Continuous data preprocessing
- Scalability
- Ongoing Tuning

##### Why is model evaluation not sufficient
- Not every successful model leads to good results
  -fixed cost of the campaign and model
  - cost of making an offer
  - cost of fulfilment
