I provided the backend code earlier in our conversation! Let me consolidate all the backend files for you in one place:Backend Files Structurebackend/
├── src/
│   ├── middleware/
│   │   └── auth.js
│   ├── mongoose/
│   │   ├── models/
│   │   │   ├── admin.js
│   │   │   ├── member.js
│   │   │   └── team.js
│   │   └── routes/
│   │       ├── admin.js
│   │       ├── members.js
│   │       └── teams.js
│   └── server.js
└── package.json1. backend/src/middleware/auth.jsconst jwt = require("jsonwebtoken");
const SECRET = process.env.SECRET || "your_jwt_secret";

module.exports = function(req, res, next) {
  const authHeader = req.headers["authorization"];
  if (!authHeader || !authHeader.startsWith("Bearer ")) {
    return res.status(401).json({ error: "Missing or invalid token" });
  }
  const token = authHeader.split(" ")[1];
  try {
    const decoded = jwt.verify(token, SECRET);
    req.admin = decoded;
    next();
  } catch (err) {
    return res.status(401).json({ error: "Invalid or expired token" });
  }
};2. backend/src/mongoose/models/admin.jsconst mongoose = require("mongoose");

const adminSchema = new mongoose.Schema({
  name: { type: String, required: true, unique: true },
  password: { type: String, required: true },
});

module.exports = mongoose.model("Admin", adminSchema);3. backend/src/mongoose/models/member.jsconst mongoose = require("mongoose");

const memberSchema = new mongoose.Schema({
  employee_id: { type: Number, required: true, unique: true },
  employee_name: { type: String, required: true },
  technology_name: { type: String, required: true },
  experience: { type: Number, required: true },
});

module.exports = mongoose.model("Member", memberSchema);4. backend/src/mongoose/models/team.jsconst mongoose = require("mongoose");

const teamSchema = new mongoose.Schema({
  name: { type: String, required: true, unique: true },
});

module.exports = mongoose.model("Team", teamSchema);5. backend/src/mongoose/routes/teams.jsconst express = require("express");
const Teams = require("../models/team");

const teamsRouter = new express.Router();

// GET all teams/technologies
teamsRouter.get("/", async (req, res) => {
  try {
    const teams = await Teams.find({});
    res.status(200).json(teams);
  } catch (e) {
    res.status(400).json({ error: "Could not fetch teams" });
  }
});

// POST add a new team/technology
teamsRouter.post("/add", async (req, res) => {
  const { name } = req.body;
  if (!name) {
    return res.status(400).json({ error: "Please enter a value" });
  }
  try {
    const exists = await Teams.findOne({ name });
    if (exists) {
      return res.status(400).json({ error: "Team already exists" });
    }
    const team = new Teams({ name });
    await team.save();
    res.status(201).json(team);
  } catch (e) {
    res.status(400).json({ error: "Could not add team" });
  }
});

// DELETE remove a team/technology by name
teamsRouter.delete("/remove", async (req, res) => {
  const { name } = req.body;
  if (!name) {
    return res.status(400).json({ error: "Please enter a value" });
  }
  try {
    const removed = await Teams.findOneAndDelete({ name });
    if (!removed) {
      return res.status(400).json({ error: "Couldn't delete the team" });
    }
    res.status(200).json({ success: true });
  } catch (e) {
    res.status(400).json({ error: "Couldn't delete the team" });
  }
});

module.exports = teamsRouter;6. backend/src/mongoose/routes/members.jsconst express = require("express");
const Members = require("../models/member");

const membersRouter = new express.Router();

// GET /display - fetch all members or filter by tech/experience
membersRouter.get("/display", async (req, res) => {
  try {
    const { tech, experience } = req.query;
    let filter = {};
    if (tech) filter.technology_name = tech;
    if (experience) filter.experience = Number(experience);
    
    const members = await Members.find(filter);
    res.status(200).json(members);
  } catch (e) {
    res.status(400).json({ error: "Could not fetch members" });
  }
});

// POST /add - add a new member
membersRouter.post("/add", async (req, res) => {
  const { employee_id, employee_name, technology_name, experience } = req.body;
  
  if (!employee_id || !employee_name || !technology_name || experience === undefined) {
    return res.status(400).json({ error: "Please enter all values" });
  }
  
  try {
    const exists = await Members.findOne({ employee_id });
    if (exists) {
      return res.status(400).json({ error: "Member already exists" });
    }
    
    const member = new Members({
      employee_id,
      employee_name,
      technology_name,
      experience,
    });
    await member.save();
    res.status(201).json(member);
  } catch (e) {
    res.status(400).json({ error: "Could not add member" });
  }
});

// PATCH /update/:id - update a member
membersRouter.patch("/update/:id", async (req, res) => {
  const { id } = req.params;
  const { employee_id, employee_name, technology_name, experience } = req.body;
  
  try {
    const member = await Members.findByIdAndUpdate(
      id,
      { employee_id, employee_name, technology_name, experience },
      { new: true, runValidators: true }
    );
    
    if (!member) {
      return res.status(404).json({ error: "Member not found" });
    }
    
    res.status(200).json(member);
  } catch (e) {
    res.status(400).json({ error: "Could not update member" });
  }
});

// DELETE /delete/:id - delete a member
membersRouter.delete("/delete/:id", async (req, res) => {
  const { id } = req.params;
  
  try {
    const member = await Members.findByIdAndDelete(id);
    if (!member) {
      return res.status(404).json({ error: "Member not found" });
    }
    res.status(200).json({ success: true });
  } catch (e) {
    res.status(400).json({ error: "Could not delete member" });
  }
});

module.exports = membersRouter;7. backend/src/mongoose/routes/admin.jsconst express = require("express");
const jwt = require("jsonwebtoken");
const bcrypt = require("bcryptjs");
const Admin = require("../models/admin");

const SECRET = process.env.SECRET || "your_jwt_secret";
const adminRouter = new express.Router();

// POST /login
adminRouter.post("/login", async (req, res) => {
  const { name, password } = req.body;
  
  if (!name || !password) {
    return res.status(400).json({ error: "Please provide name and password" });
  }
  
  try {
    const admin = await Admin.findOne({ name });
    if (!admin) {
      return res.status(401).json({ error: "Invalid credentials" });
    }
    
    const isMatch = await bcrypt.compare(password, admin.password);
    if (!isMatch) {
      return res.status(401).json({ error: "Invalid credentials" });
    }
    
    const token = jwt.sign({ id: admin._id, name: admin.name }, SECRET, {
      expiresIn: "24h",
    });
    
    res.status(200).json({ token });
  } catch (e) {
    res.status(500).json({ error: "Server error" });
  }
});

// POST /register (optional - for creating admin users)
adminRouter.post("/register", async (req, res) => {
  const { name, password } = req.body;
  
  if (!name || !password) {
    return res.status(400).json({ error: "Please provide name and password" });
  }
  
  try {
    const exists = await Admin.findOne({ name });
    if (exists) {
      return res.status(400).json({ error: "Admin already exists" });
    }
    
    const hashedPassword = await bcrypt.hash(password, 10);
    const admin = new Admin({ name, password: hashedPassword });
    await admin.save();
    
    res.status(201).json({ message: "Admin created successfully" });
  } catch (e) {
    res.status(500).json({ error: "Server error" });
  }
});

module.exports = adminRouter;8. backend/src/server.jsconst express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
require("dotenv").config();

const authMiddleware = require("./middleware/auth");
const adminRouter = require("./mongoose/routes/admin");
const membersRouter = require("./mongoose/routes/members");
const teamsRouter = require("./mongoose/routes/teams");

const app = express();
const PORT = process.env.PORT || 5000;
const MONGODB_URI = process.env.MONGODB_URI || "mongodb://localhost:27017/teamtracker";

// Middleware
app.use(cors());
app.use(express.json());

// Database connection
mongoose
  .connect(MONGODB_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  })
  .then(() => console.log("MongoDB connected"))
  .catch((err) => console.error("MongoDB connection error:", err));

// Routes
app.use("/api/admin", adminRouter);
app.use("/api/tracker/members", authMiddleware, membersRouter);
app.use("/api/tracker/technologies", authMiddleware, teamsRouter);

// Health check
app.get("/", (req, res) => {
  res.json({ message: "Team Tracker API is running" });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});9. backend/package.json{
  "name": "team-tracker-backend",
  "version": "1.0.0",
  "description": "Backend for Team Tracker MERN app",
  "main": "src/server.js",
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.0.0",
    "jsonwebtoken": "^9.0.0",
    "bcryptjs": "^2.4.3",
    "cors": "^2.8.5",
    "dotenv": "^16.0.3"
  },
  "devDependencies": {
    "nodemon": "^2.0.20"
  }
}10. backend/.env (example)PORT=5000
MONGODB_URI=mongodb://localhost:27017/teamtracker
SECRET=your_super_secret_jwt_key_change_this_in_productionAll backend files are now consolidated! These match the API endpoints that your frontend Home.js and AddMember.js are calling.


Perfect! Here's the complete consolidated frontend for your MERN Team Tracker application:Complete Frontend Structurefrontend/
├── public/
│   └── index.html
├── src/
│   ├── Components/
│   │   ├── Header.js
│   │   └── Teams.js
│   ├── pages/
│   │   ├── Home.js
│   │   ├── AddMember.js
│   │   ├── MoveMember.js
│   │   └── Login.js
│   ├── helper.js
│   ├── App.js
│   └── index.js
└── package.json1. src/Components/Header.jsimport React from "react";
import { Link, useNavigate } from "react-router-dom";

function Header() {
  const navigate = useNavigate();

  const handleLogout = () => {
    window.localStorage.removeItem("token");
    navigate("/login");
  };

  return (
    <header>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/add-member">Add Member</Link>
        <Link to="/move-member">Move Member</Link>
        <button onClick={handleLogout}>Logout</button>
      </nav>
    </header>
  );
}

export default Header;2. src/Components/Teams.jsimport React from "react";

function Teams({
  data,
  edit,
  editId,
  empId,
  empName,
  experience,
  teamName,
  handleEdit,
  handleCancel,
  handleDelete,
  handleDone,
  handleChange,
}) {
  // Group members by technology
  const groupedData = data.reduce((acc, member) => {
    const tech = member.technology_name;
    if (!acc[tech]) {
      acc[tech] = [];
    }
    acc[tech].push(member);
    return acc;
  }, {});

  return (
    <div className="teams-container">
      {Object.keys(groupedData).map((tech) => (
        <div key={tech} className="team-section">
          <h2>{tech}</h2>
          <table>
            <thead>
              <tr>
                <th>Employee ID</th>
                <th>Employee Name</th>
                <th>Experience</th>
                <th>Actions</th>
              </tr>
            </thead>
            <tbody>
              {groupedData[tech].map((member) => (
                <tr key={member._id}>
                  {edit && editId === member._id ? (
                    <>
                      <td>
                        <input
                          type="number"
                          name="empId"
                          value={empId}
                          onChange={handleChange}
                        />
                      </td>
                      <td>
                        <input
                          type="text"
                          name="empName"
                          value={empName}
                          onChange={handleChange}
                        />
                      </td>
                      <td>
                        <input
                          type="number"
                          name="experience"
                          value={experience}
                          onChange={handleChange}
                        />
                      </td>
                      <td>
                        <button onClick={handleDone}>Done</button>
                        <button onClick={handleCancel}>Cancel</button>
                      </td>
                    </>
                  ) : (
                    <>
                      <td>{member.employee_id}</td>
                      <td>{member.employee_name}</td>
                      <td>{member.experience}</td>
                      <td>
                        <button onClick={() => handleEdit(member._id)}>
                          Edit
                        </button>
                        <button onClick={(e) => handleDelete(e, member._id)}>
                          Delete
                        </button>
                      </td>
                    </>
                  )}
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      ))}
    </div>
  );
}

export default Teams;3. src/pages/Home.jsimport React, { Component } from "react";
import Header from "../Components/Header";
import Teams from "../Components/Teams";

class Home extends Component {
  state = {
    data: [],
    initialData: [],
    team: [],
    edit: false,
    editId: undefined,
    empId: "",
    empName: "",
    experience: "",
    experienceFilter: "",
    checked: "Experience",
    teamName: "",
    token: "",
  };

  componentDidMount() {
    const token = this.getLocalStorage();
    if (!token) {
      window.location.href = "/login";
      return;
    }
    this.setState({ token });
    this.handleGetMembers();
    this.handleGetTech();
  }

  getLocalStorage = () => {
    return window.localStorage.getItem("token");
  };

  handleGetMembers = async (url) => {
    try {
      const endpoint = url || "/api/tracker/members/display";
      const res = await fetch(endpoint, {
        headers: { Authorization: `Bearer ${this.state.token}` },
      });
      const members = await res.json();
      this.setState({ data: members, initialData: members });
    } catch {
      this.setState({ data: [], initialData: [] });
    }
  };

  handleGetTech = async () => {
    try {
      const res = await fetch("/api/tracker/technologies/get", {
        headers: { Authorization: `Bearer ${this.state.token}` },
      });
      const team = await res.json();
      this.setState({ team });
    } catch {
      this.setState({ team: [] });
    }
  };

  handleChange = (e) => {
    const { name, value } = e.target;
    this.setState({ [name]: value });
  };

  handleDeleteMembers = async (id) => {
    try {
      await fetch(`/api/tracker/members/delete/${id}`, {
        method: "DELETE",
        headers: { Authorization: `Bearer ${this.state.token}` },
      });
      this.handleGetMembers();
    } catch {}
  };

  handleDelete = async (e, id) => {
    await this.handleDeleteMembers(id);
  };

  handleEdit = (id) => {
    const member = this.state.data.find((m) => m._id === id);
    if (member) {
      this.setState({
        edit: true,
        editId: id,
        empId: member.employee_id,
        empName: member.employee_name,
        experience: member.experience,
        teamName: member.technology_name,
      });
    }
  };

  handleChecked = (value) => {
    this.setState({
      checked: value,
      teamName: "",
      experienceFilter: "",
    });
  };

  handleClear = async () => {
    await this.handleGetMembers();
    this.setState({
      checked: "Experience",
      experienceFilter: "",
      teamName: "",
      empId: "",
      empName: "",
      experience: "",
      edit: false,
      editId: undefined,
    });
  };

  handleGo = async () => {
    const { checked, teamName, experienceFilter } = this.state;
    let url = "/api/tracker/members/display";
    let params = [];

    if (checked === "Team" && teamName) {
      params.push(`tech=${teamName}`);
    }
    if (checked === "Experience" && experienceFilter) {
      params.push(`experience=${experienceFilter}`);
    }
    if (checked === "Both") {
      if (teamName) params.push(`tech=${teamName}`);
      if (experienceFilter) params.push(`experience=${experienceFilter}`);
    }

    if (params.length > 0) {
      url += "?" + params.join("&");
    }

    await this.handleGetMembers(url);
  };

  handleCancel = () => {
    this.setState({
      edit: false,
      editId: undefined,
      empId: "",
      empName: "",
      experience: "",
      teamName: "",
    });
  };

  handleEditRequest = async () => {
    const { editId, empId, empName, experience, teamName, token } = this.state;
    try {
      await fetch(`/api/tracker/members/update/${editId}`, {
        method: "PATCH",
        headers: {
          Authorization: `Bearer ${token}`,
          "Content-type": "application/json",
        },
        body: JSON.stringify({
          employee_id: empId,
          employee_name: empName,
          experience,
          technology_name: teamName,
        }),
      });
      this.setState({
        edit: false,
        editId: undefined,
        empId: "",
        empName: "",
        experience: "",
        teamName: "",
      });
      this.handleGetMembers();
    } catch {}
  };

  handleDone = async (e) => {
    await this.handleEditRequest();
  };

  render() {
    const {
      data,
      team,
      checked,
      experienceFilter,
      teamName,
      edit,
      editId,
      empId,
      empName,
      experience,
    } = this.state;

    return (
      <>
        <Header />
        <section>
          <label>Filter By:</label>
          <input
            type="radio"
            name="checked"
            value="Experience"
            checked={checked === "Experience"}
            onChange={(e) => this.handleChecked(e.target.value)}
          />
          Experience
          <input
            type="radio"
            name="checked"
            value="Team"
            checked={checked === "Team"}
            onChange={(e) => this.handleChecked(e.target.value)}
          />
          Team
          <input
            type="radio"
            name="checked"
            value="Both"
            checked={checked === "Both"}
            onChange={(e) => this.handleChecked(e.target.value)}
          />
          Both
          <select
            name="teamName"
            value={teamName}
            onChange={this.handleChange}
            disabled={checked === "Experience"}
          >
            <option value="">Select Team</option>
            {team.map((t) => (
              <option key={t.name} value={t.name}>
                {t.name}
              </option>
            ))}
          </select>
          <input
            type="number"
            name="experienceFilter"
            value={experienceFilter}
            onChange={this.handleChange}
            disabled={checked === "Team"}
            placeholder="Experience"
          />
          <button onClick={this.handleGo}>Go</button>
          <button onClick={this.handleClear}>Clear</button>
        </section>
        <section>
          {data.length === 0 && <div className="noTeam">No Teams Found</div>}
          <Teams
            data={data}
            edit={edit}
            editId={editId}
            empId={empId}
            empName={empName}
            experience={experience}
            teamName={teamName}
            handleEdit={this.handleEdit}
            handleCancel={this.handleCancel}
            handleDelete={this.handleDelete}
            handleDone={this.handleDone}
            handleChange={this.handleChange}
          />
        </section>
      </>
    );
  }
}

export default Home;4. src/pages/AddMember.jsimport React, { Component } from "react";
import Header from "../Components/Header";
import { validateEmployeeId, validateEmpName, validateExperience } from "../helper";

class AddMember extends Component {
  state = {
    empId: "",
    empName: "",
    teamName: "",
    experience: "",
    newTeam: "",
    createTeam: false,
    deleteTeam: false,
    teams: [],
    errorStmEmpId: "",
    errorStmEmpName: "",
    errorStmExperience: "",
    errorNewTeam: "",
    errorDeleteTeam: "",
    token: "",
  };

  componentDidMount() {
    const token = this.getLocalStorage();
    if (!token) {
      window.location.href = "/login";
      return;
    }
    this.setState({ token });
    this.handleGetTeam();
  }

  getLocalStorage = () => window.localStorage.getItem("token");

  handleGetTeam = async () => {
    try {
      const res = await fetch("/api/tracker/technologies/get", {
        headers: { Authorization: `Bearer ${this.state.token}` },
      });
      const teams = await res.json();
      this.setState({ teams });
    } catch {
      this.setState({ teams: [] });
    }
  };

  handleChange = (e) => {
    const { name, value } = e.target;
    let errEmpId = this.state.errorStmEmpId;
    let errEmpName = this.state.errorStmEmpName;
    let errExp = this.state.errorStmExperience;
    if (name === "empId") errEmpId = validateEmployeeId(value);
    if (name === "empName") errEmpName = validateEmpName(value);
    if (name === "experience") errExp = validateExperience(value);
    this.setState({
      [name]: value,
      errorStmEmpId: errEmpId,
      errorStmEmpName: errEmpName,
      errorStmExperience: errExp,
    });
  };

  handleClear = () => {
    this.setState({
      empId: "",
      empName: "",
      teamName: "",
      experience: "",
      newTeam: "",
      errorStmEmpId: "",
      errorStmEmpName: "",
      errorStmExperience: "",
      errorNewTeam: "",
      errorDeleteTeam: "",
    });
  };

  handleAddOrDeleteTeam = (e, action) => {
    if (action === "add") this.setState({ createTeam: true, deleteTeam: false, newTeam: "", errorNewTeam: "" });
    if (action === "delete") this.setState({ createTeam: false, deleteTeam: true, errorDeleteTeam: "" });
  };

  handleCancel = (e, action) => {
    if (action === "add") this.setState({ createTeam: false, newTeam: "", errorNewTeam: "" });
    if (action === "delete") this.setState({ deleteTeam: false, errorDeleteTeam: "" });
  };

  handleSave = async (e) => {
    e.preventDefault();
    this.saveTeam();
  };

  saveTeam = async () => {
    const { newTeam, token } = this.state;
    if (!newTeam) {
      this.setState({ errorNewTeam: "*Please enter a value" });
      return;
    }
    try {
      const res = await fetch("/api/tracker/technologies/add", {
        method: "POST",
        headers: { Authorization: `Bearer ${token}`, "Content-type": "application/json" },
        body: JSON.stringify({ name: newTeam }),
      });
      if (res.status === 201) {
        this.setState({ createTeam: false, newTeam: "", errorNewTeam: "" });
        this.handleGetTeam();
      } else {
        const err = await res.json();
        this.setState({ errorNewTeam: err.error || "Team already exists" });
      }
    } catch {
      this.setState({ errorNewTeam: "Error creating team" });
    }
  };

  handleRemoveTeam = async (e, tech) => {
    await this.removeTeamRequest(tech);
  };

  removeTeamRequest = async (name) => {
    const { token } = this.state;
    try {
      const res = await fetch("/api/tracker/technologies/remove", {
        method: "DELETE",
        headers: { Authorization: `Bearer ${token}`, "Content-type": "application/json" },
        body: JSON.stringify({ name }),
      });
      if (res.status === 200) {
        this.handleGetTeam();
        this.setState({ errorDeleteTeam: "" });
      } else {
        this.setState({ errorDeleteTeam: "Couldn't delete the team" });
      }
    } catch {
      this.setState({ errorDeleteTeam: "Couldn't delete the team" });
    }
  };

  handleAddMember = async (e) => {
    e.preventDefault();
    this.AddRequest();
  };

  AddRequest = async () => {
    const { empId, empName, teamName, experience, token } = this.state;
    const idErr = validateEmployeeId(empId);
    const nameErr = validateEmpName(empName);
    const expErr = validateExperience(experience);
    this.setState({
      errorStmEmpId: idErr,
      errorStmEmpName: nameErr,
      errorStmExperience: expErr,
    });
    if (idErr || nameErr || expErr || !teamName) return;
    try {
      const res = await fetch("/api/tracker/members/add", {
        method: "POST",
        headers: { Authorization: `Bearer ${token}`, "Content-type": "application/json" },
        body: JSON.stringify({
          employee_id: Number(empId),
          employee_name: empName,
          technology_name: teamName,
          experience: Number(experience),
        }),
      });
      if (res.status === 201) {
        this.handleClear();
      } else {
        const err = await res.json();
        this.setState({ errorStmEmpId: err.error || "Could not add member" });
      }
    } catch {
      this.setState({ errorStmEmpId: "Could not add member" });
    }
  };

  render() {
    const {
      empId,
      empName,
      teamName,
      experience,
      teams,
      createTeam,
      deleteTeam,
      newTeam,
      errorStmEmpId,
      errorStmEmpName,
      errorStmExperience,
      errorNewTeam,
      errorDeleteTeam,
    } = this.state;

    return (
      <>
        <Header />
        <form onSubmit={this.handleAddMember}>
          <h1>Add Team Member</h1>
          <div>
            <label>Employee ID</label>
            <input type="text" name="empId" value={empId} onChange={this.handleChange} />
            <span>{errorStmEmpId}</span>
          </div>
          <div>
            <label>Employee Name</label>
            <input type="text" name="empName" value={empName} onChange={this.handleChange} />
            <span>{errorStmEmpName}</span>
          </div>
          <div>
            <label>Team</label>
            <select name="teamName" value={teamName} onChange={this.handleChange}>
              <option value="">Select Team</option>
              {teams.map((team) => (
                <option key={team.name} value={team.name}>
                  {team.name}
                </option>
              ))}
            </select>
            <button type="button" onClick={(e) => this.handleAddOrDeleteTeam(e, "add")}>
              +
            </button>
            <button type="button" onClick={(e) => this.handleAddOrDeleteTeam(e, "delete")}>
              Delete
            </button>
            {createTeam && (
              <div className="addList">
                <p>Create New Label</p>
                <input type="text" name="newTeam" value={newTeam} onChange={this.handleChange} />
                <button type="button" onClick={this.handleSave}>
                  Save
                </button>
                <button type="button" onClick={(e) => this.handleCancel(e, "add")}>
                  Cancel
                </button>
                <span>{errorNewTeam}</span>
              </div>
            )}
            {deleteTeam && (
              <div className="addList">
                <p>Delete Team</p>
                <table>
                  <tbody>
                    {teams.map((team) => (
                      <tr key={team.name}>
                        <td>{team.name}</td>
                        <td>
                          <button type="button" onClick={(e) => this.handleRemoveTeam(e, team.name)}>
                            x
                          </button>
                        </td>
                        <td>
                          <button type="button" onClick={(e) => this.handleCancel(e, "delete")}>
                            Cancel
                          </button>
                        </td>
                      </tr>
                    ))}
                  </tbody>
                </table>
                <span>{errorDeleteTeam}</span>
              </div>
            )}
          </div>
          <div>
            <label>Experience</label>
            <input type="number" name="experience" value={experience} onChange={this.handleChange} />
            <span>{errorStmExperience}</span>
          </div>
          <div>
            <button className="button" type="submit">
              Add
            </button>
            <button className="button" type="button" onClick={this.handleClear}>
              Clear
            </button>
          </div>
        </form>
      </>
    );
  }
}

export default AddMember;5. src/pages/MoveMember.jsimport React, { Component } from "react";
import Header from "../Components/Header";

class MoveMember extends Component {
  state = {
    members: [],
    teams: [],
    selectedMember: "",
    newTeam: "",
    token: "",
    error: "",
    success: "",
  };

  componentDidMount() {
    const token = this.getLocalStorage();
    if (!token) {
      window.location.href = "/login";
      return;
    }
    this.setState({ token });
    this.loadData();
  }

  getLocalStorage = () => window.localStorage.getItem("token");

  loadData = async () => {
    try {
      const [membersRes, teamsRes] = await Promise.all([
        fetch("/api/tracker/members/display", {
          headers: { Authorization: `Bearer ${this.state.token}` },
        }),
        fetch("/api/tracker/technologies/get", {
   
