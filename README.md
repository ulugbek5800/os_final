# docker_mini_project
Project to create simple webpage to check timetable use resources from docker

Guides

**1. Install Docker and Start PostgreSQL in Docker**

```bash
# Pull the latest PostgreSQL Docker image
docker pull postgres:latest

# Create a Docker container for the PostgreSQL database
docker run --name uni-db -e POSTGRES_USER=ulugbek -e POSTGRES_PASSWORD=ulugbek_pass -d -p 5432:5432 postgres:latest

# Verify the container is running
docker ps

# Access the running PostgreSQL container
docker exec -it uni-db psql -U ulugbek
```

### 2. **Set Up the PostgreSQL Database and Tables**

```sql
-- Create the Timetable table
CREATE TABLE Timetable (
    course_id SERIAL PRIMARY KEY,
    course_name VARCHAR(255),
    day VARCHAR(50),
    time VARCHAR(50),
    room VARCHAR(50),
    level INT
);

-- Insert Sample Data into the Timetable table
INSERT INTO Timetable (course_name, day, time, room, level) VALUES
('Computer Science 101', 'Monday', '9:00 AM', 'Room 101', 1),
('Operating Systems', 'Tuesday', '10:00 AM', 'Room 102', 1),
('Data Structures', 'Wednesday', '2:00 PM', 'Room 103', 2),
('Advanced Algorithms', 'Thursday', '11:00 AM', 'Room 104', 3),
('Machine Learning', 'Friday', '1:00 PM', 'Room 105', 2),
('Database Systems', 'Monday', '3:00 PM', 'Room 106', 3),
('Web Development', 'Tuesday', '11:00 AM', 'Room 107', 1),
('Networking', 'Thursday', '2:00 PM', 'Room 108', 2);
('Programming 1', 'Friday', '1:00 PM', 'Room 105', 1),
('Programming 2', 'Monday', '3:00 PM', 'Room 106', 1),
('Data Structures 1', 'Tuesday', '11:00 AM', 'Room 107', 2),
('Data Structures 2', 'Thursday', '2:00 PM', 'Room 108', 2)
('database Applications', 'Thursday', '2:00 PM', 'Room 108', 3);;

-- Optionally, create the Students table
CREATE TABLE Students (
    student_id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    level INT
);

-- Insert Sample Students Data
INSERT INTO Students (name, level) VALUES
('Ulugbek Suleymonov', 2),
('Mardon Rasulov', 2),
('Asad Xodjayev', 3),
('Doston Tolibov', 1);
```

### 3. **Install Python and Flask Dependencies**

```bash
# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate  # On Windows use venv\Scripts\activate

# Install Flask and pg8000 for database connection
pip install flask pg8000
```

### 4. **Create Flask Application**

```python
from flask import Flask, render_template, request
import pg8000

app = Flask(__name__)

@app.route("/", methods=["GET", "POST"])
def index():
    if request.method == "POST":
        # Get the level from the form
        level = request.form["level"]
        return render_template("timetable.html", level=level, data=[], message="Loading timetable...")
    
    return render_template("index.html")

@app.route("/timetable", methods=["GET"])
def timetable():
    level = request.args.get('level')  # Get level from the query parameters
    if not level:
        return "Level not provided", 400

    # Connect to the PostgreSQL database
    conn = pg8000.connect(
        user="ulugbek", 
        password="ulugbek_pass", 
        host="localhost", 
        port=5432, 
        database="postgres"
    )

    cur = conn.cursor()
    cur.execute("SET search_path TO public;")
    query = "SELECT * FROM timetable WHERE level = %s;"
    cur.execute(query, (level,))
    rows = cur.fetchall()

    # Pass data to the template
    if rows:
        return render_template("timetable.html", level=level, data=rows, message="")
    else:
        return render_template("timetable.html", level=level, data=[], message="No data found for this level.")


if __name__ == "__main__":
    app.run(debug=True)
```

### 5. **HTML Templates for Flask**

#### **`index.html` (Form for Level Input)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Timetable by Ulugbek</title>
</head>
<body>
    <h1>Welcome, Ulugbek</h1>
    <h2>University Timetable</h2>
    <form action="/timetable" method="GET">
        <label for="level">Select Level:</label>
        <select name="level" id="level">
            <option value="1">Level 1</option>
            <option value="2">Level 2</option>
            <option value="3">Level 3</option>
        </select>
        <button type="submit">Submit</button>
    </form>

</body>
</html>
```

#### **`timetable.html` (Displaying Timetable)**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Timetable for Level {{ level }}</title>
</head>
<body>
    <h1>Timetable for Level {{ level }}</h1>
    
    {% if data %}
        <table border="1">
            <thead>
                <tr>
                    <th>Course ID</th>
                    <th>Course Name</th>
                    <th>Day</th>
                    <th>Time</th>
                    <th>Room</th>
                </tr>
            </thead>
            <tbody>
                {% for row in data %}
                    <tr>
                        <td>{{ row[0] }}</td>
                        <td>{{ row[1] }}</td>
                        <td>{{ row[2] }}</td>
                        <td>{{ row[3] }}</td>
                        <td>{{ row[4] }}</td>
                    </tr>
                {% endfor %}
            </tbody>
        </table>
    {% else %}
        <p>{{ message }}</p>
    {% endif %}
    
    <a href="/">Go back</a>
</body>
</html>
```

### 6. **Running the Flask Application**

```bash
# Make sure you're in the directory with your Flask app
# Run the Flask app
python3 app.py
```

### 7. **Accessing the Application**

- Navigate to `http://127.0.0.1:5000/` in your web browser to see the application in action.

### 8. **Stop the Docker Container**

```bash
# Stop the PostgreSQL Docker container after you're done
docker stop university-db
```

---
