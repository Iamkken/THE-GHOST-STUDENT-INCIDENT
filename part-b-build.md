1.  app.get('/search-students', async (req, res) => {
  const { q } = req.query;
  try {
    if (!q) {
      return res.status(400).json({ message: "Query parameter 'q' is required" });
    }

    const students = await Student.find({
      $or: [
        { name: { $regex: q, $options: "i" } },
        { email: { $regex: q, $options: "i" } },
        { course: { $regex: q, $options: "i" } }
      ]
    });

    return res.status(200).json({ message: "Students fetched successfully", students });
  } catch (error) {
    return res.status(500).json({ message: "Internal server error" });
  }
});

A list/search endpoint normally returns an empty collection when there are no matches because 
the request itself was valid. code 404 is generally used when a specific resource cannot be found rather than
when a colection search has zero result.


2.  app.get("/get-student/:id", async (req, res) => {
  const { id } = req.params;
  if (!mongoose.Types.ObjectId.isValid(id)) {
    return res.status(400).json({ message: "Invalid student ID" }); 
  }
  try {
    const student = await Student.findById(id);
    if (!student) {
      return res.status(404).json({ message: "Student not found" });
    }
    return res.status(200).json({ message: "Student fetched successfully", student });
  } catch (error) {
    return res.status(500).json({ message: "Internal server error" });
  }
});

//isValid() returns 200 with student: null when a valid but non existent ID is inputted. However, 
//mongoose.Types.ObjectId.isValid returns 400 as invalid Id, and 404 when a valid Id but non existent student is used. 


3.  app.patch('/students/:id/course', async (req, res) => {
  const {id} = req.params;
  const {course} = req.body;
  
  if(!course){
    return res.status(400).json ({message: "Course is required"});
  }
  try{
    const student = await Student.findByIdAndUpdate(
      id,
      {course},
      {new: true, runValidators: true});

      if(!student) {
        return res.status(404).json ({message: "Student not found"});
      }
      return res.status(200).json ({message:"Student course updated successfully",student});
    } catch(error) {
      if (error.name ==="ValidationError"){
        return res.sendStatus(400).json({ message: error.message});
      }
      return res.status(500).json ({message: "Internal server error"});
    }
  });

  {new: true, runValidator: true}
    new: true >> returns the updated student
    runValidator: true >> makes the mongoose to enforce the minlength rule during the update


4.  unique: true >>> is not just enough, becaise the database needs to actualy have that unique index. 

  const studentSchema = new mongoose.Schema({
  name: { type: String, required: true },
  age: Number,
  email: { type: String, required: true, unique: true },
  phone: String,
  address: String,
  course: { type: String, minlength: 2 },
  institution: String
  });

  app.post("/create-student", async (req, res) => {
  const { name, age, email, phone, address, course, institution } = req.body;
  If(!name || !email) 
  {
    return res.status(400).json ({message: "Name and email required"});
  }
  try{
    const student = new Student({
      name,
      age,
      email,
      phone,
      address,
      course,
      institution
    });
    await student.save();
    return res.status(200).json ({message: " Student created successfully", student});
  } catch (error) {
    if (error.code === 11000) {
      return res.status (409).json({message: "Email already exist"});
    }
    if(error.name === "ValidationError") {
      return res.status(400).json ({message: error.message});
    }
    return res.status(500).json({ message: "Internal server error"});
    }
    });


5.  app.delete("/delete-student/:id", async (req, res) => {
  const { id } = req.params;
  if (!mongoose.Types.ObjectId.isValid(id)) {
    return res.status(400).json ({message: "Invalid student ID"});
  }
  try{
    const student = await Student.findByIdAndDelete(id);
    if(!student) {
      return res.status(404).json ({ message: "Student not found; nothing was deleted"});
    }
    return res.status(200).json ({message: "Student deleted successfully"});
  } catch (error) {
    return res.status(500).json ({message: "Internal server error"});
    }
});

//200 is used because we return a confirmation message in the response
//mongoose.Types.ObjectId.isValid returns 400 as invalid Id, and 404 when a valid Id but non existent student is used. 

