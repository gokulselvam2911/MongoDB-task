# Zen Class Program

This project represents the backend schema and queries for a Zen class program, which includes different collections such as `users`, `codekata`, `attendance`, `topics`, `tasks`, `company_drives`, and `mentors`. These collections allow for tracking of student activities, mentoring details, company drives, attendance, and more.

## Project Structure

The project includes MongoDB collections and queries designed to manage the following aspects of the Zen class program:

- **Users**: Stores user details (students and mentors), their participation in coding challenges, and company drives.
- **Codekata**: Tracks coding problems solved by users.
- **Attendance**: Manages attendance records for each user.
- **Topics**: Details of the topics taught in the program.
- **Tasks**: Stores tasks associated with each topic.
- **Company Drives**: Keeps track of company recruitment drives and the students who appeared for them.
- **Mentors**: Stores mentor information and the mentees they are mentoring.

## MongoDB Schema

### 1. **Users Collection (users)**

This collection contains details about users like their name, email, role (student or mentor), and their codekata score.

```javascript
db.createCollection("users");

db.users.insertMany([
  {
    _id: ObjectId("user_id_1"),
    name: "John Doe",
    email: "john.doe@example.com",
    role: "student",
    codekata_score: 150,
    company_drives: [
      ObjectId("company_drive_id_1"),
      ObjectId("company_drive_id_2")
    ]
  },
  {
    _id: ObjectId("user_id_2"),
    name: "Jane Smith",
    email: "jane.smith@example.com",
    role: "mentor",
    mentees: [
      ObjectId("user_id_3"),
      ObjectId("user_id_4")
    ]
  }
]);
```

### 2. **Codekata Collection (codekata)**

Stores the coding problems solved by users.

```javascript
db.createCollection("codekata");

db.codekata.insertMany([
  {
    _id: ObjectId("problem_id_1"),
    user_id: ObjectId("user_id_1"),
    problem_name: "Binary Search",
    difficulty: "Medium",
    solved_on: ISODate("2020-10-10")
  }
]);
```

### 3. **Attendance Collection (attendance)**

Tracks attendance for each user on a given date.

```javascript
db.createCollection("attendance");

db.attendance.insertMany([
  {
    _id: ObjectId("attendance_id_1"),
    user_id: ObjectId("user_id_1"),
    date: ISODate("2020-10-01"),
    status: "present"
  },
  {
    _id: ObjectId("attendance_id_2"),
    user_id: ObjectId("user_id_1"),
    date: ISODate("2020-10-02"),
    status: "absent"
  }
]);
```

### 4. **Topics Collection (topics)**

Contains information about topics taught in the Zen class program.

```javascript
db.createCollection("topics");

db.topics.insertMany([
  {
    _id: ObjectId("topic_id_1"),
    topic_name: "Data Structures",
    description: "Introduction to arrays, linked lists, trees, etc.",
    start_date: ISODate("2020-10-01"),
    end_date: ISODate("2020-10-10")
  }
]);
```

### 5. **Tasks Collection (tasks)**

Stores the tasks associated with each topic.

```javascript
db.createCollection("tasks");

db.tasks.insertMany([
  {
    _id: ObjectId("task_id_1"),
    topic_id: ObjectId("topic_id_1"),
    task_name: "Linked List Implementation",
    description: "Implement a singly linked list in your preferred language",
    due_date: ISODate("2020-10-10")
  }
]);
```

### 6. **Company Drives Collection (company_drives)**

Stores information about company recruitment drives.

```javascript
db.createCollection("company_drives");

db.company_drives.insertMany([
  {
    _id: ObjectId("company_drive_id_1"),
    company_name: "Google",
    date: ISODate("2020-10-20"),
    students_appeared: [
      ObjectId("user_id_1"),
      ObjectId("user_id_2")
    ]
  }
]);
```

### 7. **Mentors Collection (mentors)**

Contains information about mentors and the mentees they mentor.

```javascript
db.createCollection("mentors");

db.mentors.insertMany([
  {
    _id: ObjectId("mentor_id_1"),
    name: "Mentor A",
    mentees: [
      ObjectId("user_id_1"),
      ObjectId("user_id_2")
    ]
  }
]);
```

## Queries

### 1. **Find all topics and tasks taught in October**

```javascript
db.topics.find({
  start_date: { $gte: ISODate("2020-10-01T00:00:00Z") },
  end_date: { $lte: ISODate("2020-10-31T23:59:59Z") }
});

db.tasks.find({
  due_date: { $gte: ISODate("2020-10-01T00:00:00Z"), $lte: ISODate("2020-10-31T23:59:59Z") }
});
```

### 2. **Find all company drives between 15th October 2020 and 31st October 2020**

```javascript
db.company_drives.find({
  date: { $gte: ISODate("2020-10-15T00:00:00Z"), $lte: ISODate("2020-10-31T23:59:59Z") }
});
```

### 3. **Find all company drives and students who appeared for the placement**

```javascript
db.company_drives.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "students_appeared", 
      foreignField: "_id", 
      as: "students"
    }
  }
]);
```

### 4. **Find the number of problems solved by a user in Codekata**

```javascript
db.codekata.aggregate([
  { $match: { user_id: ObjectId("user_id_1") } },
  { $count: "solved_problems_count" }
]);
```

### 5. **Find all the mentors with more than 15 mentees**

```javascript
db.mentors.find({
  $expr: { $gt: [{ $size: "$mentees" }, 15] }
});
```

### 6. **Find the number of users who are absent and tasks are not submitted between 15th October 2020 and 31st October 2020**

```javascript
db.attendance.aggregate([
  {
    $match: {
      date: { $gte: ISODate("2020-10-15T00:00:00Z"), $lte: ISODate("2020-10-31T23:59:59Z") },
      status: "absent"
    }
  },
  {
    $lookup: {
      from: "tasks",
      localField: "user_id", 
      foreignField: "user_id",
      as: "tasks"
    }
  },
  {
    $match: {
      "tasks.status": { $ne: "submitted" }
    }
  },
  {
    $count: "absent_and_non_submitted_count"
  }
]);
```

## Technologies Used

* MongoDB
