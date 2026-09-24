# Setting Up a Simple Data Visualization Dashboard with Flask

This guide is meant to help a complete beginner create a web app with Flask.

# Languages Used
Python, HTML/CSS, JavaScript, SQL (optional), SQLite (optional)

# Tools
You will need the following things:
- A python file named app.py
- Flask
- An excel spreadsheet / database file
- A folder labelled static with one file inside:  app.js
- A folder labelled templates with one file inside: index.html
- A Windows OS

# Step 1: Create a folder
This folder is going to hold everything in the project. You can name it whatever you want, lets go with myproject.

# Step 2: Obtain Your Data
Get your data and put it in your myproject folder, lets name it app.xlsx or app.db if you already have one. Acquiring the data can be done in a variety of ways, but the most important part is that you have an idea of what data is stored. 
Being familiar with the data will help you answer questions later on when we start to create charts. 

# Step 3: Install Flask

First, go ahead and update pip. Never hurts!

```py -m pip install --upgrade pip```

Now, If you do not already have Flask, go ahead and install that now using the Command Line Interface (CLI). 
Its strongly recommended to use folders and virtual environments to keep things orderly when you download flask, good thing we made that folder earlier! 

Here we can just ```cd myproject``` into the folder.

Next, we want to make the virtual environment. We can use ```py -3 -m venv .venv``` Now activate the virtual environment with 

```.venv\Scripts\activate```. 

You'll know it worked when you see the shell name change.

Now install Flask! 
```pip install Flask```

You can also check the version just to be sure. 
```flask --version```

# Step 4: Create folders and files
Make sure you are cd'ed into the myproject folder. 

First, create createDB.py, then app.py.

```type nul > createDB.py```

```type nul > app.py```

Make the static and templates folders next.

```mkdir static```

```mkdir templates```

CD into the static folder and create app.js

```cd static```

```type nul > app.js```

CD out

```cd ..```

CD into the templates folder

```cd templates```

And make the index.html file

```type nul > index.html```

CD back to the myproject folder

```cd ..```

# Step 5: Create the Database (Optional. Skip if you already have the db file.)
Lets open the createDB.py file. Here we want to insert the following code:

```
from pathlib import Path
import pandas as pd
from sqlalchemy import create_engine

# 1. Dynamically target user's documents folder path
folder = Path.home() / INSERT THE PATH TO myproject HERE
db_file_path = folder / "app.db"

# 2. Format the path for SQLite connection string
db_url = f"sqlite:///{db_file_path.as_posix()}"

# 3. Read the Excel data
excel_file = "app.xlsx"
df = pd.read_excel(excel_file, sheet_name="Sheet 1") # make sure your sheet name matches here

# 4. Clean column names
df.columns = [str(col).strip().replace(' ', '_').lower() for col in df.columns]

# 5. Create engine and push dataframe to SQL
engine = create_engine(db_url)

# 6.  if_exists options: 'fail', 'replace' (drops old table & recreates), or 'append'
df.to_sql(name="my_table", con=engine, if_exists="replace", index=False) # note that the name of this table is "my_table"

# 8. Confirmation
print(f"Data successfully saved to SQLite database file at: {db_file_path}")
```
Now, you can run this file and it should create the app.db file.

# Step 6: Connect the app.db and index.html to app.py

The app.py file should look like this: 
```
from flask import Flask, jsonify, render_template
import sqlite3

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/api/db')
def get_all():
    # opens database
    conn = sqlite3.connect('app.db') # connects to file
    cursor = conn.cursor() # sends query, reads results

    # run query
    cursor.execute('SELECT * FROM my_table') #cursor, read everything from the table
    rows = cursor.fetchall() # cursor, translate all this data into python and save it in rows
    columns = [desc[0] for desc in cursor.description] #cursor, get the column names
    conn.close() # close connection
    data = [dict(zip(columns, row)) for row in rows] # zip, pair each name with its value and turn it into a dictionary
    return jsonify(data) # turn into json in browser

if __name__ == '__main__':
    app.run()

```

# Step 7: Connect the index.html to app.js and to Plotly

The index.html file should look like this: 
```
<head>
<script src="https://cdn.plot.ly/plotly-2.35.2.min.js"></script>
</head>
<body>
<script src="{{ url_for('static', filename='app.js') }}"></script>
</body>
```

# Step 8: Connect app.js to app.db

The app.js file should look like this:

```
fetch('/api/db') // sends request to db
    .then()
    .catch(error => console.error('Error fetching data:', error));
```

# Step 9: Coding!

With everything connected, you are now free to use Plotly to create whichever charts fit your data. 
