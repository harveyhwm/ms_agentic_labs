<img src="https://upload.wikimedia.org/wikipedia/commons/4/44/Microsoft_logo.svg" width="40" alt="Microsoft Logo">

# Lab 1: Prepare and Build a Fabric Data Agent

###
### **1. Preparation**

Before starting **Lab 1**, make sure you’ve completed the following setup steps:

1. ✅ **Have access to a Microsoft Fabric workspace with F2 Capacity or above**

That's it! The rest can be done as go along....

###
---

###
### **2. Next Steps**

Once you have an empty Lakehouse ready:

- Import this notebook into your Fabric workspace, or create a blank Notebook in your workspace and copy the code snippets below. Either approach is fine.

<br>
<img src="./images/import_notebook.png" alt="Product Table" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="70%"/>
<br>


- Either (a) Create a new Lakehouse

<br>
<img src="./images/new_lakehouse.png" alt="Product Table" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="70%"/>
<br>

**OR**
  
- (b) Attach an existing Lakehouse to the Notebook as a **data source**


<br>
<img src="./images/import_lakehouse.png" alt="Product Table" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="70%"/>
<br>


You should now see a Lakehouse attached to your notebook as follows:


<br>
<img src="./images/lakehouse_attached.png" alt="Product Table" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="70%"/>
<br>


This connection will allow you to prepare the Lakehouse with the data used throughout the lab exercises.

###
---

###
### **3. Populating the Lakehouse**

The code below will pull a set of parquet files into your Fabric workspace from an Azure file storage location, and will automatically transfer them into your default Lakehouse (attached to the Notebook) in the Tables/ directory. This essentially populates your Lakehouse with an active schema and gives us the data we need to start building our Fabric Data Agent!

Run this script (it will take no more than 5 minutes):

```python
import io
import requests
import numpy as np
import pandas as pd
import pyarrow.parquet as pq

table_names = ['inventory','product_categories','products','product_details', 'reviews','sales','users','warehouses']

for t in table_names:
    parquet_url = f'https://hm1cloudshell.blob.core.windows.net/fabrictest/__{t}.parquet?sv=2024-11-04&ss=bfqt&srt=sco&sp=rwdlacupiytfx&se=2026-06-23T13:43:15Z&st=2025-06-23T05:43:15Z&spr=https,http&sig=jizpj1SlKziJbvEGQsHX%2BMqUxGNx9dv%2BO36Efbx5Ie0%3D'
    response = requests.get(parquet_url)
    parquet_file = io.BytesIO(response.content)
    pandas_df = pd.read_parquet(parquet_file)
    spark_df = spark.createDataFrame(pandas_df)
    spark.sql(f'DROP TABLE IF EXISTS {t}')
    spark_df.write.mode('overwrite').saveAsTable(t)
```
###
----
###
### **4. Creating Your First Agent**

To create our first agent, we must goto our Workspace and Select "New".  On the way down, we will see an option to select "New Agent" (preview)


<br>
<img src="./images/create_agent_1.png" alt="Create Agent" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="80%"/>
<br>


Give it a sensible name, and save it. Now all we have to do it connect our Lakehouse, which is already primed with data. To do this, we can simply click New Data Source (+) and select the name of our Lakehouse.

<br>
<img src="./images/connect_lakehouse.png" alt="Connect Lakehouse" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="80%"/>
<br>

Once the Lakehouse has been added, we can choose which tables we want to work with, as shown below. We must then click "Publish" to save the changes.


<br>
<img src="./images/select_tables.png" alt="Select Tables" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="80%"/>
<br>

Congratulations! You are now ready to start asking real questions on your data 😎

###
---
###

### **5. Asking Questions**
<br>

#### **EASIER**

Let's start with some simpler questions.  By "simple" in this context, what we mean is questions that can easily be answered from a single table, or with a fairly simple join, without having to give the bot any specific instructions relating to your data.

Try asking the following questions to get a feel for the responses:
<br>

<span style="color:#1E90FF; font-size:17px;"><strong>QUESTION 01:</strong></span> **Can you give me the top performing product category by revenue this month?**

<span style="color:#1E90FF; font-size:17px;"><strong>QUESTION 02:</strong></span> **How does this compare to its performance each month for the last 12 months?**

<span style="color:#1E90FF; font-size:17px;"><strong>QUESTION 03:</strong></span> **What is the average rating of all our Video Game titles?**

<span style="color:#1E90FF; font-size:17px;"><strong>QUESTION 04:</strong></span> **How many video games are averaging below 3.0? What is this as a percentage of all Video Game titles?**

Out agent should have no trouble answering any of these. They exist in the data natively.
<br><br>

#### **MORE COMPLEX**

However, what happens if we ask it about something which requires less obvious table joins?

<span style="color:#1E90FF; font-size:17px;"><strong>QUESTION 05:</strong></span> **When was the PlayStation 5 first available, and what are we currently charging for it?**

This question causes the agent an issue, as it needs to be guided with instructions about our data.

<br>
<img src="./images/question_5_fail.png" alt="Question 5 Fail" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="60%"/>
<br>

So..... how can we solve this? 🤔

💡 Let's add the content of [**lakehouse_instructions_1.txt**](./assets/lakehouse_instructions_1.txt) into our AI Instructions, to educate the agent about our products and product_details tables, and the schema they contain, as they may not be aware of the necessary column, which in this case, is **date_first_available** (from product details) and **title** and **price** (from products).

💡 In addition, we'll also add the content of [**agent_instructions_1.txt**](./assets/agent_instructions_1.txt) into our Lakehouse Instructions, to guide the matching style towards wildcard-based matched and to attempt to match a wider array of columns.

<br>
<img src="./images/lab1_instructions.png" alt="Instructions" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="100%"/>
<br>

We can paste the text from our instructions into the agent's "Data Source Instructions" pane, as follows:

<br>
<img src="./images/lab1_instructions_2.png" alt="Instructions" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="65%"/>
<br>

💡 Let's also upload the example query in [**queries_1.json**](./assets/queries_1.json) to help guide the solution. The query supplied will, whilst naturally overfitting to the question at hand, help the bot to learn how to tacklee any question relating to products or release dates.

<br>
<img src="./images/lab1_queries_2.png" alt="Queries" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="30%"/>
<br>
<img src="./images/lab1_queries_1.png" alt="Queries" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="100%"/>
<br>


This query will guide the solution in the correct use of columns, case and wildcard structure for the question at hand.  Whilst it may appear to over-fit to the original question at first, we will also see that it can generalize to others.

Once we've input this text, we can then click just outside the box so that it saves our input, and we can then click **Publish** in the top menu, to save our changed to our published version.

<br>
<img src="./images/question_5_success.png" alt="Question 5 Success" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="65%"/>
<br>

.......the agent should now be able to answer - great job! 😄

<br>

Let's try a similar question:

<br>

<span style="color:#1E90FF; font-size:17px;"><strong>QUESTION 06:</strong></span> **What about the PlayStation 4?**

<br>
<img src="./images/question_6_success.png" alt="Question 6 Success" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="65%"/>
<br>

We are able to answer this immediately, despite it giving no information on its own, as it has used the chat history to gain context, and can benefit from the same instructions.

####

<span style="color:#1E90FF; font-size:17px;"><strong>QUESTION 07:</strong></span> **Which warehouses currently stock tennis balls?**


####

The agent initially fails this question, as it is only looking at product *categories*, not the *title*, which is in the main **products** table.


<br>
<img src="./images/question_7_fail.png" alt="Question 7 Fail" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="65%"/>
<br>

We can fix this by uploading more example queries, from the file [**queries_2.json**](./assets/queries_2.json).  These question-query pairs will tell the agent that searching specific products will require the title column, now just categories. Essentially, we will be casting a wider net.

Now let's ask it a different question, on a product not mentioned in the new queries uploaded:

####

<span style="color:#1E90FF; font-size:17px;"><strong>QUESTION 08:</strong></span> **Which warehouses currently stock yoga mats?**

We see that it handles this one like a pro!
<br>

<br>
<img src="./images/question_8_success.png" alt="Question 8 Success" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="65%"/>
<br>




#### **A GREAT BUSINESS QUESTION**
####


Let's now look at something deeply actionable which joins across multiple tables. We want to know about in-demand items that are out of stock.  The Fabric Data Agent can inform us, and can later be connected to other Microsoft AI Services to action on the results:

####

<span style="color:#1E90FF; font-size:17px;"><strong>QUESTION 09:</strong></span> **Which of the last month's most popular Electronics or Video Game products (by sales) are currently out of stock in the Seattle warehouse? Can you list the 20 most popular of these, with their subcategory, and sales and revenue in the last 30 days, as a table?**

<br>

<br>
<img src="./images/question_9_success.png" alt="Question 9 Success" style="border: 1px solid #AAA; border-radius: 4px; padding: 4px;" width="65%"/>
<br>

A closing thought.....what would you do with the information that you get back?



###
---
###

### **6. Wrapping Up**


Publish the agent in its current state. You will be able to return to it later for integration during ther subsequent labs.

While we’ve only added a small number of training examples, the results speak for themselves. The agent is already demonstrating solid reasoning and can handle surprisingly nuanced queries across multiple tables. That’s no small feat with this limited data.

Of course, building a production-grade agent will require a much larger set of examples—ideally hundreds or thousands—to capture the full range of how people ask questions and interact with structured data. But even at this early stage, the performance is promising and shows there’s real potential here. With more examples and refinement, this could become a powerful tool in the enterprise data stack.