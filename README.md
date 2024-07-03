# GUVI TASK - DAY 36

# Design database for Zen class programme

1.  About the Task.

    > - In this task, we are given a list of entities for us to design a robust database which can help us solve the given problems.
    > - Given entities are:

    1. Users
    2. Mentors
    3. Tasks
    4. Topics
    5. Attendance
    6. Company Drives
    7. Code Kata

2.  Setup

    > - First, we create a database, using the MongoDB Compass UI.
    > - We create a database named `day36` and a collection named `users` at start.
    > - Then, we create 6 more collections for our other 6 entities.

3.  Solution

    1.  Find all the topics and tasks which are taught in the month of October.

        ```
        db.tasks.aggregate([
          {
              $match: {
                  date: {
                      $gte: "2020-10-01",
                      $lt: "2024-11-01"
                  }
              }
          }
        ]);
        ```

    2.  Find all the company drives which appeared between 15 oct-2020 and 31-oct-2020.

        ```
        db.company_drives.aggregate([
          {
              $match: {
                      date: {
                          $gte: "2020-10-15",
                          $lt: "2020-10-31"
                      }
                  }
          }
        ]);
        ```

    3.  Find all the company drives and students who appeared for the placement.

        ```
        db.company_drives.aggregate([
          {
              $unwind: "$attendees"
          },
          {
              $lookup: {
              from: "users",
              localField: "attendees.id",
              foreignField: "_id",
              as: "studentDetails"
              }
          },
          {
              $unwind: "$studentDetails"
          },
          {
              $group: {
                  _id: "$_id",
                  company_name: { $first: "$company_name" },
                  date: { $first: "$date" },
                  attendees: {
                      $push: {
                      id: "$attendees.id",
                      status: "$attendees.status",
                      studentDetails: "$studentDetails"
                      }
                  }
              }
          }
        ]);
        ```

    4.  Find the number of problems solved by the user in codekata.

        ```
        db.code_kata.aggregate([
          {
              $match:{
                  "user_id": ObjectId("66715716582c0c7790fa280e")
              }

          },
          {
              $project: {
                  user_id: 1,
                  problemsSolved: { $size: "$problems_solved" }
              }
          }
        ]);
        ```

    5.  Find all the mentors who have a mentee's count of more than 15.

        ```
        db.mentors.aggregate([
          {
              $project: {
                  name: 1,
                  menteeCount: { $size: "$mentees" }
              }
          },
          {
              $match: {
                  menteeCount: { $gt: 15 }
              }
          }
        ]);
        ```

    6.  Find the number of users who are absent and task is not submitted between 15 oct-2020 and 31-oct-2020.

      
        ```
        db.attendance.aggregate([
            {
                $match:{
                    "status":{
                        $eq:"absent",
                    },
                    "date":{
                        $gte:"2020-10-15",
                        $lt:"2020-10-31"
                    }
                }
            },
            {
                $count:"totalUsers"
            }
        ]);

        db.tasks.aggregate([
            {
                $match:{
                    "submissions.status":{
                        $eq:"not submitted",
                    },
                    "date":{
                        $gte:"2020-10-15",
                        $lt:"2020-10-31"
                    }
                }
            },
            {
                $count:"totalUsers"
            }
        ]);
         ```
      
