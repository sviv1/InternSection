// MentorDashboard.jsx
```jsx
import { useState, useEffect } from "react";
import { io } from "socket.io-client";
import axios from "axios";

const socket = io("http://localhost:5000");

export default function MentorDashboard() {
  const [mentor, setMentor] = useState(null);
  const [availableSlots, setAvailableSlots] = useState([]);
  const [upcomingSessions, setUpcomingSessions] = useState([]);
  const [pastSessions, setPastSessions] = useState([]);
  const [loading, setLoading] = useState(true);
  const [selectedDay, setSelectedDay] = useState("");
  const [selectedSlot, setSelectedSlot] = useState("");
  
  const days = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"];
  const slots = ["Morning", "Afternoon", "Evening"];

  useEffect(() => {
    axios.get("http://localhost:5000/mentor-profile", { withCredentials: true })
      .then((response) => {
        console.log("Mentor data fetched:", response.data);
        setMentor(response.data.mentor || { name: "John Doe", specialization: "Mathematics" });
        setAvailableSlots(response.data.availableSlots || []);
        setUpcomingSessions(response.data.upcomingSessions || []);
        setPastSessions(response.data.pastSessions || []);
        setLoading(false);
      })
      .catch((error) => {
        console.error("Error fetching mentor data:", error);
        setMentor({ name: "John Doe", specialization: "Mathematics" });
        setAvailableSlots(["Monday - Morning", "Wednesday - Evening"]);
        setUpcomingSessions([]);
        setPastSessions([]);
        setLoading(false);
      });
  }, []);

  useEffect(() => {
    if (!mentor) return;
    
    socket.on("sessionBooked", (session) => {
      if (session.mentorId === mentor.id) {
        setUpcomingSessions((prev) => [...prev, session]);
        setAvailableSlots((prev) => prev.filter((slot) => slot !== session.slot));
      }
    });

    socket.on("sessionCompleted", ({ mentorId, slot }) => {
      if (mentorId === mentor.id) {
        setAvailableSlots((prev) => [...prev, slot]);
      }
    });

    return () => {
      socket.off("sessionBooked");
      socket.off("sessionCompleted");
    };
  }, [mentor]);

  if (loading) {
    return <div className="text-center py-10">Loading...</div>;
  }

  if (!mentor) {
    return <div className="text-center py-10">Failed to load mentor data.</div>;
  }

  const updateAvailability = () => {
    if (!selectedDay || !selectedSlot) return;
    const newSlot = `${selectedDay} - ${selectedSlot}`;
    axios.post("http://localhost:5000/update-availability", { mentorId: mentor.id, slot: newSlot })
      .then(() => setAvailableSlots((prev) => [...prev, newSlot]))
      .catch((error) => console.error("Error updating availability:", error));
  };

  const removeAvailability = (slotToRemove) => {
    axios.post("http://localhost:5000/remove-availability", { mentorId: mentor.id, slot: slotToRemove })
      .then(() => setAvailableSlots((prev) => prev.filter((slot) => slot !== slotToRemove)))
      .catch((error) => console.error("Error removing availability:", error));
  };

  return (
    <div className="max-w-4xl mx-auto p-6 bg-gray-100 min-h-screen">
      <h2 className="text-2xl font-bold mb-4">Mentor Dashboard</h2>
      <div className="bg-white p-4 rounded-lg shadow-md mb-4">
        <h3 className="text-xl font-semibold">Profile</h3>
        <p><strong>Name:</strong> {mentor.name}</p>
        <p><strong>Specialization:</strong> {mentor.specialization}</p>
      </div>

      <div className="bg-white p-4 rounded-lg shadow-md mb-4">
        <h3 className="text-xl font-semibold">Availability</h3>
        <div className="mb-2">
          <select onChange={(e) => setSelectedDay(e.target.value)} className="border p-2 mr-2">
            <option value="">Select Day</option>
            {days.map(day => <option key={day} value={day}>{day}</option>)}
          </select>
          <select onChange={(e) => setSelectedSlot(e.target.value)} className="border p-2 mr-2">
            <option value="">Select Slot</option>
            {slots.map(slot => <option key={slot} value={slot}>{slot}</option>)}
          </select>
          <button className="px-4 py-2 bg-blue-500 text-white rounded" onClick={updateAvailability}>Add Slot</button>
        </div>
        <ul>
          {availableSlots.map((slot, index) => (
            <li key={index} className="flex justify-between items-center border-b py-2">
              {slot}
              <button className="ml-2 text-red-500" onClick={() => removeAvailability(slot)}>Remove</button>
            </li>
          ))}
        </ul>
      </div>

      <div className="bg-white p-4 rounded-lg shadow-md mb-4">
        <h3 className="text-xl font-semibold">Upcoming Classes</h3>
        <ul>
          {upcomingSessions.map((session) => (
            <li key={session.id} className="border-b py-2">{session.date} - {session.time} with {session.studentName}</li>
          ))}
        </ul>
      </div>

      <div className="bg-white p-4 rounded-lg shadow-md">
        <h3 className="text-xl font-semibold">Past Classes</h3>
        <ul>
          {pastSessions.map((session, index) => (
            <li key={index} className="border-b py-2">{session.date} - {session.time} with {session.studentName}</li>
          ))}
        </ul>
      </div>
    </div>
  );
}
```