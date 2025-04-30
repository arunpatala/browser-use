# Example Use Cases for Browser-Use

This document provides detailed examples of how to use Browser-Use to automate browser tasks. Each example outlines the actions taken and the classes/functions called in the codebase.

---

## 1. Shopping Automation

**Task:** Add grocery items to cart and checkout.

**Actions and Code Calls:**

1. **Initialize the Agent:**
   - Create an `Agent` instance with a task and an LLM (e.g., `ChatOpenAI`).
   - **Code:**  
     ```python
     from langchain_openai import ChatOpenAI
     from browser_use import Agent
     import asyncio
     from dotenv import load_dotenv
     load_dotenv()

     async def main():
         agent = Agent(
             task="Add grocery items to cart and checkout",
             llm=ChatOpenAI(model="gpt-4o"),
         )
         await agent.run()

     asyncio.run(main())
     ```

2. **Navigate to the Shopping Website:**
   - The agent uses the `go_to_url` action to navigate to the shopping site.
   - **Code:**  
     ```python
     # Inside the agent's step loop, the LLM decides to call:
     # Controller.act(action=GoToUrlAction(url="https://example-shopping-site.com"))
     ```

3. **Search for Items:**
   - The agent uses the `search_google` action to find grocery items.
   - **Code:**  
     ```python
     # Controller.act(action=SearchGoogleAction(query="organic apples"))
     ```

4. **Add Items to Cart:**
   - The agent clicks on the "Add to Cart" button for each item.
   - **Code:**  
     ```python
     # Controller.act(action=ClickElementAction(index=1))  # Assuming index 1 is the "Add to Cart" button
     ```

5. **Proceed to Checkout:**
   - The agent clicks the "Checkout" button.
   - **Code:**  
     ```python
     # Controller.act(action=ClickElementAction(index=2))  # Assuming index 2 is the "Checkout" button
     ```

6. **Complete the Purchase:**
   - The agent fills in shipping details and confirms the purchase.
   - **Code:**  
     ```python
     # Controller.act(action=InputTextAction(index=3, text="John Doe"))
     # Controller.act(action=InputTextAction(index=4, text="123 Main St"))
     # Controller.act(action=ClickElementAction(index=5))  # Confirm purchase
     ```

---

## 2. LinkedIn to Salesforce Integration

**Task:** Add the latest LinkedIn follower to leads in Salesforce.

**Actions and Code Calls:**

1. **Initialize the Agent:**
   - Create an `Agent` instance with the task and an LLM.
   - **Code:**  
     ```python
     agent = Agent(
         task="Add my latest LinkedIn follower to my leads in Salesforce",
         llm=ChatOpenAI(model="gpt-4o"),
     )
     ```

2. **Navigate to LinkedIn:**
   - The agent uses the `go_to_url` action to navigate to LinkedIn.
   - **Code:**  
     ```python
     # Controller.act(action=GoToUrlAction(url="https://www.linkedin.com"))
     ```

3. **Log in to LinkedIn:**
   - The agent inputs credentials and clicks the login button.
   - **Code:**  
     ```python
     # Controller.act(action=InputTextAction(index=1, text="username"))
     # Controller.act(action=InputTextAction(index=2, text="password"))
     # Controller.act(action=ClickElementAction(index=3))  # Login button
     ```

4. **Navigate to Followers:**
   - The agent navigates to the followers page.
   - **Code:**  
     ```python
     # Controller.act(action=ClickElementAction(index=4))  # Navigate to followers
     ```

5. **Extract Follower Information:**
   - The agent uses the `extract_content` action to retrieve the latest follower's details.
   - **Code:**  
     ```python
     # Controller.act(action=extract_content(goal="Get the latest follower's name and email", should_strip_link_urls=True))
     ```

6. **Navigate to Salesforce:**
   - The agent uses the `go_to_url` action to navigate to Salesforce.
   - **Code:**  
     ```python
     # Controller.act(action=GoToUrlAction(url="https://www.salesforce.com"))
     ```

7. **Log in to Salesforce:**
   - The agent inputs credentials and clicks the login button.
   - **Code:**  
     ```python
     # Controller.act(action=InputTextAction(index=5, text="salesforce_username"))
     # Controller.act(action=InputTextAction(index=6, text="salesforce_password"))
     # Controller.act(action=ClickElementAction(index=7))  # Login button
     ```

8. **Add Follower as a Lead:**
   - The agent navigates to the leads section and adds the follower's information.
   - **Code:**  
     ```python
     # Controller.act(action=ClickElementAction(index=8))  # Navigate to leads
     # Controller.act(action=InputTextAction(index=9, text="Follower Name"))
     # Controller.act(action=InputTextAction(index=10, text="follower@example.com"))
     # Controller.act(action=ClickElementAction(index=11))  # Save lead
     ```

---

## 3. Job Application Automation

**Task:** Read a CV, find ML jobs, save them to a file, and apply for them in new tabs.

**Actions and Code Calls:**

1. **Initialize the Agent:**
   - Create an `Agent` instance with the task and an LLM.
   - **Code:**  
     ```python
     agent = Agent(
         task="Read my CV & find ML jobs, save them to a file, and then start applying for them in new tabs",
         llm=ChatOpenAI(model="gpt-4o"),
     )
     ```

2. **Read the CV:**
   - The agent uses the `extract_content` action to read the CV file.
   - **Code:**  
     ```python
     # Controller.act(action=extract_content(goal="Read the CV file", should_strip_link_urls=False))
     ```

3. **Search for ML Jobs:**
   - The agent uses the `search_google` action to find ML job listings.
   - **Code:**  
     ```python
     # Controller.act(action=SearchGoogleAction(query="machine learning jobs"))
     ```

4. **Save Job Listings to a File:**
   - The agent extracts job listings and saves them to a file.
   - **Code:**  
     ```python
     # Controller.act(action=extract_content(goal="Extract job listings", should_strip_link_urls=True))
     # Controller.act(action=save_to_file(filepath="ml_jobs.txt"))
     ```

5. **Apply for Jobs:**
   - The agent opens each job listing in a new tab and applies.
   - **Code:**  
     ```python
     # Controller.act(action=OpenTabAction(url="https://example-job-site.com/job1"))
     # Controller.act(action=InputTextAction(index=1, text="Cover Letter"))
     # Controller.act(action=ClickElementAction(index=2))  # Submit application
     ```

---

## 4. Google Docs Letter Automation

**Task:** Write a letter in Google Docs to a family member, thanking them for everything, and save the document as a PDF.

**Actions and Code Calls:**

1. **Initialize the Agent:**
   - Create an `Agent` instance with the task and an LLM.
   - **Code:**  
     ```python
     agent = Agent(
         task="Write a letter in Google Docs to my Papa, thanking him for everything, and save the document as a PDF",
         llm=ChatOpenAI(model="gpt-4o"),
     )
     ```

2. **Navigate to Google Docs:**
   - The agent uses the `go_to_url` action to navigate to Google Docs.
   - **Code:**  
     ```python
     # Controller.act(action=GoToUrlAction(url="https://docs.google.com"))
     ```

3. **Log in to Google:**
   - The agent inputs credentials and clicks the login button.
   - **Code:**  
     ```python
     # Controller.act(action=InputTextAction(index=1, text="google_username"))
     # Controller.act(action=InputTextAction(index=2, text="google_password"))
     # Controller.act(action=ClickElementAction(index=3))  # Login button
     ```

4. **Create a New Document:**
   - The agent clicks the "New Document" button.
   - **Code:**  
     ```python
     # Controller.act(action=ClickElementAction(index=4))  # New document button
     ```

5. **Write the Letter:**
   - The agent inputs the letter content.
   - **Code:**  
     ```python
     # Controller.act(action=InputTextAction(index=5, text="Dear Papa, Thank you for everything..."))
     ```

6. **Save as PDF:**
   - The agent uses the `save_pdf` action to save the document as a PDF.
   - **Code:**  
     ```python
     # Controller.act(action=save_pdf())
     ```

---

## 5. Hugging Face Model Search

**Task:** Look up models with a license of `cc-by-sa-4.0` and sort by most likes on Hugging Face, then save the top 5 to a file.

**Actions and Code Calls:**

1. **Initialize the Agent:**
   - Create an `Agent` instance with the task and an LLM.
   - **Code:**  
     ```python
     agent = Agent(
         task="Look up models with a license of cc-by-sa-4.0 and sort by most likes on Hugging face, save top 5 to file",
         llm=ChatOpenAI(model="gpt-4o"),
     )
     ```

2. **Navigate to Hugging Face:**
   - The agent uses the `go_to_url` action to navigate to Hugging Face.
   - **Code:**  
     ```python
     # Controller.act(action=GoToUrlAction(url="https://huggingface.co"))
     ```

3. **Search for Models:**
   - The agent uses the `search_google` action to find models with the specified license.
   - **Code:**  
     ```python
     # Controller.act(action=SearchGoogleAction(query="cc-by-sa-4.0 models on Hugging Face"))
     ```

4. **Extract Model Information:**
   - The agent uses the `extract_content` action to retrieve model details.
   - **Code:**  
     ```python
     # Controller.act(action=extract_content(goal="Extract model names and likes", should_strip_link_urls=True))
     ```

5. **Save Top 5 Models to a File:**
   - The agent saves the extracted information to a file.
   - **Code:**  
     ```python
     # Controller.act(action=save_to_file(filepath="top_models.txt"))
     ```

---

## Summary

These examples demonstrate how Browser-Use can be used to automate various browser tasks, from shopping and job applications to document creation and data extraction. Each example outlines the actions taken and the corresponding classes and functions called, providing a clear guide for users and developers. 