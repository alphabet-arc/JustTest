Login.js - Complete Implementation

import React, { Component } from "react";

class Login extends Component {
  state = {
    name: "",
    password: "",
  };

  componentDidMount() {
    // Clear local storage on page load
    localStorage.clear();
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
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ name, password }),
      });

      const data = await response.json();

      if (response.status === 200) {
        // Store token in localStorage
        localStorage.setItem("token", data.token);
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


Header---------

import React from "react";
import { Link } from "react-router-dom";
import logout from "../icon/logout.png";
import { withRouter } from "react-router-dom";
import home from "../icon/home.png";

export function Header(props) {
  function handleLogout() {
    // Remove token from localStorage
    localStorage.removeItem("token");
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


Home.js --------

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
    const token = localStorage.getItem("token");
    
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
      const token = localStorage.getItem("token");
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
      const token = localStorage.getItem("token");
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
      const token = localStorage.getItem("token");
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
      const token = localStorage.getItem("token");
      const response = await fetch(`/api/tracker/members/update/${editId}`, {
        method: "PATCH",
        headers: {
          "Content-Type": "application/json",
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


Team.js-----------

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


Addmember.js--------
import React, { Component } from "react";
import Header from "../Components/Header";
import remove from "../icon/close.png";

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
    const token = localStorage.getItem("token");
    
    if (!token) {
      this.props.history.push("/login");
      return;
    }

    this.handleGetTeam();
  };

  handleGetTeam = async () => {
    try {
      const token = localStorage.getItem("token");
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
      const token = localStorage.getItem("token");
      const response = await fetch("/api/tracker/members/add", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
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
      const token = localStorage.getItem("token");
      const response = await fetch("/api/tracker/technologies/add", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${token}`,
        },
        body: JSON.stringify({ name: newTeam }),
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
      const token = localStorage.getItem("token");
      const response = await fetch("/api/tracker/technologies/add", {
        method: "DELETE",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${token}`,
        },
        body: JSON.stringify({ name: tech }),
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



moveMember------------

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
    const token = localStorage.getItem("token");
    
    if (!token) {
      this.props.history.push("/login");
      return;
    }

    this.handleGetTeam();
    this.handleGetMembers();
  };

  handleGetTeam = async () => {
    try {
      const token = localStorage.getItem("token");
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
      const token = localStorage.getItem("token");
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
      const token = localStorage.getItem("token");
      const response = await fetch(
        `/api/tracker/members/update/${member._id}`,
        {
          method: "PATCH",
          headers: {
            "Content-Type": "application/json",
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


backend/src/middlewares/auth.js
const jwt = require("jsonwebtoken");

// Authentication middleware to verify JWT token
const auth = (req, res, next) => {
  try {
    // Get token from Authorization header
    const authHeader = req.headers.authorization;
    
    if (!authHeader || !authHeader.startsWith("Bearer ")) {
      return res.status(401).json({ error: "No token provided" });
    }

    const token = authHeader.split(" ")[1];

    // Verify token - use the secret from helper.js
    const decoded = jwt.verify(token, process.env.SECRET_KEY || "your_secret_token");
    
    // Add user data to request object
    req.user = decoded;
    
    // Proceed to next middleware
    next();
  } catch (error) {
    return res.status(401).json({ error: "Invalid token" });
  }
};

module.exports = auth;





Backend/src/routers/team.js ==============



const express = require("express");
const Admin = require("../mongoose/models/admin");
const Teams = require("../mongoose/models/teams");
const Members = require("../mongoose/models/members");
const auth = require("../middlewares/auth");

const membersRouter = express.Router();

// 1) /admin/login -> POST Method
membersRouter.post("/admin/login", async (req, res) => {
  try {
    const { name, password } = req.body;

    // Find admin by name and password
    const admin = await Admin.findOne({ name, password });

    if (!admin) {
      return res.status(400).json({ error: "Username or password is wrong" });
    }

    // Generate JWT token
    const jwt = require("jsonwebtoken");
    const token = jwt.sign(
      { _id: admin._id.toString() },
      process.env.SECRET_KEY || "your_secret_token",
      { expiresIn: "7d" }
    );

    // Add token to admin's tokens array
    admin.tokens = admin.tokens || [];
    admin.tokens.push({ token });
    await admin.save();

    res.status(200).json({ token });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// 2) /tracker/members/add -> POST Method
membersRouter.post("/tracker/members/add", auth, async (req, res) => {
  try {
    const { employee_id, employee_name, technology_name, experience } = req.body;

    // Check if member with same employee_id and technology_name exists
    const existingMember = await Members.findOne({
      employee_id,
      technology_name,
    });

    if (existingMember) {
      return res.status(400).json({ error: "Member with same team already exists" });
    }

    // Check if technology exists in teams collection
    const team = await Teams.findOne({ name: technology_name });
    if (!team) {
      // Add technology to teams collection
      await Teams.create({ name: technology_name });
    }

    // Create new member
    const member = new Members({
      employee_id,
      employee_name,
      technology_name,
      experience,
    });

    await member.save();

    res.status(201).json(member);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// 3) /tracker/technologies/get -> GET Method
membersRouter.get("/tracker/technologies/get", auth, async (req, res) => {
  try {
    const teams = await Teams.find({});
    res.status(200).json(teams);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// 4) /tracker/technologies/add -> POST Method
membersRouter.post("/tracker/technologies/add", auth, async (req, res) => {
  try {
    const { name } = req.body;

    // Check if team already exists
    const existingTeam = await Teams.findOne({ name });
    if (existingTeam) {
      return res.status(400).json({ error: "Team already exist" });
    }

    const team = new Teams({ name });
    await team.save();

    res.status(201).json(team);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// 5) /tracker/technologies/add -> DELETE Method
membersRouter.delete("/tracker/technologies/add", auth, async (req, res) => {
  try {
    const { name } = req.body;

    // Delete the team
    const result = await Teams.findOneAndDelete({ name });

    if (!result) {
      return res.status(404).json({ error: "Team not found" });
    }

    res.status(200).json({ message: "Team deleted successfully" });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// 6) /tracker/members/update/:id -> PATCH Method
membersRouter.patch("/tracker/members/update/:id", auth, async (req, res) => {
  try {
    const { id } = req.params;
    const updates = req.body;

    const member = await Members.findByIdAndUpdate(id, updates, {
      new: true,
      runValidators: true,
    });

    if (!member) {
      return res.status(404).json({ error: "Member not found" });
    }

    res.status(200).json(member);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// 7) /tracker/members/display/tech=&technology_name&experience=&experience -> GET Method
membersRouter.get("/tracker/members/display", auth, async (req, res) => {
  try {
    const { tech, experience } = req.query;

    let query = {};

    // Filter by technology_name if provided
    if (tech) {
      query.technology_name = tech;
    }

    // Filter by experience if provided
    if (experience) {
      query.experience = { $gte: parseInt(experience) };
    }

    const members = await Members.find(query);
    res.status(200).json(members);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// 8) /tracker/members/delete/:id -> DELETE Method
membersRouter.delete("/tracker/members/delete/:id", auth, async (req, res) => {
  try {
    const { id } = req.params;

    const member = await Members.findByIdAndDelete(id);

    if (!member) {
      return res.status(404).json({ error: "Member not found" });
    }

    res.status(200).json({ message: "Member deleted successfully" });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

module.exports = membersRouter;



Backend/src/mongoose/models/admin.js =========

const mongoose = require("mongoose");

const adminSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
  },
  password: {
    type: String,
    required: true,
  },
  tokens: [
    {
      token: {
        type: String,
        required: true,
      },
    },
  ],
});

const Admin = mongoose.model("Admin", adminSchema);

module.exports = Admin;



Backend/src/mongoose/models/team.js ===========

const mongoose = require("mongoose");

const teamsSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    unique: true,
  },
});

const Teams = mongoose.model("Teams", teamsSchema);

module.exports = Teams;





src/app,js ===================



const express = require("express");
const cors = require("cors");
const membersRouter = require("./routers/teams");

const app = express();

// Middleware
app.use(cors());
app.use(express.json());

// Routes
app.use("/api", membersRouter);

module.exports = app;






src/index.js ======================


const app = require("./app");
const mongoose = require("mongoose");

const port = process.env.PORT || 8001;

// MongoDB connection
mongoose.connect("mongodb://localhost:27017/team-tracker", {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  useCreateIndex: true,
  useFindAndModify: false,
})
.then(() => {
  console.log("Connected to MongoDB");
})
.catch((error) => {
  console.error("MongoDB connection error:", error);
});

// Start server
app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});


mongoose/mongoose,.js============
const mongoose = require("mongoose");

mongoose.connect("mongodb://localhost:27017/team-tracker", {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  useCreateIndex: true,
  useFindAndModify: false,
});

module.exports = mongoose;



moongoose/models/default========
const mongoose = require("mongoose");
const Admin = require("./admin");
const Teams = require("./teams");

// Function to set up default database data
const setupDefaultDB = async () => {
  try {
    // Check if admin already exists
    const adminExists = await Admin.findOne({ name: "Admin" });
    
    if (!adminExists) {
      // Create default admin
      const admin = new Admin({
        name: "Admin",
        password: "Fresco@123",
        tokens: [],
      });
      await admin.save();
      console.log("Default admin created");
    }

    // Check if default teams exist
    const teamsCount = await Teams.countDocuments();
    
    if (teamsCount === 0) {
      // Create default teams
      const defaultTeams = ["ReactJS", "NodeJS", "Angular"];
      
      for (const teamName of defaultTeams) {
        const team = new Teams({ name: teamName });
        await team.save();
      }
      console.log("Default teams created");
    }
  } catch (error) {
    console.error("Error setting up default database:", error);
  }
};

module.exports = setupDefaultDB;

