import React, { useState, useEffect } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import * as XLSX from "xlsx";

const days = ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"];

function StaffInputCard({
  staff,
  index,
  handleChange,
  handleRemoveStaff,
  handleDayToggle,
  handleAllDaysToggle,
  handleAvailabilityChange,
  handleBreakChange,
  totalAssignedHours,
  maxWeeklyHours,
}) {
  const exceeded = totalAssignedHours[staff.name] > (staff.maxWeeklyHours || maxWeeklyHours);

  return (
    <Card key={staff.id} className="mb-4">
      <CardContent className="p-4 space-y-2">
        <div className="flex space-x-2 text-sm items-center">
          <input
            type="text"
            placeholder="Name"
            value={staff.name}
            onChange={(e) => handleChange(index, "name", e.target.value)}
            className="border p-1 w-full"
          />
          <input
            type="number"
            placeholder="Max Hrs"
            value={staff.maxWeeklyHours || ""}
            onChange={(e) => handleChange(index, "maxWeeklyHours", e.target.value)}
            className="border p-1 w-24 text-sm"
          />
          <Button variant="outline" size="sm" onClick={() => handleRemoveStaff(staff.id)}>
            Remove
          </Button>
        </div>
        <div className="flex flex-wrap gap-2 text-xs">
          <label className="flex items-center space-x-1">
            <input
              type="checkbox"
              checked={staff.selectedDays.length === days.length}
              onChange={() => handleAllDaysToggle(index)}
            />
            <span>All</span>
          </label>
          {days.map((day) => (
            <label key={day} className="flex items-center space-x-1">
              <input
                type="checkbox"
                checked={staff.selectedDays.includes(day)}
                onChange={() => handleDayToggle(index, day)}
              />
              <span>{day}</span>
            </label>
          ))}
        </div>
        <div className="flex flex-wrap gap-2 mt-2">
          {days.map(
            (day) =>
              staff.selectedDays.includes(day) && (
                <div key={day} className="flex flex-col items-start">
                  <label className="text-xs font-semibold text-gray-700">
                    {day}
                  </label>
                  <input
                    type="text"
                    placeholder="e.g. 9-17"
                    value={staff.availability[day] || "9-17"}
                    onChange={(e) => handleAvailabilityChange(index, day, e.target.value)}
                    className="border p-1 w-24 text-xs"
                  />
                  <input
                    type="number"
                    placeholder="Break (h)"
                    value={staff.breaks?.[day] || 1}
                    onChange={(e) => handleBreakChange(index, day, e.target.value)}
                    className="border p-1 w-24 text-xs mt-1"
                  />
                </div>
              )
          )}
        </div>
        <div className={`text-xs ${exceeded ? 'text-red-600 font-semibold' : 'text-gray-600'}`}>
          Total Assigned Hours: {totalAssignedHours[staff.name] || 0}
          {exceeded && ' (Exceeded!)'}
        </div>
      </CardContent>
    </Card>
  );
}

export default function RosteringMVP() {
  const [maxWeeklyHours, setMaxWeeklyHours] = useState(40);
  const [staffList, setStaffList] = useState([]);
  const [totalAssignedHours, setTotalAssignedHours] = useState({});
  const [showRoster, setShowRoster] = useState(false);

  const handleAddStaff = () => {
    setStaffList([...staffList, {
      id: Date.now(),
      name: "",
      selectedDays: [],
      availability: {},
      breaks: {},
      maxWeeklyHours: ""
    }]);
  };

  const calculateHours = (timeStr, breakTime = 1) => {
    const [start, end] = timeStr.split("-").map(Number);
    return end > start ? end - start - breakTime : 0;
  };

  useEffect(() => {
    const totals = {};
    staffList.forEach((staff) => {
      let sum = 0;
      days.forEach((day) => {
        if (staff.selectedDays.includes(day)) {
          const slot = staff.availability[day] || "9-17";
          const breakTime = parseFloat(staff.breaks?.[day] || 1);
          sum += calculateHours(slot, breakTime);
        }
      });
      totals[staff.name] = sum;
    });
    setTotalAssignedHours(totals);
  }, [staffList]);

  const generateRoster = () => {
    const updatedStaffList = staffList.map((staff) => {
      let runningTotal = 0;
      const selectedDays = [];
      const limit = parseFloat(staff.maxWeeklyHours || maxWeeklyHours);

      for (const day of days) {
        if (staff.selectedDays.includes(day)) {
          const slot = staff.availability[day] || "9-17";
          const breakTime = parseFloat(staff.breaks?.[day] || 1);
          const hours = calculateHours(slot, breakTime);

          if (runningTotal + hours <= limit) {
            runningTotal += hours;
            selectedDays.push(day);
          }
        }
      }

      return { ...staff, selectedDays };
    });

    setStaffList(updatedStaffList);
    setShowRoster(true);
  };

  const exportToExcel = () => {
    const worksheetData = [
      ["Day", ...staffList.map((s) => s.name)]
    ];

    days.forEach((day) => {
      const row = [day];
      staffList.forEach((staff) => {
        const time = staff.selectedDays.includes(day) ? staff.availability[day] || "9-17" : "-";
        row.push(time);
      });
      worksheetData.push(row);
    });

    const totalRow = ["Total Hrs"];
    staffList.forEach((staff) => {
      totalRow.push(totalAssignedHours[staff.name] || 0);
    });
    worksheetData.push(totalRow);

    const worksheet = XLSX.utils.aoa_to_sheet(worksheetData);
    const workbook = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(workbook, worksheet, "Roster");
    XLSX.writeFile(workbook, "roster.xlsx");
  };

  return (
    <div className="mb-4 p-4">
      <label className="block text-sm font-medium mb-1">
        Max Working Hours / Week (Global Rule):
      </label>
      <input
        type="number"
        className="border p-1 w-32 text-sm"
        value={maxWeeklyHours}
        onChange={(e) => setMaxWeeklyHours(parseInt(e.target.value) || 0)}
        placeholder="e.g. 40"
      />

      <Button className="mt-4 mr-2" onClick={handleAddStaff}>
        Add Staff
      </Button>

      <Button className="mt-4 mr-2" onClick={generateRoster}>
        Generate Roster
      </Button>

      <Button className="mt-4" onClick={exportToExcel}>
        Export to Excel
      </Button>

      {staffList.map((staff, index) => (
        <StaffInputCard
          key={staff.id}
          staff={staff}
          index={index}
          handleChange={(i, field, value) => {
            const updated = [...staffList];
            updated[i][field] = value;
            setStaffList(updated);
          }}
          handleRemoveStaff={(id) => setStaffList(staffList.filter((s) => s.id !== id))}
          handleDayToggle={(i, day) => {
            const updated = [...staffList];
            const selected = updated[i].selectedDays;
            updated[i].selectedDays = selected.includes(day)
              ? selected.filter((d) => d !== day)
              : [...selected, day];
            setStaffList(updated);
          }}
          handleAllDaysToggle={(i) => {
            const updated = [...staffList];
            updated[i].selectedDays = updated[i].selectedDays.length === days.length ? [] : [...days];
            setStaffList(updated);
          }}
          handleAvailabilityChange={(i, day, value) => {
            const updated = [...staffList];
            updated[i].availability[day] = value;
            setStaffList(updated);
          }}
          handleBreakChange={(i, day, value) => {
            const updated = [...staffList];
            updated[i].breaks[day] = value;
            setStaffList(updated);
          }}
          totalAssignedHours={totalAssignedHours}
          maxWeeklyHours={maxWeeklyHours}
        />
      ))}

      {showRoster && (
        <div className="overflow-auto mt-8">
          <h2 className="text-lg font-bold mb-2">Generated Roster</h2>
          <table className="table-auto border border-gray-300">
            <thead>
              <tr>
                <th className="border px-2 py-1 text-sm">Day</th>
                {staffList.map((staff) => (
                  <th key={staff.name} className="border px-2 py-1 text-sm">
                    {staff.name}
                  </th>
                ))}
              </tr>
            </thead>
            <tbody>
              {days.map((day) => (
                <tr key={day}>
                  <td className="border px-2 py-1 text-sm font-medium">{day}</td>
                  {staffList.map((staff) => {
                    const time = staff.selectedDays.includes(day) ? staff.availability[day] || "9-17" : "-";
                    return (
                      <td key={staff.name + day} className="border px-2 py-1 text-sm text-center">
                        {time}
                      </td>
                    );
                  })}
                </tr>
              ))}
              <tr className="bg-gray-100">
                <td className="border px-2 py-1 font-semibold text-sm">Total Hrs</td>
                {staffList.map((staff) => (
                  <td key={staff.name + "total"} className="border px-2 py-1 text-sm text-center">
                    {totalAssignedHours[staff.name] || 0}
                  </td>
                ))}
              </tr>
            </tbody>
          </table>
        </div>
      )}
    </div>
  );
}
