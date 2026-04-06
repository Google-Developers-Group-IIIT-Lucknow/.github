# GDG Devsphere

Welcome to **Devsphere** — a fast-paced, high-impact coding battleground where only your skills matter the most.

## Event Overview

**GDG Devsphere** is a **3-hour individual participation event** designed to challenge your development skills across multiple domains.

You’ll be given a curated sheet of **9 repositories**:

* **App Development** (3)
* **Web Development** (3)
* **FOSS (Open Source)** (3)

Each category includes:

* 🟢 Easy
* 🟡 Medium
* 🔴 Hard

Every difficulty carries **different marks**, so solve wisely and maximize your score!

***Keep in mind, there is no step marking. It's either full or none.***

## Your Mission

* You will be given a document consisting 9 problems from different domains. You can attempt any number of questions.
* Each domain has 3 questions categorized as Easy, Medium and Hard, carrying marks accordingly.
* Fork the corresponding repository first and then clone it. Solve the problem in your local machine.

    1. **Fork the repo**
    - Click the *Fork* button at the top-right corner of the repository.
    
    <img width="195" height="91" alt="image" src="https://github.com/user-attachments/assets/10ea1c07-b75b-429e-a2a1-0c1811f7609c" />
    
    2. **Clone your forked repo**
       
    - Click on *<>Code* and copy the HTTPS/SSH link. 
    
    <img width="172" height="82" alt="image" src="https://github.com/user-attachments/assets/83c60433-477e-405a-9c06-bc6945643009" /> 
    
    <img width="366" height="255" alt="image" src="https://github.com/user-attachments/assets/ac63f27e-316b-4588-a305-cd69e279161b" />

    1.

     ```bash
    https://github.com/Google-Developers-Group-IIIT-Lucknow/{repo-name}.git
    ```
    
    ```bash
    git@github.com:Google-Developers-Group-IIIT-Lucknow/{repo-name}.git
    ```
    
    2. **Then run in your local terminal:**
      
    ```bash
    git clone git@github.com:Google-Developers-Group-IIIT-Lucknow/{repo-name}.git
    ```

* While committing your changes, you might encounter some pre-commit hooks that would run some checks. You will be able to commit only when the pre-commit hook checks pass.

    1. **Add the changes**

    ```bash
    git add . # To add all the changes at one
    git add file-path # To add the changes in a single file
    ```

    2. **Commit changes**

    ```bash
    git commit -m "type a message here"
    ```
    
* Once, you are able to commit successfully, push your changes to your own fork.

    1. **Push the changes**

    ```bash
    git push origin {branch name}
    ```
    
* On your fork, there will be some more checks to validate your PR. In case, any of the checks fails, you will have to make changes, commit them and push again to re-run the workflow.
* Once the workflow succeed, you will be able to open your PR to the main repo of the problem. 

Once, your PR opens successfully, you can see the leadboard updating with your username getting more marks as per the difficulty. 

## Leaderboard : https://dev-sphere-leader-board.vercel.app/

## Rules

* **AI tools are strictly prohibited**
  → Immediate disqualification if detected
* **Google is allowed**


## **All the best!**
