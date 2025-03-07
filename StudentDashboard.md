// StudentDashboard.jsx
```jsx
import { useState, useEffect } from "react";
import { io } from "socket.io-client";
import axios from "axios";

const socket = io("http://localhost:5000");

export default function StudentDashboard() {
  const [mentors, setMentors] = useState([]);
  const [loading, setLoading] = useState(true);
  const [search, setSearch] = useState("");
  const [selectedSpecialization, setSelectedSpecialization] = useState("");
  const [selectedSlot, setSelectedSlot] = useState("");

  const specializations = ["Mathematics", "Science", "English", "Programming"];
  const slots = ["Morning", "Afternoon", "Evening"];

  useEffect(() => {
    axios.get("http://localhost:5000/mentors")
      .then((response) => {
        setMentors(response.data || []);
        setLoading(false);
      })
      .catch((error) => {
        console.error("Error fetching mentors:", error);
        setLoading(false);
      });
  }, []);

  useEffect(() => {
    socket.on("sessionBooked", ({ mentorId, slot }) => {
      setMentors((prev) =>
        prev.map((mentor) =>
          mentor.id === mentorId
            ? { ...mentor, availableSlots: mentor.availableSlots.filter((s) => s !== slot) }
            : mentor
        )
      );
    });

    return () => {
      socket.off("sessionBooked");
    };
  }, []);

  const bookSession = (mentorId, slot) => {
    axios.post("http://localhost:5000/book-session", { mentorId, slot })
      .then(() => {
        socket.emit("bookSession", { mentorId, slot });
      })
      .catch((error) => console.error("Error booking session:", error));
  };

  const filteredMentors = mentors.filter((mentor) =>
    (search === "" || mentor.name.toLowerCase().includes(search.toLowerCase())) &&
    (selectedSpecialization === "" || mentor.specialization === selectedSpecialization) &&
    (selectedSlot === "" || mentor.availableSlots.includes(selectedSlot))
  );

  if (loading) {
    return <div className="text-center py-10">Loading...</div>;
  }

  return (
    <div className="max-w-5xl mx-auto p-6 bg-gray-100 min-h-screen">
      <h2 className="text-3xl font-bold mb-6">Student Dashboard</h2>
      
      <div className="bg-white p-4 rounded-lg shadow-md mb-4">
        <h3 className="text-xl font-semibold">Profile</h3>
        <p><strong>Name:</strong> John Doe</p>
        <p><strong>Email:</strong> johndoe@example.com</p>
      </div>
      
      <div className="bg-white p-4 rounded-lg shadow-md mb-4 flex flex-wrap gap-4">
        <input type="text" placeholder="Search mentors..." value={search} onChange={(e) => setSearch(e.target.value)} className="border p-2 flex-1" />
        <select onChange={(e) => setSelectedSpecialization(e.target.value)} className="border p-2">
          <option value="">Select Specialization</option>
          {specializations.map((spec) => <option key={spec} value={spec}>{spec}</option>)}
        </select>
        <select onChange={(e) => setSelectedSlot(e.target.value)} className="border p-2">
          <option value="">Select Slot</option>
          {slots.map((slot) => <option key={slot} value={slot}>{slot}</option>)}
        </select>
      </div>
      
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        {filteredMentors.map((mentor) => (
          <div key={mentor.id} className="bg-white p-4 rounded-lg shadow-lg">
            <h3 className="text-xl font-semibold">{mentor.name}</h3>
            <p className="text-gray-600">Specialization: {mentor.specialization}</p>
            <p className="text-gray-600">Available Slots:</p>
            <ul className="mb-2">
              {mentor.availableSlots.map((slot, index) => (
                <li key={index} className="text-sm bg-blue-100 px-2 py-1 rounded inline-block m-1">{slot}</li>
              ))}
            </ul>
            <button
              className="mt-2 px-4 py-2 bg-green-500 text-white rounded w-full"
              onClick={() => bookSession(mentor.id, selectedSlot)}
              disabled={!selectedSlot || !mentor.availableSlots.includes(selectedSlot)}
            >
              Book Session
            </button>
          </div>
        ))}
      </div>
    </div>
  );
}
```
