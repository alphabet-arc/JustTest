// ============================================================================
// Login.js - COMPLETELY FIXED
// ============================================================================

import React, { Component } from "react";

class Login extends Component {
  state = {
    name: "",
    password: "",
  };

  componentDidMount() {
    // Clear local storage on page load
    if (typeof window !== 'undefined' && window.localStorage) {
      localStorage.clear();
    }
  }

  handleChange = (e) => {
    this.setState({
      [e.target.name]: e.target.value,
    });
  };

  loginRequest = async () => {
    const { name, password } = this.state;
    
    try {
      const response = await fetch("/api/admin/login", {
        method: "POST",
        headers: {
          "Content-type": "application/json; charset=UTF-8",
        },
        body: JSON.stringify({ name, password }),
      });

      const data = await response.json();

      if (response.status === 200) {
        // Store token in localStorage
        if (typeof window !== 'undefined' && window.localStorage) {
          localStorage.setItem("token", data.token);
        }
        // Navigate to home page
        this.props.history.push("/");
      } else {
        alert("Invalid Credentials");
      }
    } catch (error) {
      console.error("Login error:", error);
      alert("Invalid Credentials");
    }
  };

  handleLogin = async () => {
    const { name, password } = this.state;

    // Check if both fields have values
    if (name.trim() && password.trim()) {
      await this.loginRequest();
    }
  };

  render() {
    const { name, password } = this.state;
    const isButtonDisabled = !name.trim() || !password.trim();

    return (
      <div className="login">
        <h1>Login</h1>
        <div>
          <input
            className="login input"
            type="text"
            name="name"
            value={name}
            onChange={this.handleChange}
            placeholder="Username"
          />
        </div>
        <div>
          <input
            className="login input"
            type="password"
            name="password"
            value={password}
            onChange={this.handleChange}
            placeholder="Password"
          />
        </div>
        <button
          className="login button"
          onClick={this.handleLogin}
          disabled={isButtonDisabled}
        >
          Login
        </button>
      </div>
    );
  }
}

export default Login;


// ============================================================================
// Header.js - COMPLETELY FIXED
// ============================================================================

import React from "react";
import { Link } from "react-router-dom";
import logout from "../icon/logout.png";
import { withRouter } from "react-router-dom";
import home from "../icon/home.png";

export function Header(props) {
  function handleLogout() {
    // Remove token from localStorage
    if (typeof window !== 'undefined' && window.localStorage) {
      localStorage.removeItem("token");
    }
    // Navigate to login page
    props.history.push("/login");
  }

  function handleHome() {
    // Navigate to home page
    props.history.push("/");
  }

  return (
    <>
      <nav>
        <Link to="/addMember">Add Member</Link>
        <Link to="/moveMember">Move Member</Link>
      </nav>
      <header>
        <h1>Team Tracker</h1>
        <img src={logout} alt="logout" onClick={handleLogout} />
        <img src={home} alt="home" onClick={handleHome} />
      </header>
    </>
  );
}

export default withRouter(Header);


// ============================================================================
// Teams.js - COMPLETELY FIXED
// ============================================================================

import React from "react";

export function Teams(props) {
  const {
    data,
    edit,
    editId,
    empId,
    empName,
    experience,
    handleCancel,
    handleDone,
    handleEdit,
    handleDelete,
    handleChange,
  } = props;

  if (!data || data.length === 0) {
    return null;
  }

  const teamName = data[0].technology_name;

  return (
    <>
      <h2>{teamName}</h2>
      <table>
        <thead>
          <tr>
            <th>#</th>
            <th>Employee ID</th>
            <th>Name</th>
            <th>Experience</th>
            <th>Edit</th>
            <th>Delete</th>
          </tr>
        </thead>
        <tbody>
          {data.map((member, index) => {
            const isEditMode = edit && editId === member._id;

            return (
              <tr key={member._id}>
                <td>{index + 1}</td>
                <td>
                  {isEditMode ? (
                    <input
                      type="number"
                      name="empId"
                      value={empId}
                      onChange={handleChange}
                    />
                  ) : (
                    member.employee_id
                  )}
                </td>
                <td>
                  {isEditMode ? (
                    <input
                      type="text"
                      name="empName"
                      value={empName}
                      onChange={handleChange}
                    />
                  ) : (
                    member.employee_name
                  )}
                </td>
                <td>
                  {isEditMode ? (
                    <input
                      type="number"
                      name="experience"
                      value={experience}
                      onChange={handleChange}
                    />
                  ) : (
                    member.experience
                  )}
                </td>
                <td>
                  {isEditMode ? (
                    <>
                      <button onClick={handleDone}>Done</button>
                      <button onClick={handleCancel}>Cancel</button>
                    </>
                  ) : (
                    <button onClick={() => handleEdit(member._id)}>
                      Edit
                    </button>
                  )}
                </td>
                <td>
                  <button onClick={(e) => handleDelete(e, member._id)}>
                    Delete
                  </button>
                </td>
              </tr>
            );
          })}
        </tbody>
      </table>
    </>
  );
}

export default Teams;


// ============================================================================
// Home.js - COMPLETELY FIXED
// ============================================================================

import React, { Component } from "react";
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
  };

  componentDidMount() {
    this.getLocalStorage();
  }

  getLocalStorage = () => {
    let token = null;
    if (typeof window !== 'undefined' && window.localStorage) {
      token = localStorage.getItem("token");
    }
    
    if (!token) {
      // Redirect to login if no token
      this.props.history.push("/login");
      return;
    }

    // Fetch initial data
    this.handleGetMembers();
    this.handleGetTech();
  };

  handleGetMembers = async (url = "/api/tracker/members/display") => {
    try {
      let token = null;
      if (typeof window !== 'undefined' && window.localStorage) {
        token = localStorage.getItem("token");
      }
      
      const response = await fetch(url, {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });

      if (response.status === 200) {
        const members = await response.json();
        this.setState({
          initialData: members,
          data: members,
        });
      }
    } catch (error) {
      console.error("Error fetching members:", error);
    }
  };

  handleGetTech = async () => {
    try {
      let token = null;
      if (typeof window !== 'undefined' && window.localStorage) {
        token = localStorage.getItem("token");
      }
      
      const response = await fetch("/api/tracker/technologies/get", {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });

      if (response.status === 200) {
        const teams = await response.json();
        this.setState({ team: teams });
      }
    } catch (error) {
      console.error("Error fetching technologies:", error);
    }
  };

  handleChange = (e) => {
    this.setState({
      [e.target.name]: e.target.value,
    });
  };

  handleDeleteMembers = async (id) => {
    try {
      let token = null;
      if (typeof window !== 'undefined' && window.localStorage) {
        token = localStorage.getItem("token");
      }
      
      const response = await fetch(`/api/tracker/members/delete/${id}`, {
        method: "DELETE",
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });

      if (response.status === 200) {
        // Refresh the members list
        this.handleGetMembers();
      }
    } catch (error) {
      console.error("Error deleting member:", error);
    }
  };

  handleDelete = async (e, id) => {
    e.preventDefault();
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
      });
    }
  };

  handleChecked = (value) => {
    this.setState({ checked: value });
  };

  handleClear = async () => {
    // Reset filters and show all data
    this.setState({
      experienceFilter: "",
      checked: "Experience",
      data: this.state.initialData,
    });
  };

  handleGo = async () => {
    const { checked, experienceFilter } = this.state;
    const { team } = this.state;

    if (checked === "Experience" && experienceFilter) {
      // Filter by experience
      const url = `/api/tracker/members/display?experience=${experienceFilter}`;
      await this.handleGetMembers(url);
    } else if (checked === "Team") {
      // Filter by team
      const selectedTeam = team.find((t) => t.name === experienceFilter);
      if (selectedTeam) {
        const url = `/api/tracker/members/display?tech=${selectedTeam.name}`;
        await this.handleGetMembers(url);
      }
    } else if (checked === "Both" && experienceFilter) {
      // Filter by both
      const selectedTeam = team.find((t) => t.name === experienceFilter);
      if (selectedTeam) {
        const expValue = this.state.experience || experienceFilter;
        const url = `/api/tracker/members/display?tech=${selectedTeam.name}&experience=${expValue}`;
        await this.handleGetMembers(url);
      }
    }
  };

  handleCancel = () => {
    this.setState({
      edit: false,
      editId: undefined,
      empId: "",
      empName: "",
      experience: "",
    });
  };

  handleEditRequest = async () => {
    const { editId, empId, empName, experience } = this.state;
    
    try {
      let token = null;
      if (typeof window !== 'undefined' && window.localStorage) {
        token = localStorage.getItem("token");
      }
      
      const response = await fetch(`/api/tracker/members/update/${editId}`, {
        method: "PATCH",
        headers: {
          "Content-type": "application/json; charset=UTF-8",
          Authorization: `Bearer ${token}`,
        },
        body: JSON.stringify({
          employee_id: empId,
          employee_name: empName,
          experience: experience,
        }),
      });

      if (response.status === 200) {
        // Refresh and reset
        await this.handleGetMembers();
        this.handleCancel();
      }
    } catch (error) {
      console.error("Error updating member:", error);
    }
  };

  handleDone = async (e) => {
    e.preventDefault();
    await this.handleEditRequest();
  };

  render() {
    const { data, team, edit, checked, experienceFilter } = this.state;
    const isGoButtonDisabled = !experienceFilter;

    // Group data by technology_name
    const groupedData = {};
    data.forEach((member) => {
      const tech = member.technology_name;
      if (!groupedData[tech]) {
        groupedData[tech] = [];
      }
      groupedData[tech].push(member);
    });

    return (
      <>
        <Header />
        <section>
          <label>Filter By</label>
          <input
            type="radio"
            name="filter"
            value="Experience"
            checked={checked === "Experience"}
            onChange={() => this.handleChecked("Experience")}
          />
          <label>Experience</label>
          <input
            type="radio"
            name="filter"
            value="Team"
            checked={checked === "Team"}
            onChange={() => this.handleChecked("Team")}
          />
          <label>Team</label>
          <input
            type="radio"
            name="filter"
            value="Both"
            checked={checked === "Both"}
            onChange={() => this.handleChecked("Both")}
          />
          <label>Both</label>
          
          {checked === "Experience" && (
            <input
              type="number"
              name="experienceFilter"
              value={experienceFilter}
              onChange={this.handleChange}
              placeholder="Experience"
            />
          )}
          
          {checked === "Team" && (
            <select
              name="experienceFilter"
              value={experienceFilter}
              onChange={this.handleChange}
            >
              <option value="">--Select Team--</option>
              {team.map((t) => (
                <option key={t._id} value={t.name}>
                  {t.name}
                </option>
              ))}
            </select>
          )}
          
          {checked === "Both" && (
            <>
              <select
                name="experienceFilter"
                value={experienceFilter}
                onChange={this.handleChange}
              >
                <option value="">--Select Team--</option>
                {team.map((t) => (
                  <option key={t._id} value={t.name}>
                    {t.name}
                  </option>
                ))}
              </select>
              <input
                type="number"
                name="experience"
                value={this.state.experience}
                onChange={this.handleChange}
                placeholder="Experience"
              />
            </>
          )}
          
          <button onClick={this.handleGo} disabled={isGoButtonDisabled}>
            Go
          </button>
          <button onClick={this.handleClear}>Clear</button>
        </section>

        {Object.keys(groupedData).length === 0 ? (
          <div className="noTeam">No Teams Found</div>
        ) : (
          Object.keys(groupedData).map((techName) => (
            <Teams
              key={techName}
              data={groupedData[techName]}
              edit={edit}
              editId={this.state.editId}
              empId={this.state.empId}
              empName={this.state.empName}
              experience={this.state.experience}
              handleCancel={this.handleCancel}
              handleDone={this.handleDone}
              handleEdit={this.handleEdit}
              handleDelete={this.handleDelete}
              handleChange={this.handleChange}
            />
          ))
        )}
      </>
    );
  }
}

export default Home;


// ============================================================================
// AddMember.js - COMPLETELY FIXED
// ============================================================================

import React, { Component } from "react";
import Header from "../Components/Header";

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
    errorStmtEmpId: "",
    errorStmtEmpName: "",
    errorStmtExperience: "",
  };

  componentDidMount() {
    this.getLocalStorage();
  }

  getLocalStorage = () => {
    let token = null;
    if (typeof window !== 'undefined' && window.localStorage) {
      token = localStorage.getItem("token");
    }
    
    if (!token) {
      this.props.history.push("/login");
      return;
    }

    this.handleGetTeam();
  };

  handleGetTeam = async () => {
    try {
      let token = null;
      if (typeof window !== 'undefined' && window.localStorage) {
        token = localStorage.getItem("token");
      }
      
      const response = await fetch("/api/tracker/technologies/get", {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });

      if (response.status === 200) {
        const teams = await response.json();
        this.setState({ teams });
      }
    } catch (error) {
      console.error("Error fetching teams:", error);
    }
  };

  handleChange = (e) => {
    const { name, value } = e.target;
    this.setState({ [name]: value }, () => {
      this.validateField(name, value);
    });
  };

  validateField = (fieldName, value) => {
    switch (fieldName) {
      case "empId":
        if (!value) {
          this.setState({ errorStmtEmpId: "Please enter a value" });
        } else if (value < 100000 || value > 3000000) {
          this.setState({
            errorStmtEmpId: "Employee ID is expected between 100000 and 3000000",
          });
        } else {
          this.setState({ errorStmtEmpId: "" });
        }
        break;

      case "empName":
        if (!value) {
          this.setState({ errorStmtEmpName: "Please enter a value" });
        } else if (!/^[a-zA-Z\s]+$/.test(value)) {
          this.setState({
            errorStmtEmpName: "Employee name can have only alphabets and spaces",
          });
        } else if (value.trim().length < 3) {
          this.setState({
            errorStmtEmpName: "Employee Name should have at least 3 letters",
          });
        } else {
          this.setState({ errorStmtEmpName: "" });
        }
        break;

      case "experience":
        if (!value && value !== 0) {
          this.setState({ errorStmtExperience: "Please enter a value" });
        } else {
          this.setState({ errorStmtExperience: "" });
        }
        break;

      default:
        break;
    }
  };

  handleAddMember = async (e) => {
    e.preventDefault();
    const { empId, empName, teamName, experience } = this.state;

    // Validate all fields
    this.validateField("empId", empId);
    this.validateField("empName", empName);
    this.validateField("experience", experience);

    // Check if there are any errors
    if (
      !empId ||
      !empName ||
      !teamName ||
      experience === "" ||
      this.state.errorStmtEmpId ||
      this.state.errorStmtEmpName ||
      this.state.errorStmtExperience
    ) {
      return;
    }

    await this.AddRequest();
  };

  AddRequest = async () => {
    const { empId, empName, teamName, experience } = this.state;

    try {
      let token = null;
      if (typeof window !== 'undefined' && window.localStorage) {
        token = localStorage.getItem("token");
      }
      
      const response = await fetch("/api/tracker/members/add", {
        method: "POST",
        headers: {
          "Content-type": "application/json; charset=UTF-8",
          Authorization: `Bearer ${token}`,
        },
        body: JSON.stringify({
          employee_id: parseInt(empId),
          employee_name: empName,
          technology_name: teamName,
          experience: parseInt(experience),
        }),
      });

      if (response.status === 201) {
        alert("Member added successfully");
        // Clear form
        this.setState({
          empId: "",
          empName: "",
          teamName: "",
          experience: "",
          errorStmtEmpId: "",
          errorStmtEmpName: "",
          errorStmtExperience: "",
        });
      } else {
        const data = await response.json();
        alert(data.error || "Member with same team already exists");
      }
    } catch (error) {
      console.error("Error adding member:", error);
      alert("Error adding member");
    }
  };

  handleClear = () => {
    this.setState({
      empId: "",
      empName: "",
      teamName: "",
      experience: "",
      errorStmtEmpId: "",
      errorStmtEmpName: "",
      errorStmtExperience: "",
    });
  };

  handleAddOrDeleteTeam = (e, action) => {
    if (action === "add") {
      this.setState({ createTeam: !this.state.createTeam, deleteTeam: false });
    } else {
      this.setState({ deleteTeam: !this.state.deleteTeam, createTeam: false });
    }
  };

  handleCancel = (e, action) => {
    if (action === "add") {
      this.setState({ createTeam: false, newTeam: "" });
    } else {
      this.setState({ deleteTeam: false });
    }
  };

  handleSave = async (e) => {
    e.preventDefault();
    await this.saveTeam();
  };

  saveTeam = async () => {
    const { newTeam } = this.state;

    if (!newTeam.trim()) {
      return;
    }

    try {
      let token = null;
      if (typeof window !== 'undefined' && window.localStorage) {
        token = localStorage.getItem("token");
      }
      
      const response = await fetch("/api/tracker/technologies/add", {
        method: "POST",
        headers: {
          "Content-type": "application/json; charset=UTF-8",
          Authorization: `Bearer ${token}`,
        },
        body: JSON.stringify({ technology_name: newTeam }),
      });

      if (response.status === 201) {
        await this.handleGetTeam();
        this.setState({ createTeam: false, newTeam: "" });
      } else {
        const data = await response.json();
        alert(data.error || "Team already exist");
      }
    } catch (error) {
      console.error("Error saving team:", error);
    }
  };

  handleRemoveTeam = async (e, tech) => {
    e.preventDefault();
    await this.removeTeamRequest(tech);
  };

  removeTeamRequest = async (tech) => {
    try {
      let token = null;
      if (typeof window !== 'undefined' && window.localStorage) {
        token = localStorage.getItem("token");
      }
      
      const response = await fetch(`/api/tracker/technologies/remove/${tech}`, {
        method: "DELETE",
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });

      if (response.status === 200) {
        await this.handleGetTeam();
      } else {
        alert("Couldn't delete the team");
      }
    } catch (error) {
      console.error("Error deleting team:", error);
      alert("Couldn't delete the team");
    }
  };

  render() {
    const {
      empId,
      empName,
      teamName,
      experience,
      createTeam,
      deleteTeam,
      teams,
      newTeam,
      errorStmtEmpId,
      errorStmtEmpName,
      errorStmtExperience,
    } = this.state;

    const isAddButtonDisabled =
      !empId ||
      !empName ||
      !teamName ||
      experience === "" ||
      !!errorStmtEmpId ||
      !!errorStmtEmpName ||
      !!errorStmtExperience;

    return (
      <>
        <Header />
        <form>
          <h1>Add Team Member</h1>
          <div>
            <label>Employee ID</label>
            <input
              type="number"
              name="empId"
              value={empId}
              onChange={this.handleChange}
            />
            {errorStmtEmpId && (
              <span className="error">{errorStmtEmpId}</span>
            )}
          </div>

          <div>
            <label>Employee Name</label>
            <input
              type="text"
              name="empName"
              value={empName}
              onChange={this.handleChange}
            />
            {errorStmtEmpName && (
              <span className="error">{errorStmtEmpName}</span>
            )}
          </div>

          <div>
            <label>Team</label>
            <select name="teamName" value={teamName} onChange={this.handleChange}>
              <option value="">--Select Team--</option>
              {teams.map((team) => (
                <option key={team._id} value={team.name}>
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
          </div>

          {createTeam && (
            <div className="addList">
              <p>Create New Label</p>
              <input
                type="text"
                name="newTeam"
                value={newTeam}
                onChange={this.handleChange}
              />
              <button onClick={this.handleSave}>Save</button>
              <button onClick={(e) => this.handleCancel(e, "add")}>
                Cancel
              </button>
            </div>
          )}

          {deleteTeam && (
            <div className="addList">
              <p>Delete Team</p>
              <table>
                <tbody>
                  {teams.map((team) => (
                    <tr key={team._id}>
                      <td>{team.name}</td>
                      <td>
                        <button onClick={(e) => this.handleRemoveTeam(e, team.name)}>
                          x
                        </button>
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
              <button onClick={(e) => this.handleCancel(e, "delete")}>
                Cancel
              </button>
            </div>
          )}

          <div>
            <label>Experience</label>
            <input
              type="number"
              name="experience"
              value={experience}
              onChange={this.handleChange}
            />
            {errorStmtExperience && (
              <span className="error">{errorStmtExperience}</span>
            )}
          </div>

          <div>
            <button onClick={this.handleAddMember} disabled={isAddButtonDisabled}>
              Add
            </button>
            <button type="button" onClick={this.handleClear}>
              Clear
            </button>
          </div>
        </form>
      </>
    );
  }
}

export default AddMember;


// ============================================================================
// MoveMember.js - COMPLETELY FIXED
// ============================================================================

import React, { Component } from "react";
import Header from "../Components/Header";

class MoveMember extends Component {
  state = {
    teams: [],
    data: [],
    empId: "",
    errorStmtEmpId: "",
  };

  componentDidMount() {
    this.getLocalStorage();
  }

  getLocalStorage = () => {
    let token = null;
    if (typeof window !== 'undefined' && window.localStorage) {
      token = localStorage.getItem("token");
    }
    
    if (!token) {
      this.props.history.push("/login");
      return;
    }

    this.handleGetTeam();
    this.handleGetMembers();
  };

  handleGetTeam = async () => {
    try {
      let token = null;
      if (typeof window !== 'undefined' && window.localStorage) {
        token = localStorage.getItem("token");
      }
      
      const response = await fetch("/api/tracker/technologies/get", {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });

      if (response.status === 200) {
        const teams = await response.json();
        this.setState({ teams });
      }
    } catch (error) {
      console.error("Error fetching teams:", error);
    }
  };

  handleGetMembers = async () => {
    try {
      let token = null;
      if (typeof window !== 'undefined' && window.localStorage) {
        token = localStorage.getItem("token");
      }
      
      const response = await fetch("/api/tracker/members/display", {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });

      if (response.status === 200) {
        const members = await response.json();
        this.setState({ data: members });
      }
    } catch (error) {
      console.error("Error fetching members:", error);
    }
  };

  handleChange = (e) => {
    const { name, value } = e.target;
    this.setState({ [name]: value }, () => {
      if (name === "empId") {
        this.validateEmpId(value);
      }
    });
  };

  validateEmpId = (value) => {
    if (!value) {
      this.setState({ errorStmtEmpId: "Please enter a value" });
    } else if (value < 100000 || value > 3000000) {
      this.setState({
        errorStmtEmpId: "Employee ID is expected between 100000 and 3000000",
      });
    } else {
      this.setState({ errorStmtEmpId: "" });
    }
  };

  handleClear = () => {
    this.setState({
      empId: "",
      errorStmtEmpId: "",
    });
  };

  MoveRequest = async (id) => {
    const member = this.state.data.find((m) => m.employee_id === parseInt(id));
    
    if (!member) {
      alert("Member not found");
      return;
    }

    const fromTeam = member.technology_name;
    const toTeam = prompt(`Move from ${fromTeam} to which team?`);

    if (!toTeam) {
      return;
    }

    // Check if the team exists
    const teamExists = this.state.teams.some((t) => t.name === toTeam);
    if (!teamExists) {
      alert("Team does not exist");
      return;
    }

    try {
      let token = null;
      if (typeof window !== 'undefined' && window.localStorage) {
        token = localStorage.getItem("token");
      }
      
      const response = await fetch(
        `/api/tracker/members/update/${member._id}`,
        {
          method: "PATCH",
          headers: {
            "Content-type": "application/json; charset=UTF-8",
            Authorization: `Bearer ${token}`,
          },
          body: JSON.stringify({
            technology_name: toTeam,
          }),
        }
      );

      if (response.status === 200) {
        alert("Member moved successfully");
        await this.handleGetMembers();
        this.setState({ empId: "", errorStmtEmpId: "" });
      } else {
        alert("Error moving member");
      }
    } catch (error) {
      console.error("Error moving member:", error);
      alert("Error moving member");
    }
  };

  handleMove = async (e) => {
    e.preventDefault();
    const { empId, errorStmtEmpId } = this.state;

    this.validateEmpId(empId);

    if (!empId || errorStmtEmpId) {
      return;
    }

    await this.MoveRequest(empId);
  };

  render() {
    const { empId, teams, errorStmtEmpId } = this.state;
    const isMoveButtonDisabled = !empId || !!errorStmtEmpId;

    return (
      <>
        <Header />
        <form>
          <h1>Move Team Member</h1>
          <div>
            <label>Employee ID</label>
            <input
              type="number"
              name="empId"
              value={empId}
              onChange={this.handleChange}
            />
            {errorStmtEmpId && (
              <span className="error">{errorStmtEmpId}</span>
            )}
          </div>

          <div>
            <label>From</label>
            <select disabled>
              <option>--Select Team--</option>
              {teams.map((team) => (
                <option key={team._id} value={team.name}>
                  {team.name}
                </option>
              ))}
            </select>
          </div>

          <div>
            <label>To</label>
            <select disabled>
              <option>--Select Team--</option>
              {teams.map((team) => (
                <option key={team._id} value={team.name}>
                  {team.name}
                </option>
              ))}
            </select>
          </div>

          <div>
            <button onClick={this.handleMove} disabled={isMoveButtonDisabled}>
              Move
            </button>
            <button type="button" onClick={this.handleClear}>
              Clear
            </button>
          </div>
        </form>
      </>
    );
  }
}

export default MoveMember;
      
