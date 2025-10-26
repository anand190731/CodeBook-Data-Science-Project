# 💻 CodeBook: The Social Media for Coders

## 🧠 Project Overview
Welcome to **CodeBook – Your Data Science Internship Begins!**  
This project simulates a real-world data science internship at a fictional company called **CodeBook – The Social Media for Coders**.  
Your challenge: analyze, clean, and build recommendation features using **only pure Python** — no pandas, NumPy, or external libraries!  

---

## 🏢 Scenario
You’ve just been hired as a **Data Science Intern** at CodeBook, a Delhi-based company.  
Your manager **Puneet Kumar** has given you a series of tasks.  
If you complete them successfully, you’ll receive a **₹10 LPA full-time offer**!  

---

# 🧩 Task 1: Data Loading & Exploration

### 🎯 Objective
Load and explore the user data from a JSON file to understand its structure.

### 📁 Dataset Example
```json
{
    "users": [
        {"id": 1, "name": "Amit", "friends": [2, 3], "liked_pages": [101]},
        {"id": 2, "name": "Priya", "friends": [1, 4], "liked_pages": [102]},
        {"id": 3, "name": "Rahul", "friends": [1], "liked_pages": [101, 103]},
        {"id": 4, "name": "Sara", "friends": [2], "liked_pages": [104]}
    ],
    "pages": [
        {"id": 101, "name": "Python Developers"},
        {"id": 102, "name": "Data Science Enthusiasts"},
        {"id": 103, "name": "AI & ML Community"},
        {"id": 104, "name": "Web Dev Hub"}
    ]
}
```

### 🧰 Code Implementation
```python
import json

def load_data(filename):
    with open(filename, "r") as file:
        data = json.load(file)
    return data

def display_users(data):
    print("Users and Their Connections:\n")
    for user in data["users"]:
        print(f"{user['name']} (ID: {user['id']}) - Friends: {user['friends']} - Liked Pages: {user['liked_pages']}")
    print("\nPages:\n")
    for page in data["pages"]:
        print(f"{page['id']}: {page['name']}")

data = load_data("data.json")
display_users(data)
```

### ✅ Expected Output
```
Users and Their Connections:
Amit (ID: 1) - Friends: [2, 3] - Liked Pages: [101]
Priya (ID: 2) - Friends: [1, 4] - Liked Pages: [102]
Rahul (ID: 3) - Friends: [1] - Liked Pages: [101, 103]
Sara (ID: 4) - Friends: [2] - Liked Pages: [104]

Pages:
101: Python Developers
102: Data Science Enthusiasts
103: AI & ML Community
104: Web Dev Hub
```

---

# 🧹 Task 2: Data Cleaning & Structuring

### 🎯 Objective
Clean the data by handling missing values, removing duplicates, and standardizing formats.

### 🧩 Problems Found
- Empty or missing names  
- Duplicate friend entries  
- Inactive users with no friends or liked pages  
- Duplicate pages with same IDs  

### 🧰 Code Implementation
```python
import json

def clean_data(data):
    data["users"] = [user for user in data["users"] if user["name"].strip()]
    for user in data["users"]:
        user["friends"] = list(set(user["friends"]))
    data["users"] = [user for user in data["users"] if user["friends"] or user["liked_pages"]]
    unique_pages = {}
    for page in data["pages"]:
        unique_pages[page["id"]] = page
    data["pages"] = list(unique_pages.values())
    return data

data = json.load(open("data2.json"))
data = clean_data(data)
json.dump(data, open("cleaned_data.json", "w"), indent=4)
print("Data cleaned successfully!")
```

### ✅ Expected Output
```json
{
    "users": [
        {"id": 1, "name": "Amit", "friends": [2, 3], "liked_pages": [101]},
        {"id": 2, "name": "Priya", "friends": [1, 4], "liked_pages": [102]},
        {"id": 4, "name": "Sara", "friends": [2], "liked_pages": [104]}
    ],
    "pages": [
        {"id": 101, "name": "Python Developers"},
        {"id": 102, "name": "Data Science Enthusiasts"},
        {"id": 103, "name": "AI & ML Community"},
        {"id": 104, "name": "Web Development"}
    ]
}
```

---

# 👥 Task 3: People You May Know

### 🎯 Objective
Recommend users new friends based on mutual connections.

### ⚙️ Logic
- If User A and User B are not friends but share mutual friends, recommend them to each other.
- More mutual friends = higher priority.

### 🧰 Code Implementation
```python
import json

def load_data(filename):
    with open(filename, "r") as file:
        return json.load(file)

def find_people_you_may_know(user_id, data):
    user_friends = {user["id"]: set(user["friends"]) for user in data["users"]}
    if user_id not in user_friends:
        return []
    direct_friends = user_friends[user_id]
    suggestions = {}

    for friend in direct_friends:
        for mutual in user_friends[friend]:
            if mutual != user_id and mutual not in direct_friends:
                suggestions[mutual] = suggestions.get(mutual, 0) + 1

    sorted_suggestions = sorted(suggestions.items(), key=lambda x: x[1], reverse=True)
    return [uid for uid, _ in sorted_suggestions]

data = load_data("massive_data.json")
print(find_people_you_may_know(1, data))
```

### ✅ Example Output
```
People You May Know for User 1: [7,8,9,10,11,12]
```

---

# 📄 Task 4: Pages You Might Like

### 🎯 Objective
Recommend pages to users based on similar interests using collaborative filtering.

### ⚙️ Logic
- If two users like the same pages, recommend other pages one likes to the other.
- Rank recommendations by shared page count.

### 🧰 Code Implementation
```python
import json

def load_data(filename):
    with open(filename, "r") as file:
        return json.load(file)

def find_pages_you_might_like(user_id, data):
    user_pages = {user["id"]: set(user["liked_pages"]) for user in data["users"]}
    if user_id not in user_pages:
        return []

    user_liked = user_pages[user_id]
    page_suggestions = {}

    for other_user, pages in user_pages.items():
        if other_user != user_id:
            shared = user_liked.intersection(pages)
            for page in pages:
                if page not in user_liked:
                    page_suggestions[page] = page_suggestions.get(page, 0) + len(shared)

    sorted_pages = sorted(page_suggestions.items(), key=lambda x: x[1], reverse=True)
    return [page for page, _ in sorted_pages]

data = load_data("massive_data.json")
print(find_pages_you_might_like(1, data))
```

### ✅ Example Output
```
Pages You Might Like for User 1: [(103, 2), (105, 1), (107, 1), (104, 0), (106, 0), (108, 0), (109, 0), (110, 0), (111, 0), (112, 0), (113, 0), (114, 0), (115, 0), (116, 0), (117, 0), (118, 0), (119, 0), (120, 0), (121, 0), (122, 0), (123, 0), (124, 0), (125, 0), (126, 0), (127, 0)]
```

---

# 🏁 Conclusion
✅ Loaded and explored data using **JSON and core Python**  
✅ Cleaned and structured data (removed duplicates, handled missing values)  
✅ Built **People You May Know** using mutual friend logic  
✅ Built **Pages You Might Like** using collaborative filtering  

From raw data to smart recommendations — all built using pure Python! 🐍  

---

## 🧰 Tools Used
- **Language:** Python 3  
- **Modules:** `json`, `os`, `collections`  
- **Libraries:** None (No pandas or NumPy)  

---

## 🧠 Key Learnings
- Data wrangling using native Python data structures  
- Recommendation system logic design  
- Collaborative filtering fundamentals  
- Problem-solving with real-world datasets  

---

## 📂 Repository Structure
```
├── data.json
├── data2.json
├── cleaned_data.json
├── massive_data.json
├── 01_load_and_display_data.ipynb
├── 02_data_cleaning.ipynb
├── 03_people_you_may_know.ipynb
├── 04_pages_you_might_like.ipynb
└── README.md
```

---

## ✨ Author
**Anandam P M**  
Aspiring Data Scientist | Learning Python, SQL, and Machine Learning  
📍 Kannur, Kerala  
🔗 [LinkedIn](https://www.linkedin.com/in/anandampm](https://www.linkedin.com/in/anandam-p-m))  
