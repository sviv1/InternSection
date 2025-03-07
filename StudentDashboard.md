```jsx
import { useState, useEffect } from "react";
import axios from "axios";
import { io } from "socket.io-client";

const socket = io("http://localhost:5000"); // Replace with your backend WebSocket URL

export default function MentorDashboard() {
  const [mentor, setMentor] = useState({
    name: "Dr. Ramesh Verma",
    image: "https://randomuser.me/api/portraits/men/2.jpg",
    subjects: ["Mathematics", "Physics"],
    availability: [],
  });
  
  const [newAvailability, setNewAvailability] = useState({ day: "", slot: "" });
  const [availabilityList, setAvailabilityList] = useState([]);
  
  useEffect(() => {
    axios.get("http://localhost:5000/mentor/availability") // Replace with actual API endpoint
      .then((response) => {
        setAvailabilityList(response.data.availability);
      })
      .catch((error) => console.error("Error fetching availability:", error));

    socket.on("availabilityUpdated", (updatedAvailability) => {
      setAvailabilityList(updatedAvailability);
    });

    return () => {
      socket.off("availabilityUpdated");
    };
  }, []);

  const handleAddAvailability = () => {
    if (newAvailability.day && newAvailability.slot) {
      axios.post("http://localhost:5000/mentor/availability", newAvailability) // API call
        .then(() => {
          socket.emit("updateAvailability");
          setNewAvailability({ day: "", slot: "" });
        })
        .catch((error) => console.error("Error adding availability:", error));
    }
  };

  const handleRemoveAvailability = (index) => {
    const slotToRemove = availabilityList[index];
    axios.delete("http://localhost:5000/mentor/availability", { data: slotToRemove })
      .then(() => {
        socket.emit("updateAvailability");
      })
      .catch((error) => console.error("Error removing availability:", error));
  };

  const [pastClasses, setPastClasses] = useState([]);
  const [upcomingClasses, setUpcomingClasses] = useState([]);
  const [timers, setTimers] = useState({});

  useEffect(() => {
    axios.get("http://localhost:5000/mentor/classes") // API call
      .then((response) => {
        setPastClasses(response.data.pastClasses);
        setUpcomingClasses(response.data.upcomingClasses);
      })
      .catch((error) => console.error("Error fetching classes:", error));
  }, []);

  useEffect(() => {
    const interval = setInterval(() => {
      const updatedTimers = {};
      upcomingClasses.forEach((cls) => {
        const timeLeft = new Date(cls.date) - new Date();
        updatedTimers[cls.id] = timeLeft > 0 ? timeLeft : 0;
      });
      setTimers(updatedTimers);
    }, 1000);
    return () => clearInterval(interval);
  }, [upcomingClasses]);

  return (
    <div className="max-w-4xl mx-auto p-6 bg-gray-100 min-h-screen">
      <div className="bg-white shadow-md rounded-lg p-6 flex items-center gap-4 mb-6">
        <img
          src={mentor.image}
          alt="Mentor"
          className="w-20 h-20 rounded-full border-4 border-blue-500"
        />
        <div>
          <h2 className="text-2xl font-bold">{mentor.name}</h2>
          <p className="text-gray-600">Subjects: {mentor.subjects.join(", ")}</p>
        </div>
      </div>

      <h2 className="text-xl font-semibold mb-4">Manage Availability</h2>
      <div className="flex gap-4 mb-4">
        <select
          value={newAvailability.day}
          onChange={(e) => setNewAvailability({ ...newAvailability, day: e.target.value })}
          className="p-2 border rounded w-full"
        >
          <option value="">Select Day</option>
          {['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'].map(day => (
            <option key={day} value={day}>{day}</option>
          ))}
        </select>
        <select
          value={newAvailability.slot}
          onChange={(e) => setNewAvailability({ ...newAvailability, slot: e.target.value })}
          className="p-2 border rounded w-full"
        >
          <option value="">Select Slot</option>
          {['Morning', 'Afternoon', 'Evening'].map(slot => (
            <option key={slot} value={slot}>{slot}</option>
          ))}
        </select>
        <button
          onClick={handleAddAvailability}
          className="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600"
        >
          Add
        </button>
      </div>

      <h3 className="text-lg font-semibold mb-2">Your Availability:</h3>
      <ul className="bg-white p-4 rounded-lg shadow-md">
        {availabilityList.length > 0 ? (
          availabilityList.map((slot, index) => (
            <li key={index} className="p-2 border-b last:border-b-0 flex justify-between">
              {slot.day} - {slot.slot}
              <button onClick={() => handleRemoveAvailability(index)} className="text-red-500">Remove</button>
            </li>
          ))
        ) : (
          <p className="text-gray-500">No availability set.</p>
        )}
      </ul>

      <h2 className="text-xl font-semibold mt-6 mb-4">Past Classes</h2>
      <ul className="bg-white p-4 rounded-lg shadow-md">
        {pastClasses.map(cls => (
          <li key={cls.id} className="p-2 border-b last:border-b-0">
            {cls.student} - {cls.subject} - {cls.date}
          </li>
        ))}
      </ul>

      <h2 className="text-xl font-semibold mt-6 mb-4">Upcoming Classes</h2>
      <ul className="bg-white p-4 rounded-lg shadow-md">
        {upcomingClasses.map(cls => (
          <li key={cls.id} className="p-2 border-b last:border-b-0">
            {cls.student} - {cls.subject} - {new Date(cls.date).toLocaleString()} -
            <span className="text-green-600 font-semibold">
              {timers[cls.id] ? `${Math.floor(timers[cls.id] / 60000)} min left` : "Class started"}
            </span>
          </li>
        ))}
      </ul>
    </div>
  );
}
```