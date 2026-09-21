1a   GET /get-student-by-name?name=Ada uses Student.find({ name }).
    If two Adas exist, what is the shape of the JSON the client actually receives?

    Student.find({ name }) searches the database and returns all the students named Ada in an ARRAY. irrespective of the number,
    since there are 2 students created with the name Ada, it returns both students. (app.js, line 83-91)
    EXAMPLE: "message": "Student fetched successfully",
             "student": [
            {
                 "_id": "6aabce4ea3366cfc451d4003",
                "name": "Ada",
                "age": 13,
                "email": "aaaaaaaa@gmail.com",
                 "phone": "01111111111",
                "address": "#8 okon avenue",
                "course": "SLT",
                 "__v": 0
                },
                {
                "_id": "6aabcf44a3366cfc451d4004",
                "name": "Ada",
                "age": 17,
                "email": "bbbbbbbbbbbb@gmail.com",
                "phone": "02222222222",
                "address": "#8 akpan estate",
                "course": "Computer science",
            "__v": 0
            }
            ]
            }

1b   Would findOne behave differently?
    YES, findOne simply returns only one matching student(Ada). it will NOT return both students.
    Also, since the name "Ada" is not unique, findOne will not tell us which Ada the Admin intended to retrieve

1c  When would you use each?
    Student.find({ name }) is used when we wants to return all the students named "Ada" from the databas.e
    findOne is used when our intention is to return any one student named "Ada" without any criteria.



2a  That same route is case-sensitive and requires an exact name.
    What happens if the admin searches ada or Ada (trailing space)?
    both "ada" and "Ada " does not match any student stored in our database(ie, they're both not the same as "Ada")
    so both options throws up an error as mongoDB by default operates on case-sensitive basis.

2b  how to make a MongoDB/Mongoose name search case-insensitive without fetching every student into Node and filtering in JavaScript.
    This is done by using the MongoDB's $regex with the i option.
    ie, line 86 in app. js will be changed to,
    const student = await student.find({
        name: {$regex: name.trim(), $options: "i"
    });
    Here, name.trim() removes extra spaces in "Ada ", while $option: "i" makes the search to ignore uppercase/lowercase differences.
    therefore "ada" and "Ada " can both match the name in our database "Ada"



3a  PUT /update-student/:id uses findByIdAndUpdate(..., { new: true }).
    The admin swears she still sees the old document. Give two possible reasons this can happen:
    one that is about how she is calling the endpoint (params vs body, method, URL)
 Ans: This implies that the Admin is putting the id in the wrong place. The id must be in :id in the url(app.js, line 73)
     while the fields that needs to e changed goes into the body.(app.js, line 74)

3b  one that is about Mongoose update options (research new, runValidators, and what happens to fields she did not send in the body)
Ans: new:true, tells Mongoose to return the updated parameter, so this is not the problem as it is currently captured in app.js (
    line 76)
    runValidator:true, makes mongoose to apply certain scema validation during an update.
    Fields that she did not send in the body is expected to remian unchanged rather than being removed.



4a  GET /get-student/:id
    Valid ObjectId, student exists → ?
    if the id is valid and exist in MongoDB database, findById returns the student's information with code  200(app.js line 65)

4b  Valid ObjectId, student does not exist → what does findById return, and what status does your code send?
    If the id is in a valid ObjectId format, even though it is incorrect(ie, there is no student with such id)
    it returns "null" and does NOT throw an error, with status code 200

4c  Invalid id such as abc123 → why is this a 500 and not a 404?
    Research: CastError vs "document not found". They are not the same bug.
    It returns error code 500, this is because the id (abc123) is not a valid id in the first place, so Mongoose cannot
    properly perform a lookup, so it throws a castError.
    404 response code means the requested id was not found at all.


5  mongoose.model("Student", studentSchema)
    What is the actual collection name in MongoDB Compass / mongosh?
    (Hint: it is probably not "Student".) Explain why that matters if someone writes a raw Mongo query against the wrong collection and thinks "the API is empty".

    The correct collection name in MpngoDB compass is "students", MongoDB normally pluralizes and lowercases the model name as the MongoDB collection name. so it is "students" not "Student"
    This matters because, someone might run a db.Student.find() instead of db.students.find(), and may conclude that the API is empty.


6   Why does POST /create-student fail (or save undefined fields) if the client forgets Content-Type: application/json?
    Which one line in app.js is responsible for making req.body work at all?

    Express will not phrase the request body as JSON, so, req.body will be undefined causing the request to fail.
    The line responsible for making req.body work is line 8 (app.use(express.json());





